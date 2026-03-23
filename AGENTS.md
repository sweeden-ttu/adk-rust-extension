# AGENTS.md - ADK Rust Extension Developer Guide

## Overview

This is a VS Code extension for building ADK-Rust agents. The extension provides project scaffolding, environment checking, build/run integration, and the ADK Studio visual builder.

## Project Structure

```
src/
├── extension.ts          # Main entry point, registers all commands
├── configManager.ts      # Settings management
├── environmentChecker.ts # Rust/ADK Studio verification
├── projectScaffolder.ts  # Project template generation
├── buildRunner.ts        # Cargo build/run execution
├── studioManager.ts      # ADK Studio server management
├── statusManager.ts      # Project status tracking
├── projectTreeProvider.ts# Sidebar tree view
├── logger.ts             # Logging utilities
├── messageBus.ts         # Inter-component messaging
├── sidebarFallback.ts    # Webview/native view switching
├── sidebarWebviewProvider.ts
├── sidebarMessageHandlers.ts
├── dataConverters.ts     # Type transformations
├── types.ts              # Shared type definitions
├── templates/            # Project template files
│   ├── simple-chat.ts
│   ├── tool-using-agent.ts
│   ├── multi-agent-workflow.ts
│   └── graph-workflow.ts
└── test/                 # Test suite
    ├── suite/            # Integration tests
    ├── mocks/             # Mock implementations
    ├── setup.ts
    └── testUtils.ts
```

## Build, Lint, and Test Commands

### Development

```bash
# Compile TypeScript
npm run compile

# Watch mode compilation
npm run watch

# Build extension bundle
npm run bundle

# Watch mode bundle (for development)
npm run bundle:watch
```

### Linting

```bash
# Run ESLint
npm run lint

# Fix ESLint issues
npm run lint:fix
```

### Testing

```bash
# Run all unit tests
npm run test

# Run tests in watch mode
npm run test:watch

# Run a single test file
npx mocha --require ts-node/register 'src/studioManager.test.ts'

# Run a single test (using --grep for test name)
npm run test -- --grep "test name pattern"

# Run integration tests
npm run test:integration
```

### Full Build

```bash
# Run before publishing (compiles + bundles)
npm run vscode:prepublish
```

## Code Style Guidelines

### TypeScript Configuration

- **Strict mode is enabled** in `tsconfig.json`
- **`noImplicitAny: true`** — never use implicit `any`
- Use explicit return types for public functions

### ESLint Rules

The project uses ESLint with typescript-eslint. Key rules:

| Rule | Setting | Notes |
|------|---------|-------|
| `@typescript-eslint/no-unused-vars` | `error` | Prefix unused params with `_` |
| `@typescript-eslint/no-explicit-any` | `error` | Never use `any` type |
| `@typescript-eslint/explicit-function-return-type` | `off` | Allow implicit returns |
| `no-console` | `warn` | Prefer logger over console |

### Imports

- Use **barrel exports** where appropriate (e.g., `./templates/index.ts`)
- Group imports: external → internal → types
- Use path aliases if defined in tsconfig

```typescript
// Example import ordering
import * as vscode from 'vscode';
import * as path from 'path';

import { ConfigurationManager } from './configManager';
import { StudioManager } from './studioManager';
import { ProjectTreeProvider, AdkProject, AdkTreeItem } from './projectTreeProvider';
```

### Naming Conventions

| Type | Convention | Example |
|------|------------|---------|
| Files | kebab-case | `studioManager.ts` |
| Classes | PascalCase | `StudioManager` |
| Functions | camelCase | `checkEnvironment` |
| Interfaces | PascalCase | `AdkProject` |
| Constants | UPPER_SNAKE_CASE | `DEFAULT_PORT` |
| Private members | `_camelCase` | `_dispose()` |

### Error Handling

- Use typed error objects where possible
- Wrap async operations in try/catch
- Log errors with the Logger utility before throwing
- Return `Result<T>` patterns for operations that can fail gracefully

```typescript
// Preferred pattern
try {
  await someOperation();
} catch (error) {
  logger.error('Operation failed', error);
  vscode.window.showErrorMessage('Operation failed: ' + (error as Error).message);
}
```

### VSCode Extension Patterns

- **Extension Context**: Always register disposables with `context.subscriptions`
- **Output Channels**: Reuse existing channels instead of creating new ones
- **Diagnostics**: Clear and update on each operation
- **Webviews**: Use `webview.asWebviewUri` for local resources

```typescript
// Register command with disposables
context.subscriptions.push(
  vscode.commands.registerCommand('adkRust.build', async () => {
    // implementation
  })
);
```

### Testing Patterns

- Test files: `*.test.ts` in the same directory as source
- Use mocks from `./test/mocks/` for VS Code APIs
- Follow the existing test structure in `src/test/suite/`

```typescript
import * as sinon from 'sinon';  // if needed
import { getMockVscode } from './mocks/vscode';

// Use the mock in tests
const vscode = getMockVscode();
```

### Logging

- Use the `Logger` class from `./logger.ts`
- Levels: `debug`, `info`, `warn`, `error`
- Never log secrets — use `maskSensitiveData()` for env vars

```typescript
const logger = getLogger();
logger.info('Extension activating');
logger.debug('Config:', settings);
```

### Configuration

Settings are defined in `package.json` under `contributes.configuration`. Access via `ConfigurationManager`:

```typescript
const configManager = new ConfigurationManager();
const settings = configManager.getSettings();
```

## Key Dependencies

- **esbuild** — Bundle for VS Code
- **mocha** — Test framework
- **typescript-eslint** — Linting
- **dotenv** — Environment file handling
- **@types/vscode** — VS Code API types

## Common Tasks

### Adding a New Command

1. Register in `package.json` `contributes.commands`
2. Add handler in `extension.ts` `activate()` function
3. Add tests in appropriate test file

### Adding a New Template

1. Create template file in `src/templates/`
2. Export from `src/templates/index.ts`
3. Update default template list in `package.json`

### Debugging

- Use VS Code debugger with `.vscode/launch.json`
- Check output channel "ADK Rust" for logs
- Enable verbose logging in settings

## VS Code API Patterns

- **Tree Views**: Use `vscode.createTreeView` or custom webview
- **Webviews**: HTML/CSS/JS in sandboxed iframe
- **Status Bar**: `vscode.StatusBarItem`
- **Notifications**: `vscode.window.show*Message`
- **Terminal**: `vscode.Terminal` for build output
