```markdown
# speq Development Patterns

> Auto-generated skill from repository analysis

## Overview
This skill teaches the core development patterns and conventions used in the `speq` TypeScript codebase. You'll learn about file naming, import/export styles, commit message conventions, and how to write and organize tests. This guide also provides suggested commands for common workflows to streamline your development process.

## Coding Conventions

### File Naming
- **Pattern:** PascalCase
- **Example:**  
  ```plaintext
  MyComponent.ts
  UserService.ts
  ```

### Import Style
- **Pattern:** Relative imports
- **Example:**  
  ```typescript
  import { UserService } from './UserService';
  import { calculateTotal } from '../utils/CalculateTotal';
  ```

### Export Style
- **Pattern:** Named exports
- **Example:**  
  ```typescript
  // In UserService.ts
  export function getUser(id: string) { ... }

  // In another file
  import { getUser } from './UserService';
  ```

### Commit Messages
- **Pattern:** Conventional commits with `feat` prefix
- **Example:**  
  ```
  feat: add user authentication logic
  ```

## Workflows

### Feature Development
**Trigger:** When implementing a new feature  
**Command:** `/feature-development`

1. Create a new TypeScript file using PascalCase for the filename.
2. Use relative imports to include dependencies.
3. Export your functions or classes using named exports.
4. Write or update corresponding test files (see Testing Patterns).
5. Commit your changes using the conventional commit format with a `feat` prefix.

### Testing
**Trigger:** When validating code functionality  
**Command:** `/run-tests`

1. Locate or create a test file matching the `*.test.*` pattern.
2. Write tests for your features or bug fixes.
3. Run your test suite using your preferred test runner (framework not specified; check project scripts or documentation).
4. Review test results and fix any failures.

## Testing Patterns

- **Test File Naming:**  
  Use the `*.test.*` pattern, e.g., `UserService.test.ts`.
- **Framework:**  
  Not specified—refer to project documentation or scripts.
- **Example:**  
  ```typescript
  // UserService.test.ts
  import { getUser } from './UserService';

  test('should return user by id', () => {
    const user = getUser('123');
    expect(user.id).toBe('123');
  });
  ```

## Commands
| Command               | Purpose                                    |
|-----------------------|--------------------------------------------|
| /feature-development  | Start a new feature with correct patterns  |
| /run-tests            | Run all tests in the codebase              |
```
