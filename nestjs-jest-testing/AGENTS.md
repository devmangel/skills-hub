# NestJS Jest Testing - Guía Completa

Guía completa de mejores prácticas para testing de aplicaciones NestJS con Jest y TypeScript.

## Tabla de Contenidos

1. [Estructura y Organización](#1-estructura-y-organización)
2. [Unit Testing](#2-unit-testing)
3. [Mocking y Spies](#3-mocking-y-spies)
4. [Integration Testing](#4-integration-testing)
5. [E2E Testing](#5-e2e-testing)
6. [Database Testing](#6-database-testing)
7. [Optimización y Performance](#7-optimización-y-performance)
8. [CI/CD y Cobertura](#8-cicd-y-cobertura)

---

## 1. Estructura y Organización

### structure-file-location
**Impacto: CRÍTICO**

Coloca los archivos de test junto a los archivos fuente que están probando.

**Incorrecto:**
```
src/
  users/
    users.service.ts
    users.controller.ts
test/
  users/
    users.service.spec.ts
    users.controller.spec.ts
```

**Correcto:**
```
src/
  users/
    users.service.ts
    users.service.spec.ts
    users.controller.ts
    users.controller.spec.ts
```

**Por qué:** Mantener los tests cerca del código fuente facilita el mantenimiento, hace más fácil encontrar los tests relacionados, y mejora la experiencia del desarrollador.

---

### structure-naming-convention
**Impacto: CRÍTICO**

Usa nomenclatura consistente: `*.spec.ts` para unit tests, `*.e2e-spec.ts` para E2E tests.

**Incorrecto:**
```typescript
// users.test.ts
// users-test.ts
// users_spec.ts
```

**Correcto:**
```typescript
// users.service.spec.ts (unit test)
// users.controller.spec.ts (unit test)
// app.e2e-spec.ts (E2E test)
```

**Por qué:** Convención estándar de NestJS que permite a Jest identificar automáticamente los tests y aplicar las configuraciones correctas.

---

### structure-describe-blocks
**Impacto: ALTO**

Organiza tests en bloques `describe()` lógicos y anidados.

**Incorrecto:**
```typescript
it('should create user', () => {});
it('should update user', () => {});
it('should delete user', () => {});
it('should find user', () => {});
```

**Correcto:**
```typescript
describe('UsersService', () => {
  describe('create', () => {
    it('should create a new user successfully', () => {});
    it('should throw error if email exists', () => {});
  });

  describe('update', () => {
    it('should update user data', () => {});
    it('should throw error if user not found', () => {});
  });

  describe('delete', () => {
    it('should delete user successfully', () => {});
  });
});
```

**Por qué:** Mejora la legibilidad, organiza los tests por funcionalidad, y hace más fácil identificar qué parte del código está fallando.

---

### structure-setup-teardown
**Impacto: ALTO**

Usa `beforeEach()` para setup y `afterEach()` para cleanup consistentemente.

**Incorrecto:**
```typescript
describe('UsersService', () => {
  it('should create user', () => {
    const module = await Test.createTestingModule({...}).compile();
    const service = module.get(UsersService);
    // test...
  });

  it('should update user', () => {
    const module = await Test.createTestingModule({...}).compile();
    const service = module.get(UsersService);
    // test...
  });
});
```

**Correcto:**
```typescript
describe('UsersService', () => {
  let service: UsersService;
  let repository: Repository<User>;

  beforeEach(async () => {
    const module = await Test.createTestingModule({
      providers: [
        UsersService,
        {
          provide: getRepositoryToken(User),
          useValue: mockRepository,
        },
      ],
    }).compile();

    service = module.get<UsersService>(UsersService);
    repository = module.get(getRepositoryToken(User));
  });

  afterEach(() => {
    jest.clearAllMocks();
  });

  it('should create user', () => {
    // test...
  });
});
```

**Por qué:** Evita duplicación de código, asegura un estado limpio entre tests, y hace los tests más mantenibles.

---

## 2. Unit Testing

### unit-isolated-testing
**Impacto: CRÍTICO**

En unit tests, mockea TODAS las dependencias externas.

**Incorrecto:**
```typescript
describe('UsersService', () => {
  let service: UsersService;
  
  beforeEach(async () => {
    const module = await Test.createTestingModule({
      imports: [TypeOrmModule.forRoot(), UsersModule], // Usa módulos reales
      providers: [UsersService],
    }).compile();

    service = module.get(UsersService);
  });

  it('should create user', async () => {
    const result = await service.create({ email: 'test@test.com' });
    expect(result).toBeDefined();
  });
});
```

**Correcto:**
```typescript
describe('UsersService', () => {
  let service: UsersService;
  let repository: jest.Mocked<Repository<User>>;

  beforeEach(async () => {
    const mockRepository = {
      create: jest.fn(),
      save: jest.fn(),
      findOne: jest.fn(),
      update: jest.fn(),
      delete: jest.fn(),
    };

    const module = await Test.createTestingModule({
      providers: [
        UsersService,
        {
          provide: getRepositoryToken(User),
          useValue: mockRepository,
        },
      ],
    }).compile();

    service = module.get<UsersService>(UsersService);
    repository = module.get(getRepositoryToken(User));
  });

  it('should create user', async () => {
    const userData = { email: 'test@test.com', name: 'Test' };
    const savedUser = { id: 1, ...userData };

    repository.create.mockReturnValue(savedUser as User);
    repository.save.mockResolvedValue(savedUser as User);

    const result = await service.create(userData);

    expect(repository.create).toHaveBeenCalledWith(userData);
    expect(repository.save).toHaveBeenCalled();
    expect(result).toEqual(savedUser);
  });
});
```

**Por qué:** Los unit tests deben ser rápidos, confiables y aislados. Mockear dependencias evita efectos secundarios, dependencias de red/DB, y hace los tests determinísticos.

---

### unit-one-assertion-per-test
**Impacto: MEDIO**

Cada test debe verificar un comportamiento específico.

**Incorrecto:**
```typescript
it('should handle user operations', async () => {
  // Crea usuario
  const created = await service.create(userData);
  expect(created).toBeDefined();
  
  // Actualiza usuario
  const updated = await service.update(1, { name: 'New' });
  expect(updated.name).toBe('New');
  
  // Elimina usuario
  await service.delete(1);
  const deleted = await service.findOne(1);
  expect(deleted).toBeNull();
});
```

**Correcto:**
```typescript
describe('UsersService', () => {
  describe('create', () => {
    it('should create user with valid data', async () => {
      const userData = { email: 'test@test.com', name: 'Test' };
      const result = await service.create(userData);
      
      expect(result).toBeDefined();
      expect(result.email).toBe(userData.email);
      expect(repository.save).toHaveBeenCalledWith(userData);
    });
  });

  describe('update', () => {
    it('should update user name', async () => {
      const updateData = { name: 'New Name' };
      repository.findOne.mockResolvedValue({ id: 1, ...updateData } as User);
      
      const result = await service.update(1, updateData);
      
      expect(result.name).toBe(updateData.name);
      expect(repository.update).toHaveBeenCalledWith(1, updateData);
    });
  });

  describe('delete', () => {
    it('should delete user by id', async () => {
      await service.delete(1);
      
      expect(repository.delete).toHaveBeenCalledWith(1);
    });
  });
});
```

**Por qué:** Tests enfocados son más fáciles de entender, mantener y debuggear. Cuando falla, sabes exactamente qué comportamiento específico está roto.

---

### unit-test-error-cases
**Impacto: ALTO**

Siempre testea tanto casos exitosos como casos de error.

**Incorrecto:**
```typescript
describe('UsersService', () => {
  it('should create user', async () => {
    const result = await service.create(userData);
    expect(result).toBeDefined();
  });
  
  // Solo testea el caso feliz
});
```

**Correcto:**
```typescript
describe('UsersService', () => {
  describe('create', () => {
    it('should create user successfully', async () => {
      const userData = { email: 'test@test.com', name: 'Test' };
      repository.save.mockResolvedValue({ id: 1, ...userData } as User);

      const result = await service.create(userData);

      expect(result).toBeDefined();
      expect(result.email).toBe(userData.email);
    });

    it('should throw ConflictException if email exists', async () => {
      const userData = { email: 'existing@test.com', name: 'Test' };
      repository.findOne.mockResolvedValue({ id: 1, ...userData } as User);

      await expect(service.create(userData)).rejects.toThrow(
        ConflictException,
      );
    });

    it('should throw BadRequestException for invalid data', async () => {
      const invalidData = { email: 'invalid-email' };

      await expect(service.create(invalidData as any)).rejects.toThrow(
        BadRequestException,
      );
    });
  });
});
```

**Por qué:** Los casos de error son críticos para la robustez de la aplicación. Testearlos asegura que tu código maneja errores apropiadamente.

---

### unit-controller-testing
**Impacto: ALTO**

Testea controllers mockeando servicios, no lógica de negocio.

**Incorrecto:**
```typescript
describe('UsersController', () => {
  let controller: UsersController;
  let service: UsersService; // Servicio real

  beforeEach(async () => {
    const module = await Test.createTestingModule({
      controllers: [UsersController],
      providers: [UsersService, UserRepository], // Dependencias reales
    }).compile();

    controller = module.get(UsersController);
    service = module.get(UsersService);
  });

  it('should create user', async () => {
    // Prueba lógica de negocio en controller test
  });
});
```

**Correcto:**
```typescript
describe('UsersController', () => {
  let controller: UsersController;
  let service: jest.Mocked<UsersService>;

  beforeEach(async () => {
    const mockService = {
      create: jest.fn(),
      findAll: jest.fn(),
      findOne: jest.fn(),
      update: jest.fn(),
      delete: jest.fn(),
    };

    const module = await Test.createTestingModule({
      controllers: [UsersController],
      providers: [
        {
          provide: UsersService,
          useValue: mockService,
        },
      ],
    }).compile();

    controller = module.get<UsersController>(UsersController);
    service = module.get(UsersService);
  });

  describe('create', () => {
    it('should call service.create with correct data', async () => {
      const createDto = { email: 'test@test.com', name: 'Test' };
      const expectedResult = { id: 1, ...createDto };
      
      service.create.mockResolvedValue(expectedResult as User);

      const result = await controller.create(createDto);

      expect(service.create).toHaveBeenCalledWith(createDto);
      expect(result).toEqual(expectedResult);
    });

    it('should handle service errors', async () => {
      const createDto = { email: 'test@test.com', name: 'Test' };
      service.create.mockRejectedValue(new ConflictException());

      await expect(controller.create(createDto)).rejects.toThrow(
        ConflictException,
      );
    });
  });
});
```

**Por qué:** Los controllers solo deben orquestar llamadas a servicios. Sus tests verifican que llaman los métodos correctos con los argumentos correctos.

---

## 3. Mocking y Spies

### mock-factory-pattern
**Impacto: ALTO**

Crea factories reutilizables para mocks complejos.

**Incorrecto:**
```typescript
// Duplicado en cada test file
const mockRepository = {
  create: jest.fn(),
  save: jest.fn(),
  findOne: jest.fn(),
  update: jest.fn(),
  delete: jest.fn(),
};

const mockConfigService = {
  get: jest.fn(),
};

// Repetido en múltiples archivos...
```

**Correcto:**
```typescript
// test/mocks/repository.mock.ts
export const createMockRepository = <T = any>() => ({
  create: jest.fn((entity: T) => entity),
  save: jest.fn((entity: T) => Promise.resolve(entity)),
  findOne: jest.fn((options?: any) => Promise.resolve(null)),
  find: jest.fn(() => Promise.resolve([])),
  update: jest.fn(() => Promise.resolve({ affected: 1 })),
  delete: jest.fn(() => Promise.resolve({ affected: 1 })),
  count: jest.fn(() => Promise.resolve(0)),
  createQueryBuilder: jest.fn(() => ({
    where: jest.fn().mockReturnThis(),
    andWhere: jest.fn().mockReturnThis(),
    getOne: jest.fn(() => Promise.resolve(null)),
    getMany: jest.fn(() => Promise.resolve([])),
  })),
});

// test/mocks/config.mock.ts
export const createMockConfigService = () => ({
  get: jest.fn((key: string) => {
    const config = {
      DATABASE_HOST: 'localhost',
      DATABASE_PORT: 5432,
      JWT_SECRET: 'test-secret',
    };
    return config[key];
  }),
});

// Uso en tests
import { createMockRepository } from '../../../test/mocks/repository.mock';

describe('UsersService', () => {
  let service: UsersService;
  let repository: ReturnType<typeof createMockRepository>;

  beforeEach(async () => {
    repository = createMockRepository<User>();

    const module = await Test.createTestingModule({
      providers: [
        UsersService,
        {
          provide: getRepositoryToken(User),
          useValue: repository,
        },
      ],
    }).compile();

    service = module.get<UsersService>(UsersService);
  });
});
```

**Por qué:** Reduce duplicación, asegura consistencia en mocks, facilita mantenimiento, y mejora reutilización.

---

### mock-partial-mocks
**Impacto: MEDIO**

Usa mocks parciales cuando solo necesitas mockear algunos métodos.

**Incorrecto:**
```typescript
const mockService = {
  findAll: jest.fn(),
  findOne: jest.fn(),
  create: jest.fn(),
  update: jest.fn(),
  delete: jest.fn(),
  // Mockeas todo aunque solo uses uno o dos métodos
};
```

**Correcto:**
```typescript
describe('UsersController', () => {
  let controller: UsersController;
  let service: UsersService;

  beforeEach(async () => {
    const module = await Test.createTestingModule({
      controllers: [UsersController],
      providers: [UsersService],
    }).compile();

    controller = module.get(UsersController);
    service = module.get(UsersService);
  });

  it('should find all users', async () => {
    const expectedUsers = [{ id: 1, email: 'test@test.com' }];
    
    // Solo mockea el método que necesitas
    jest.spyOn(service, 'findAll').mockResolvedValue(expectedUsers as User[]);

    const result = await controller.findAll();

    expect(result).toEqual(expectedUsers);
  });
});
```

**Por qué:** Más limpio, más específico, y evita mockear métodos innecesariamente.

---

### mock-reset-between-tests
**Impacto: ALTO**

Limpia mocks entre tests para evitar interferencias.

**Incorrecto:**
```typescript
describe('UsersService', () => {
  let service: UsersService;
  let repository: any;

  beforeEach(async () => {
    repository = createMockRepository();
    // Sin limpieza de mocks
  });

  it('test 1', () => {
    repository.findOne.mockResolvedValue({ id: 1 });
    // ...
  });

  it('test 2', () => {
    // Mock del test anterior puede interferir aquí
  });
});
```

**Correcto:**
```typescript
describe('UsersService', () => {
  let service: UsersService;
  let repository: ReturnType<typeof createMockRepository>;

  beforeEach(async () => {
    repository = createMockRepository<User>();

    const module = await Test.createTestingModule({
      providers: [
        UsersService,
        {
          provide: getRepositoryToken(User),
          useValue: repository,
        },
      ],
    }).compile();

    service = module.get<UsersService>(UsersService);
  });

  afterEach(() => {
    jest.clearAllMocks(); // Limpia call history pero mantiene implementaciones
    // O usa jest.resetAllMocks() para limpiar todo
  });

  it('test 1', async () => {
    repository.findOne.mockResolvedValue({ id: 1 } as User);
    // ...
  });

  it('test 2', async () => {
    // Estado limpio, sin interferencias
    repository.findOne.mockResolvedValue({ id: 2 } as User);
    // ...
  });
});
```

**Por qué:** Previene falsos positivos/negativos causados por mocks de tests anteriores que persisten.

---

### mock-type-safety
**Impacto: MEDIO**

Mantén type safety en tus mocks usando TypeScript correctamente.

**Incorrecto:**
```typescript
const mockService: any = {
  findOne: jest.fn(),
  // Pierdes type safety completamente
};
```

**Correcto:**
```typescript
// Opción 1: Usar jest.Mocked
const mockService: jest.Mocked<UsersService> = {
  findOne: jest.fn(),
  findAll: jest.fn(),
  create: jest.fn(),
  update: jest.fn(),
  delete: jest.fn(),
} as jest.Mocked<UsersService>;

// Opción 2: Usar tipo parcial con jest.Mocked en métodos específicos
type MockedUsersService = {
  [K in keyof UsersService]: jest.MockedFunction<UsersService[K]>;
};

const createMockUsersService = (): MockedUsersService => ({
  findOne: jest.fn(),
  findAll: jest.fn(),
  create: jest.fn(),
  update: jest.fn(),
  delete: jest.fn(),
});

// Opción 3: Para mocks parciales
const mockService = {
  findOne: jest.fn(),
} as Pick<UsersService, 'findOne'>;
```

**Por qué:** Type safety previene errores de typos en nombres de métodos, asegura que los mocks coincidan con las interfaces reales, y mejora el autocomplete del IDE.

---

## 4. Integration Testing

### integration-real-modules
**Impacto: ALTO**

En integration tests, usa módulos reales cuando sea posible, mockea solo dependencias externas.

**Incorrecto:**
```typescript
// Trata integration test como unit test
describe('UsersModule Integration', () => {
  let service: UsersService;
  
  beforeEach(async () => {
    const module = await Test.createTestingModule({
      providers: [
        UsersService,
        { provide: UserRepository, useValue: mockRepo },
        { provide: AuthService, useValue: mockAuth },
        // Mockea todo
      ],
    }).compile();
  });
});
```

**Correcto:**
```typescript
describe('UsersModule Integration', () => {
  let app: INestApplication;
  let usersService: UsersService;
  let authService: AuthService;

  beforeEach(async () => {
    const moduleFixture = await Test.createTestingModule({
      imports: [
        UsersModule, // Módulo real con todas sus dependencias
        AuthModule,  // Módulo real
        // Solo mockea dependencias externas como DB
        TypeOrmModule.forRoot({
          type: 'sqlite',
          database: ':memory:',
          entities: [User],
          synchronize: true,
        }),
      ],
    }).compile();

    app = moduleFixture.createNestApplication();
    await app.init();

    usersService = moduleFixture.get<UsersService>(UsersService);
    authService = moduleFixture.get<AuthService>(AuthService);
  });

  afterEach(async () => {
    await app.close();
  });

  it('should create user and authenticate', async () => {
    // Test interacción real entre módulos
    const userData = { email: 'test@test.com', password: 'password123' };
    
    const user = await usersService.create(userData);
    expect(user).toBeDefined();
    
    const token = await authService.login(user);
    expect(token).toBeDefined();
  });
});
```

**Por qué:** Integration tests verifican que múltiples componentes trabajan juntos correctamente. Usar módulos reales da confianza en las integraciones.

---

### integration-transaction-rollback
**Impacto: ALTO**

Usa transacciones con rollback en integration tests con DB.

**Incorrecto:**
```typescript
describe('UsersModule Integration', () => {
  beforeEach(async () => {
    // No limpia datos entre tests
  });

  it('creates user', async () => {
    await usersService.create({ email: 'test@test.com' });
  });

  it('finds user', async () => {
    // Puede encontrar usuario del test anterior
    const users = await usersService.findAll();
  });
});
```

**Correcto:**
```typescript
describe('UsersModule Integration', () => {
  let app: INestApplication;
  let connection: DataSource;

  beforeEach(async () => {
    const moduleFixture = await Test.createTestingModule({
      imports: [
        TypeOrmModule.forRoot({
          type: 'postgres',
          host: 'localhost',
          port: 5433, // Puerto diferente para testing
          database: 'test_db',
          entities: [User],
          synchronize: true,
        }),
        UsersModule,
      ],
    }).compile();

    app = moduleFixture.createNestApplication();
    await app.init();

    connection = moduleFixture.get(DataSource);
  });

  afterEach(async () => {
    // Limpia todas las tablas después de cada test
    const entities = connection.entityMetadatas;
    for (const entity of entities) {
      const repository = connection.getRepository(entity.name);
      await repository.clear();
    }
  });

  afterAll(async () => {
    await app.close();
  });

  it('creates user', async () => {
    const usersService = app.get(UsersService);
    const user = await usersService.create({ email: 'test@test.com' });
    expect(user.id).toBeDefined();
  });

  it('finds user', async () => {
    // Estado limpio, no hay usuarios del test anterior
    const usersService = app.get(UsersService);
    const users = await usersService.findAll();
    expect(users).toHaveLength(0);
  });
});
```

**Por qué:** Tests aislados previenen falsos positivos/negativos causados por estado compartido entre tests.

---

## 5. E2E Testing

### e2e-supertest-setup
**Impacto: CRÍTICO**

Usa Supertest para E2E tests HTTP en NestJS.

**Incorrecto:**
```typescript
describe('Users E2E', () => {
  it('should create user', async () => {
    // Usa fetch o axios directamente
    const response = await fetch('http://localhost:3000/users', {
      method: 'POST',
      body: JSON.stringify({ email: 'test@test.com' }),
    });
  });
});
```

**Correcto:**
```typescript
import * as request from 'supertest';
import { Test, TestingModule } from '@nestjs/testing';
import { INestApplication, ValidationPipe } from '@nestjs/common';
import { AppModule } from '../src/app.module';

describe('Users E2E', () => {
  let app: INestApplication;

  beforeAll(async () => {
    const moduleFixture: TestingModule = await Test.createTestingModule({
      imports: [AppModule],
    }).compile();

    app = moduleFixture.createNestApplication();
    
    // Aplica mismos pipes y configuración que producción
    app.useGlobalPipes(new ValidationPipe({
      whitelist: true,
      forbidNonWhitelisted: true,
      transform: true,
    }));

    await app.init();
  });

  afterAll(async () => {
    await app.close();
  });

  describe('/users (POST)', () => {
    it('should create user with valid data', () => {
      return request(app.getHttpServer())
        .post('/users')
        .send({ email: 'test@test.com', name: 'Test User' })
        .expect(201)
        .expect((res) => {
          expect(res.body).toHaveProperty('id');
          expect(res.body.email).toBe('test@test.com');
        });
    });

    it('should return 400 for invalid email', () => {
      return request(app.getHttpServer())
        .post('/users')
        .send({ email: 'invalid-email', name: 'Test' })
        .expect(400)
        .expect((res) => {
          expect(res.body.message).toContain('email');
        });
    });

    it('should return 409 for duplicate email', async () => {
      // Primero crea usuario
      await request(app.getHttpServer())
        .post('/users')
        .send({ email: 'test@test.com', name: 'Test' })
        .expect(201);

      // Intenta crear otro con mismo email
      return request(app.getHttpServer())
        .post('/users')
        .send({ email: 'test@test.com', name: 'Test 2' })
        .expect(409);
    });
  });
});
```

**Por qué:** Supertest se integra perfectamente con NestJS, no requiere servidor corriendo, y proporciona API fluida para testing HTTP.

---

### e2e-authentication-testing
**Impacto: ALTO**

Testea flujos de autenticación en E2E tests.

**Correcto:**
```typescript
describe('Authentication E2E', () => {
  let app: INestApplication;
  let authToken: string;

  beforeAll(async () => {
    const moduleFixture = await Test.createTestingModule({
      imports: [AppModule],
    }).compile();

    app = moduleFixture.createNestApplication();
    await app.init();
  });

  describe('/auth/register (POST)', () => {
    it('should register new user', () => {
      return request(app.getHttpServer())
        .post('/auth/register')
        .send({
          email: 'test@test.com',
          password: 'Password123!',
          name: 'Test User',
        })
        .expect(201)
        .expect((res) => {
          expect(res.body).toHaveProperty('access_token');
          authToken = res.body.access_token;
        });
    });
  });

  describe('/auth/login (POST)', () => {
    it('should login with valid credentials', () => {
      return request(app.getHttpServer())
        .post('/auth/login')
        .send({
          email: 'test@test.com',
          password: 'Password123!',
        })
        .expect(200)
        .expect((res) => {
          expect(res.body).toHaveProperty('access_token');
          authToken = res.body.access_token;
        });
    });

    it('should reject invalid credentials', () => {
      return request(app.getHttpServer())
        .post('/auth/login')
        .send({
          email: 'test@test.com',
          password: 'WrongPassword',
        })
        .expect(401);
    });
  });

  describe('/users/profile (GET)', () => {
    it('should get profile with valid token', () => {
      return request(app.getHttpServer())
        .get('/users/profile')
        .set('Authorization', `Bearer ${authToken}`)
        .expect(200)
        .expect((res) => {
          expect(res.body.email).toBe('test@test.com');
        });
    });

    it('should reject without token', () => {
      return request(app.getHttpServer())
        .get('/users/profile')
        .expect(401);
    });

    it('should reject with invalid token', () => {
      return request(app.getHttpServer())
        .get('/users/profile')
        .set('Authorization', 'Bearer invalid-token')
        .expect(401);
    });
  });
});
```

**Por qué:** Testear autenticación E2E asegura que guards, JWT, y flujos de autenticación funcionan correctamente en conjunto.

---

### e2e-database-isolation
**Impacto: CRÍTICO**

Usa base de datos aislada para E2E tests.

**Incorrecto:**
```typescript
// Usa misma DB que desarrollo
describe('E2E', () => {
  beforeAll(async () => {
    const module = await Test.createTestingModule({
      imports: [
        TypeOrmModule.forRoot({
          host: 'localhost',
          port: 5432,
          database: 'my_dev_db', // ❌ Misma DB que desarrollo
        }),
      ],
    }).compile();
  });
});
```

**Correcto:**
```typescript
// test/jest-e2e.json
{
  "moduleFileExtensions": ["js", "json", "ts"],
  "rootDir": ".",
  "testEnvironment": "node",
  "testRegex": ".e2e-spec.ts$",
  "transform": {
    "^.+\\.(t|j)s$": "ts-jest"
  }
}

// test/test.config.ts
export const testConfig = {
  database: {
    type: 'postgres',
    host: 'localhost',
    port: 5433, // Puerto diferente
    database: 'test_db', // DB separada
    dropSchema: true,
    synchronize: true,
  },
};

describe('E2E Tests', () => {
  let app: INestApplication;

  beforeAll(async () => {
    const moduleFixture = await Test.createTestingModule({
      imports: [
        TypeOrmModule.forRoot(testConfig.database),
        AppModule,
      ],
    }).compile();

    app = moduleFixture.createNestApplication();
    await app.init();
  });

  afterAll(async () => {
    await app.close();
  });
});
```

**Por qué:** E2E tests pueden modificar/eliminar datos. Una DB aislada previene pérdida de datos y resultados inconsistentes.

---

## 6. Database Testing

### db-in-memory-database
**Impacto: ALTO**

Usa bases de datos en memoria para tests rápidos.

**Correcto:**
```typescript
// test/database.config.ts
export const testDatabaseConfig = {
  type: 'sqlite',
  database: ':memory:', // Base de datos en memoria
  entities: [User, Post, Comment],
  synchronize: true,
  logging: false,
};

// En tests
describe('UsersService Integration', () => {
  let app: INestApplication;
  let service: UsersService;

  beforeAll(async () => {
    const module = await Test.createTestingModule({
      imports: [
        TypeOrmModule.forRoot(testDatabaseConfig),
        UsersModule,
      ],
    }).compile();

    app = module.createNestApplication();
    await app.init();
    
    service = module.get<UsersService>(UsersService);
  });

  afterAll(async () => {
    await app.close();
  });

  it('should create and find user', async () => {
    const userData = { email: 'test@test.com', name: 'Test' };
    const created = await service.create(userData);
    
    const found = await service.findOne(created.id);
    expect(found.email).toBe(userData.email);
  });
});
```

**Por qué:** Bases de datos en memoria son más rápidas, no requieren setup externo, y se limpian automáticamente.

---

### db-test-fixtures
**Impacto: MEDIO**

Crea fixtures reutilizables para datos de prueba.

**Correcto:**
```typescript
// test/fixtures/user.fixture.ts
export class UserFixture {
  static createUser(overrides?: Partial<User>): User {
    return {
      id: 1,
      email: 'test@test.com',
      name: 'Test User',
      password: 'hashed_password',
      createdAt: new Date(),
      updatedAt: new Date(),
      ...overrides,
    };
  }

  static createUsers(count: number): User[] {
    return Array.from({ length: count }, (_, i) =>
      this.createUser({
        id: i + 1,
        email: `test${i}@test.com`,
        name: `Test User ${i}`,
      }),
    );
  }

  static createDto(overrides?: Partial<CreateUserDto>): CreateUserDto {
    return {
      email: 'test@test.com',
      name: 'Test User',
      password: 'Password123!',
      ...overrides,
    };
  }
}

// Uso en tests
describe('UsersService', () => {
  it('should create user', async () => {
    const userData = UserFixture.createDto();
    const expectedUser = UserFixture.createUser({ ...userData });
    
    repository.save.mockResolvedValue(expectedUser);
    
    const result = await service.create(userData);
    expect(result).toEqual(expectedUser);
  });

  it('should find multiple users', async () => {
    const users = UserFixture.createUsers(5);
    repository.find.mockResolvedValue(users);
    
    const result = await service.findAll();
    expect(result).toHaveLength(5);
  });
});
```

**Por qué:** Fixtures centralizan creación de datos de prueba, reducen duplicación, y facilitan mantenimiento.

---

### db-seeding-strategy
**Impacto: MEDIO**

Implementa estrategia de seeding para tests de integración.

**Correcto:**
```typescript
// test/seed/seed.service.ts
@Injectable()
export class SeedService {
  constructor(
    @InjectRepository(User)
    private userRepository: Repository<User>,
    @InjectRepository(Post)
    private postRepository: Repository<Post>,
  ) {}

  async seedUsers(count: number = 10): Promise<User[]> {
    const users = Array.from({ length: count }, (_, i) => ({
      email: `user${i}@test.com`,
      name: `User ${i}`,
      password: 'hashed_password',
    }));

    return await this.userRepository.save(users);
  }

  async seedPosts(user: User, count: number = 5): Promise<Post[]> {
    const posts = Array.from({ length: count }, (_, i) => ({
      title: `Post ${i}`,
      content: `Content for post ${i}`,
      author: user,
    }));

    return await this.postRepository.save(posts);
  }

  async clearAll(): Promise<void> {
    await this.postRepository.clear();
    await this.userRepository.clear();
  }
}

// En tests
describe('Posts Integration', () => {
  let seedService: SeedService;
  let testUsers: User[];

  beforeEach(async () => {
    seedService = app.get(SeedService);
    await seedService.clearAll();
    testUsers = await seedService.seedUsers(5);
  });

  it('should find posts by author', async () => {
    const author = testUsers[0];
    await seedService.seedPosts(author, 3);
    
    const posts = await postsService.findByAuthor(author.id);
    expect(posts).toHaveLength(3);
  });
});
```

**Por qué:** Seeding facilita setup de datos complejos, hace tests más legibles, y permite reutilización.

---

## 7. Optimización y Performance

### perf-parallel-tests
**Impacto: ALTO**

Ejecuta tests en paralelo para mejor performance.

**Correcto:**
```typescript
// jest.config.js
module.exports = {
  moduleFileExtensions: ['js', 'json', 'ts'],
  rootDir: 'src',
  testRegex: '.*\\.spec\\.ts$',
  transform: {
    '^.+\\.(t|j)s$': 'ts-jest',
  },
  collectCoverageFrom: ['**/*.(t|j)s'],
  coverageDirectory: '../coverage',
  testEnvironment: 'node',
  maxWorkers: '50%', // Usa 50% de CPUs disponibles
  // O especifica número exacto:
  // maxWorkers: 4,
};

// package.json
{
  "scripts": {
    "test": "jest --maxWorkers=50%",
    "test:watch": "jest --watch --maxWorkers=25%",
    "test:cov": "jest --coverage --maxWorkers=50%",
    "test:debug": "node --inspect-brk -r tsconfig-paths/register -r ts-node/register node_modules/.bin/jest --runInBand"
  }
}
```

**Por qué:** Tests paralelos reducen tiempo total de ejecución significativamente en proyectos grandes.

---

### perf-test-timeouts
**Impacto: MEDIO**

Configura timeouts apropiados para diferentes tipos de tests.

**Correcto:**
```typescript
// jest.config.js
module.exports = {
  testTimeout: 10000, // 10 segundos por defecto
};

// Para tests específicos que necesitan más tiempo
describe('Heavy Integration Tests', () => {
  // Aumenta timeout para toda la suite
  jest.setTimeout(30000);

  it('should handle large data processing', async () => {
    // Test que puede tomar 20+ segundos
  });
});

// Para tests individuales
it('should complete complex operation', async () => {
  jest.setTimeout(15000); // Solo para este test
  // ...
}, 15000); // O especifica timeout aquí

// Para E2E tests
describe('E2E Tests', () => {
  beforeAll(() => {
    jest.setTimeout(60000); // 1 minuto para E2E
  });
});
```

**Por qué:** Timeouts apropiados previenen tests colgados pero permiten operaciones legítimamente lentas.

---

### perf-selective-testing
**Impacto: MEDIO**

Usa patrones de test selectivo durante desarrollo.

**Correcto:**
```bash
# Ejecuta solo archivos modificados
npm test -- --onlyChanged

# Ejecuta tests relacionados a archivos modificados
npm test -- --changedSince=main

# Ejecuta tests que coincidan con patrón
npm test -- --testNamePattern="UserService"

# Ejecuta solo un archivo específico
npm test -- users.service.spec.ts

# Modo watch con filtros
npm test -- --watch --testPathPattern=users

# Solo tests marcados con .only (para debug)
describe.only('Focus on this suite', () => {
  it.only('Run only this test', () => {});
});

# Skip tests temporalmente
describe.skip('Skip this suite', () => {});
it.skip('Skip this test', () => {});
```

**Por qué:** Testing selectivo acelera ciclo de desarrollo al ejecutar solo tests relevantes.

---

## 8. CI/CD y Cobertura

### ci-coverage-thresholds
**Impacto: MEDIO**

Define umbrales mínimos de cobertura.

**Correcto:**
```typescript
// jest.config.js
module.exports = {
  coverageThreshold: {
    global: {
      branches: 80,
      functions: 80,
      lines: 80,
      statements: 80,
    },
    // Umbrales específicos por directorio
    './src/core/': {
      branches: 90,
      functions: 90,
      lines: 90,
      statements: 90,
    },
    './src/utils/': {
      branches: 95,
      functions: 95,
      lines: 95,
      statements: 95,
    },
  },
  collectCoverageFrom: [
    'src/**/*.ts',
    '!src/**/*.spec.ts',
    '!src/**/*.e2e-spec.ts',
    '!src/main.ts',
    '!src/**/*.module.ts',
    '!src/**/*.interface.ts',
    '!src/**/*.dto.ts',
  ],
};

// package.json
{
  "scripts": {
    "test:cov": "jest --coverage",
    "test:cov:check": "jest --coverage --coverageThreshold='{\"global\":{\"branches\":80,\"functions\":80,\"lines\":80,\"statements\":80}}'",
  }
}
```

**Por qué:** Umbrales de cobertura mantienen calidad de código y previenen regresiones en testing.

---

### ci-github-actions
**Impacto: MEDIO**

Configura CI con GitHub Actions.

**Correcto:**
```yaml
# .github/workflows/test.yml
name: Tests

on:
  push:
    branches: [main, develop]
  pull_request:
    branches: [main, develop]

jobs:
  test:
    runs-on: ubuntu-latest

    strategy:
      matrix:
        node-version: [18.x, 20.x]

    services:
      postgres:
        image: postgres:15
        env:
          POSTGRES_PASSWORD: postgres
          POSTGRES_DB: test_db
        ports:
          - 5433:5432
        options: >-
          --health-cmd pg_isready
          --health-interval 10s
          --health-timeout 5s
          --health-retries 5

    steps:
      - uses: actions/checkout@v3

      - name: Use Node.js ${{ matrix.node-version }}
        uses: actions/setup-node@v3
        with:
          node-version: ${{ matrix.node-version }}
          cache: 'npm'

      - name: Install dependencies
        run: npm ci

      - name: Run linter
        run: npm run lint

      - name: Run unit tests
        run: npm run test:cov

      - name: Run E2E tests
        run: npm run test:e2e
        env:
          DATABASE_HOST: localhost
          DATABASE_PORT: 5433
          DATABASE_NAME: test_db
          DATABASE_USER: postgres
          DATABASE_PASSWORD: postgres

      - name: Upload coverage to Codecov
        uses: codecov/codecov-action@v3
        with:
          file: ./coverage/coverage-final.json
          flags: unittests
          name: codecov-umbrella

      - name: Check coverage thresholds
        run: npm run test:cov:check
```

**Por qué:** CI automatizado asegura que todos los tests pasen antes de merge y mantiene calidad de código.

---

### ci-docker-testing
**Impacto: MEDIO**

Usa Docker para entorno de testing consistente.

**Correcto:**
```yaml
# docker-compose.test.yml
version: '3.8'

services:
  test-db:
    image: postgres:15
    environment:
      POSTGRES_USER: test_user
      POSTGRES_PASSWORD: test_password
      POSTGRES_DB: test_db
    ports:
      - "5433:5432"
    healthcheck:
      test: ["CMD-SHELL", "pg_isready -U test_user"]
      interval: 5s
      timeout: 5s
      retries: 5

  test-redis:
    image: redis:7
    ports:
      - "6380:6379"
    healthcheck:
      test: ["CMD", "redis-cli", "ping"]
      interval: 5s
      timeout: 3s
      retries: 5

# package.json
{
  "scripts": {
    "test:docker:up": "docker-compose -f docker-compose.test.yml up -d",
    "test:docker:down": "docker-compose -f docker-compose.test.yml down -v",
    "test:integration": "npm run test:docker:up && jest --config ./test/jest-integration.json && npm run test:docker:down",
    "test:e2e": "npm run test:docker:up && jest --config ./test/jest-e2e.json && npm run test:docker:down"
  }
}
```

**Por qué:** Docker asegura entorno de testing consistente entre desarrollo local y CI.

---

### ci-test-reports
**Impacto: BAJO**

Genera reportes de tests legibles.

**Correcto:**
```typescript
// jest.config.js
module.exports = {
  reporters: [
    'default',
    [
      'jest-junit',
      {
        outputDirectory: './test-results',
        outputName: 'junit.xml',
        classNameTemplate: '{classname}',
        titleTemplate: '{title}',
        ancestorSeparator: ' › ',
        usePathForSuiteName: true,
      },
    ],
    [
      'jest-html-reporter',
      {
        pageTitle: 'Test Report',
        outputPath: './test-results/report.html',
        includeFailureMsg: true,
        includeConsoleLog: true,
      },
    ],
  ],
  coverageReporters: ['json', 'lcov', 'text', 'html', 'text-summary'],
};

// package.json
{
  "devDependencies": {
    "jest-junit": "^16.0.0",
    "jest-html-reporter": "^3.10.0"
  }
}
```

**Por qué:** Reportes mejorados facilitan identificación de problemas y tracking de cobertura.

---

## Resumen de Mejores Prácticas

### Prioridades por Categoría

**CRÍTICO - Implementar siempre:**
- Estructura de archivos junto al código fuente
- Nomenclatura consistente (*.spec.ts, *.e2e-spec.ts)
- Unit tests completamente aislados con mocks
- Supertest para E2E tests
- Base de datos aislada para tests

**ALTO - Altamente recomendado:**
- Organización con describe() anidados
- Setup/teardown consistente
- Testear casos de error
- Factory pattern para mocks
- Limpieza de mocks entre tests
- Módulos reales en integration tests
- Transacciones con rollback

**MEDIO - Recomendado:**
- Un comportamiento por test
- Mocks parciales cuando sea apropiado
- Type safety en mocks
- In-memory databases
- Fixtures reutilizables
- Tests paralelos
- Timeouts apropiados
- Umbrales de cobertura

**BAJO - Buenas prácticas:**
- Testing selectivo durante desarrollo
- CI/CD configurado
- Reportes de tests

---

## Comandos Útiles

```bash
# Ejecutar todos los tests
npm test

# Ejecutar con cobertura
npm run test:cov

# Ejecutar en modo watch
npm test -- --watch

# Ejecutar solo tests modificados
npm test -- --onlyChanged

# Ejecutar E2E tests
npm run test:e2e

# Ejecutar con verbose
npm test -- --verbose

# Ejecutar tests específicos
npm test -- users.service.spec

# Debug de tests
npm run test:debug

# Ver cobertura en navegador
npm run test:cov && open coverage/index.html
```

---

## Referencias y Recursos

- [NestJS Testing Documentation](https://docs.nestjs.com/fundamentals/testing)
- [Jest Documentation](https://jestjs.io/docs/getting-started)
- [Supertest GitHub](https://github.com/visionmedia/supertest)
- [TypeORM Testing](https://typeorm.io/usage-in-nestjs)
- [Testing Best Practices](https://github.com/goldbergyoni/javascript-testing-best-practices)

---

**Última actualización:** 2026-02-06  
**Versión:** 1.0.0  
**Licencia:** MIT
