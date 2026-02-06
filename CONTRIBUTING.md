# Contributing to Skills Hub 🤝

Thank you for your interest in contributing to Skills Hub! This document provides guidelines and instructions for contributing to this project.

## 📋 Table of Contents

- [Code of Conduct](#code-of-conduct)
- [How Can I Contribute?](#how-can-i-contribute)
- [Skill Structure Guidelines](#skill-structure-guidelines)
- [Pull Request Process](#pull-request-process)
- [Style Guidelines](#style-guidelines)
- [Community](#community)

## Code of Conduct

This project adheres to a [Code of Conduct](CODE_OF_CONDUCT.md). By participating, you are expected to uphold this code. Please report unacceptable behavior to the project maintainers.

## How Can I Contribute?

### 1. Creating a New Skill

To create a new skill:

1. **Fork the repository**
2. **Create a new branch**: `git checkout -b skill/your-skill-name`
3. **Create a new directory** in the root: `your-skill-name/`
4. **Create two required files**:
   - `SKILL.md` - Metadata and quick reference
   - `AGENTS.md` - Complete guide with examples

5. **Follow the structure** outlined in [Skill Structure Guidelines](#skill-structure-guidelines)
6. **Commit your changes**: `git commit -m "Add [Your Skill Name] skill"`
7. **Push to your fork**: `git push origin skill/your-skill-name`
8. **Open a Pull Request**

### 2. Improving Existing Skills

To improve an existing skill:

1. **Fork the repository**
2. **Create a new branch**: `git checkout -b improve/skill-name-improvement`
3. **Make your changes** to the skill files
4. **Test your examples** to ensure they work correctly
5. **Commit with descriptive messages**: `git commit -m "Improve error handling examples in X skill"`
6. **Push to your fork**: `git push origin improve/skill-name-improvement`
7. **Open a Pull Request** with a clear description of improvements

### 3. Reporting Issues

Found a problem? Help us improve:

- Use the [GitHub Issues](../../issues) page
- Check if the issue already exists
- Provide a clear title and description
- Include relevant code examples if applicable
- Tag appropriately (bug, enhancement, documentation, etc.)

### 4. Suggesting Enhancements

Have an idea for a new skill or improvement?

- Open an issue with the `enhancement` label
- Describe the skill topic or enhancement clearly
- Explain why it would be valuable
- Provide examples or use cases if possible

## Skill Structure Guidelines

### SKILL.md Template

```markdown
---
name: skill-name
description: Brief description of what this skill covers
license: MIT
metadata:
  version: "1.0.0"
  framework: "Framework name (if applicable)"
  language: "TypeScript"
  tags: ["tag1", "tag2", "tag3"]
---

# Skill Name

Brief introduction to the skill.

## When to Apply

List of scenarios when this skill should be used.

## Categories by Priority

| Priority | Category | Impact | Prefix |
|----------|----------|--------|--------|
| 1 | Category Name | CRITICAL | `prefix-` |

## Quick Reference

Brief overview of main practices.

## Complete Documentation

Link to AGENTS.md

## References

- Links to official documentation
- Other relevant resources
```

### AGENTS.md Template

```markdown
# Skill Name - Complete Guide

Detailed introduction.

## Table of Contents

1. [Category 1](#1-category-1)
2. [Category 2](#2-category-2)

---

## 1. Category Name

### rule-name
**Impact: CRITICAL|HIGH|MEDIUM|LOW**

Brief explanation of the rule.

**Incorrect:**
\```typescript
// Bad example with explanation
\```

**Correct:**
\```typescript
// Good example with explanation
\```

**Why:** Explanation of why this matters.

---

[Repeat for each rule]

## Summary

Key takeaways and command references.

## References

- Official documentation links
- Additional resources
```

## Pull Request Process

### Before Submitting

1. **Test all code examples** - Ensure examples compile and run
2. **Check formatting** - Use consistent markdown formatting
3. **Verify links** - Ensure all links work correctly
4. **Run spell check** - Check for typos and grammar
5. **Review changes** - Read through your changes one final time

### PR Guidelines

1. **Title**: Use clear, descriptive titles
   - ✅ "Add React Testing Best Practices skill"
   - ✅ "Fix TypeScript examples in NestJS skill"
   - ❌ "Update stuff"
   - ❌ "Changes"

2. **Description**: Include:
   - What changes were made
   - Why the changes are needed
   - Any related issues (use `Closes #123`)
   - Screenshots if applicable

3. **Size**: Keep PRs focused and manageable
   - One skill per PR for new skills
   - Related changes together for improvements
   - Break large changes into multiple PRs

4. **Reviews**: Be responsive to feedback
   - Address review comments promptly
   - Ask questions if something is unclear
   - Be open to suggestions

### PR Template

```markdown
## Description
Brief description of changes

## Type of Change
- [ ] New skill
- [ ] Skill improvement
- [ ] Bug fix
- [ ] Documentation update

## Checklist
- [ ] Code examples tested and working
- [ ] Follows skill structure guidelines
- [ ] Markdown formatting is correct
- [ ] Links are valid
- [ ] Spell checked
- [ ] Self-reviewed changes

## Related Issues
Closes #(issue number)
```

## Style Guidelines

### Markdown

- Use consistent heading levels
- Use code blocks with language specification
- Use tables for structured data
- Keep line length reasonable (80-120 characters)
- Use bullet points for lists
- Add blank lines between sections

### Code Examples

- **Always include both incorrect and correct examples**
- Add comments explaining key points
- Use TypeScript for type safety
- Follow current best practices
- Keep examples concise but complete
- Test that examples actually work

### Language

- Use clear, concise language
- Avoid jargon when possible
- Explain technical terms when necessary
- Write in second person ("you should...")
- Use active voice
- Be professional and respectful

### TypeScript Guidelines

- Use strict TypeScript settings
- Prefer interfaces over types when possible
- Use explicit types for clarity
- Follow ESLint recommended rules
- Use modern ES6+ features
- Avoid `any` type unless absolutely necessary

## Community

### Getting Help

- **Questions**: Open a [Discussion](../../discussions)
- **Issues**: Use [GitHub Issues](../../issues)
- **Chat**: Join our community chat (if available)

### Recognition

Contributors will be:
- Listed in the repository's contributors
- Acknowledged in release notes for significant contributions
- Given credit in any related documentation

## Review Process

1. **Initial Review**: Maintainers review within 3-5 business days
2. **Feedback**: Comments and suggestions provided
3. **Iteration**: Author addresses feedback
4. **Approval**: Once approved by maintainers
5. **Merge**: Changes merged into main branch

## Questions?

Don't hesitate to ask questions:
- Open a [Discussion](../../discussions)
- Comment on an existing issue
- Reach out to maintainers

---

Thank you for contributing to Skills Hub! Your expertise helps the entire community grow. 🚀