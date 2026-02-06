---
name: nestjs-jest-testing
description: Guía completa de mejores prácticas para testing de aplicaciones backend NestJS con Jest y TypeScript. Incluye unit testing, integration testing, E2E testing, mocking, y patrones modulares. Se debe usar al escribir, revisar o refactorizar tests en proyectos NestJS.
license: MIT
metadata:
  version: "1.0.0"
  framework: "NestJS"
  testRunner: "Jest"
  language: "TypeScript"
  tags: ["nestjs", "jest", "testing", "typescript", "backend", "unit-test", "e2e", "integration"]
---

# NestJS Jest Testing - Skill

Skill completo para testing profesional de aplicaciones NestJS con Jest y TypeScript.

## Cuándo Aplicar

Usa este skill cuando:
- Escribas nuevos tests para servicios, controladores, guards, pipes, interceptors
- Configures el entorno de testing en un proyecto NestJS
- Implementes tests unitarios, de integración o E2E
- Necesites mockear dependencias externas (bases de datos, APIs, servicios)
- Refactorices tests existentes
- Optimices la cobertura de código
- Configures CI/CD con tests automatizados

## Categorías de Mejores Prácticas

| Prioridad | Categoría | Impacto | Prefijo |
|-----------|-----------|---------|---------|
| 1 | Estructura y Organización | CRÍTICO | `structure-` |
| 2 | Unit Testing | CRÍTICO | `unit-` |
| 3 | Mocking y Spies | ALTO | `mock-` |
| 4 | Integration Testing | ALTO | `integration-` |
| 5 | E2E Testing | MEDIO-ALTO | `e2e-` |
| 6 | Database Testing | MEDIO | `db-` |
| 7 | Optimización y Performance | MEDIO | `perf-` |
| 8 | CI/CD y Cobertura | BAJO-MEDIO | `ci-` |

## Referencia Rápida

### Estructura y Organización
- Colocar tests junto a archivos fuente (`.spec.ts`)
- Usar nomenclatura consistente `*.spec.ts` para unit tests
- Usar nomenclatura `*.e2e-spec.ts` para tests E2E
- Organizar setup en `beforeEach()` y cleanup en `afterEach()`

### Unit Testing
- Usar `Test.createTestingModule()` para módulos aislados
- Mockear todas las dependencias externas
- Testear un solo comportamiento por test
- Usar `describe()` para agrupar tests relacionados

### Mocking
- Usar `jest.fn()` para funciones mock
- Crear factories de mocks reutilizables
- Mockear repositorios y servicios externos
- Usar `jest.spyOn()` para espiar métodos

### Integration Testing
- Testear interacción entre múltiples componentes
- Usar módulos reales donde sea posible
- Mockear solo dependencias externas (DB, APIs)

### E2E Testing
- Usar Supertest para requests HTTP
- Configurar base de datos de testing aislada
- Limpiar estado entre tests
- Testear flujos completos de la aplicación

## Configuración Básica

```typescript
// jest.config.js
module.exports = {
  moduleFileExtensions: ['js', 'json', 'ts'],
  rootDir: 'src',
  testRegex: '.*\\.spec\\.ts$',
  transform: {
    '^.+\\.(t|j)s$': 'ts-jest',
  },
  collectCoverageFrom: [
    '**/*.(t|j)s',
  ],
  coverageDirectory: '../coverage',
  testEnvironment: 'node',
};
```

## Documentación Completa

Para guía detallada con ejemplos de código: `AGENTS.md`

## Referencias

- [NestJS Testing Documentation](https://docs.nestjs.com/fundamentals/testing)
- [Jest Documentation](https://jestjs.io/docs/getting-started)
- [TypeScript Jest](https://kulshekhar.github.io/ts-jest/)