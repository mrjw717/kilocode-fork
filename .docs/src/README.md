# Source Code Documentation (`src/`)

## Overview

The `src/` directory contains the core implementation of the Kilo Code VS Code extension. This is where the extension's main business logic, UI components, services, and integrations reside.

## Directory Structure

```
src/
├── extension.ts              # Main entry point - Extension activation
├── core/                     # Core business logic and AI agent functionality
├── services/                 # Service implementations (code index, MCP, etc.)
├── api/                      # API providers and model transformers
├── integrations/             # VS Code integrations (editor, terminal, etc.)
├── activate/                 # Extension activation and command registration
├── i18n/                     # Internationalization and localization
├── shared/                   # Shared utilities and type definitions
├── utils/                    # Helper functions and utilities
├── workers/                  # Background worker processes
├── walkthrough/              # First-run experience content
├── __mocks__/                # Test mocks
├── __tests__/                # Unit tests
└── assets/                   # Static assets (icons, images)
```

## Main Entry Point: `extension.ts`

### Purpose

The `extension.ts` file is the main entry point for the VS Code extension. It contains the `activate()` and `deactivate()` functions that VS Code calls when the extension is loaded and unloaded.

### Activation Flow

1. **Environment Setup**

    - Loads `.env` configuration using `@dotenvx/dotenvx`
    - Creates output channel for logging
    - Initializes path utilities for cross-platform compatibility

2. **Service Initialization**

    ```typescript
    - TelemetryService: Analytics and metrics collection
    - CloudService: Cloud synchronization and authentication
    - MdmService: Mobile Device Management integration
    - TerminalRegistry: Terminal command execution handlers
    - CodeIndexManager: Code intelligence and indexing
    ```

3. **Provider Setup**

    - Creates `ClineProvider` webview instance
    - Registers webview view provider with VS Code
    - Sets up message passing between extension and webview

4. **Command Registration**

    - Registers VS Code commands (`kilo-code.*`)
    - Sets up code actions and quick fixes
    - Configures terminal actions
    - Registers URI handlers for deep linking

5. **First-Run Experience**

    - Detects first installation via global state
    - Opens walkthrough tutorial for new users
    - Focuses sidebar panel for immediate use

6. **Integration Registration**
    - Diff view provider for code comparison
    - Ghost text provider for inline suggestions
    - Commit message provider for Git integration
    - MCP server manager for extensibility

### Key Components Initialized

#### TelemetryService

- Tracks feature usage and user interactions
- Sends analytics to PostHog
- Provides crash reporting and error tracking
- Respects user privacy settings

#### CloudService

- Handles user authentication (login/logout)
- Synchronizes settings across devices
- Manages API key storage
- Provides remote task control capabilities

#### ContextProxy

- Abstracts VS Code configuration access
- Provides reactive configuration updates
- Handles workspace vs. global settings
- Manages MCP server configurations

#### ClineProvider

- Main webview UI controller
- Message routing between UI and extension
- State management for conversations
- Task lifecycle management

### Deactivation

The extension cleans up resources when deactivated:

- Disposes all registered commands
- Closes active terminal sessions
- Saves conversation state
- Releases service resources

## Folder-Specific Documentation

### [Core (`core/`)](./core/README.md)

Contains the AI agent's core business logic, prompt engineering, tool implementations, and conversation management.

**Key Responsibilities:**

- AI model interaction and prompt generation
- Tool execution (file editing, command running, browsing)
- Conversation history and checkpointing
- Task management and state tracking
- Mode switching (Architect, Coder, Debugger)

### [Services (`services/`)](./services/README.md)

Service layer providing specialized functionality to the core system.

**Key Responsibilities:**

- Code indexing and semantic search
- MCP (Model Context Protocol) server management
- Browser automation (Playwright, Puppeteer)
- Commit message generation
- Tree-sitter parsing for code intelligence
- Auto-purge for conversation cleanup

### [API (`api/`)](./api/README.md)

Abstracts different AI model providers and handles API communication.

**Key Responsibilities:**

- Provider implementations (Anthropic, OpenRouter, OpenAI, etc.)
- Model configuration and settings
- Request/response transformation
- Token usage tracking
- Streaming response handling

### [Integrations (`integrations/`)](./integrations/README.md)

Bridges Kilo Code with VS Code's native features and external tools.

**Key Responsibilities:**

- Editor integration (text editing, selection)
- Terminal management and command execution
- Workspace tracking and file watching
- Theme detection and UI adaptation
- Git integration and repository management

### [Activate (`activate/`)](./activate/README.md)

Handles extension activation, command registration, and VS Code integration setup.

**Key Responsibilities:**

- Command registration (`kilo-code.*` commands)
- Code action providers (quick fixes)
- Terminal action providers
- URI handling for deep links
- Task creation from external triggers

### [i18n (`i18n/`)](./i18n/README.md)

Internationalization and localization support for 20+ languages.

**Key Responsibilities:**

- Translation string management
- Dynamic language switching
- Locale detection and fallback
- Date/time formatting
- Number and currency localization

### [Shared (`shared/`)](./shared/README.md)

Common utilities, types, and constants used across the extension.

**Key Responsibilities:**

- TypeScript type definitions
- Constants and configuration defaults
- Utility functions (array, string, date helpers)
- Shared business logic
- Extension messaging types

### [Utils (`utils/`)](./utils/README.md)

Helper functions for common operations.

**Key Responsibilities:**

- File system operations
- Path manipulation (cross-platform)
- Git operations
- Output channel logging
- Settings migration
- Auto-import functionality

### [Workers (`workers/`)](./workers/README.md)

Background processes for resource-intensive operations.

**Key Responsibilities:**

- Text-to-speech generation
- Large file processing
- Background indexing
- Parallel task execution

### [Walkthrough (`walkthrough/`)](./walkthrough/README.md)

First-run tutorial content for new users.

**Key Responsibilities:**

- Step-by-step onboarding
- Feature introduction
- Interactive examples
- Tips and best practices

## Communication Patterns

### Extension Host ↔ Webview Communication

```typescript
// Extension → Webview
provider.postMessageToWebview({
	type: "state",
	state: extensionState,
})

// Webview → Extension
vscode.postMessage({
	type: "newTask",
	text: "Create a login form",
})
```

### Service Communication

Services communicate through:

- **Direct imports**: For synchronous operations
- **Event emitters**: For asynchronous notifications
- **VS Code events**: For IDE-triggered actions

### Tool Execution Flow

1. User sends message to webview
2. Webview forwards to `ClineProvider`
3. `ClineProvider` creates/resumes `Task`
4. `Task` interacts with AI model via `buildApiHandler`
5. AI returns tool calls
6. Tools execute via registered handlers in `core/tools/`
7. Results sent back to AI for next step
8. UI updates via webview messages

## State Management

### Global State (Persisted)

- User preferences and settings
- API keys (encrypted in secure storage)
- Conversation history metadata
- First-run flags
- Allowed commands list

### Workspace State (Per-Project)

- Active task information
- Workspace-specific settings
- File index cache
- MCP server configurations

### Session State (In-Memory)

- Active task instance
- Current conversation context
- Open terminal processes
- Browser automation sessions
- Streaming response buffers

## File Organization Principles

### Naming Conventions

- **PascalCase**: Classes and components (`ClineProvider.ts`)
- **camelCase**: Functions and variables (`buildApiHandler.ts`)
- **kebab-case**: File names with multiple words (`code-index/`)
- **UPPER_CASE**: Constants and enum values

### Module Boundaries

- Each folder has clear responsibilities
- No circular dependencies between core modules
- Services don't depend on UI components
- Utils are pure functions with no side effects

### Test Organization

- Tests colocated with source in `__tests__/` folders
- Mirror production file structure
- Unit tests for business logic
- Integration tests for VS Code APIs

## Performance Considerations

### Bundle Size

- Extension code split from webview code
- Lazy loading of heavy dependencies
- Tree shaking to eliminate unused code
- Minification in production builds

### Memory Management

- Conversation checkpointing to avoid memory leaks
- Automatic cleanup of old tasks
- Debounced file watchers
- Streaming for large responses

### Initialization Speed

- Asynchronous service initialization
- Deferred loading of non-critical features
- Code index builds in background
- Incremental UI rendering

## Error Handling

### Extension-Level Errors

- Logged to output channel
- Shown as VS Code notifications
- Sent to telemetry (if enabled)
- Graceful degradation of features

### Task-Level Errors

- Displayed in conversation UI
- Allow user to retry or modify request
- Logged for debugging
- Suggestions for resolution

### Service-Level Errors

- Automatic retry with exponential backoff
- Fallback to alternative implementations
- Clear error messages to user
- Health check indicators

## Security Model

### Sandboxing

- Extension runs in Node.js extension host
- Webview runs in isolated iframe
- Limited VS Code API access
- Command execution requires allowlist

### Data Protection

- API keys in VS Code secure storage
- Sensitive data not logged
- Code snippets sanitized before transmission
- User opt-in for telemetry

### Command Safety

- User confirmation for destructive operations
- Allowlist for auto-executable commands
- Path validation for file operations
- Git integration respects `.gitignore`

## Extensibility Points

### Custom Modes

Users can define custom agent behaviors:

```json
{
	"name": "Code Reviewer",
	"slug": "reviewer",
	"description": "Reviews code for best practices",
	"instructions": "Focus on code quality and maintainability..."
}
```

### MCP Servers

External tools via Model Context Protocol:

- Custom prompts
- Resource providers
- Tool implementations
- State management

### VS Code Integration

- Commands (`contributes.commands`)
- Keybindings (`contributes.keybindings`)
- Settings (`contributes.configuration`)
- Views (`contributes.views`)

## Development Best Practices

### Code Quality

- TypeScript strict mode enabled
- ESLint rules enforced
- Prettier for consistent formatting
- Pre-commit hooks for validation

### Testing Strategy

- Unit tests for business logic (>70% coverage target)
- Integration tests for VS Code APIs
- E2E tests for critical user flows
- Manual testing checklist for releases

### Documentation

- TSDoc comments for public APIs
- README in each major folder
- Architecture decision records (ADRs)
- Inline comments for complex logic

### Version Control

- Feature branches from `main`
- Conventional commit messages
- Changesets for version management
- Squash merges for clean history

## Common Issues and Solutions

### Extension Not Loading

- Check VS Code version compatibility (`^1.84.0`)
- Verify Node.js version (20.19.2)
- Review output channel for errors
- Try `Developer: Reload Window`

### Webview Not Updating

- Check for console errors in webview DevTools
- Verify message passing between extension and UI
- Ensure provider is properly registered
- Try killing and restarting extension development host

### Performance Issues

- Check code index build status
- Review active terminal processes
- Monitor memory usage in task manager
- Disable unnecessary MCP servers

### Build Failures

- Run `pnpm install` to update dependencies
- Clear `node_modules` and reinstall
- Check for TypeScript errors with `pnpm check-types`
- Review ESLint output with `pnpm lint`

## Next Steps

For detailed documentation on specific components:

- [Core System Documentation](./core/README.md)
- [Services Documentation](./services/README.md)
- [API Documentation](./api/README.md)
- [Integrations Documentation](./integrations/README.md)

## Contributing to Source Code

When contributing to the `src/` directory:

1. Follow the established folder structure
2. Add tests for new functionality
3. Update documentation for API changes
4. Run linters and type checkers before committing
5. Consider backwards compatibility
6. Add changeset for version management
