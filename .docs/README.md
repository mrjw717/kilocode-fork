# Kilo Code - Complete Documentation

## Overview

Kilo Code is an open-source AI coding assistant built as a Visual Studio Code extension. It provides autonomous code generation, task automation, debugging, and intelligent code assistance through integration with multiple AI models and Model Context Protocol (MCP) servers.

## Repository Information

- **Repository**: https://github.com/Kilo-Org/kilocode
- **Forked Version**: https://github.com/mrjw717/kilocodeprojectunicorn.git
- **Extension Type**: VS Code Workspace Extension
- **Current Version**: 4.115.0
- **Minimum VS Code Version**: 1.84.0
- **Node Version**: 20.19.2

## Core Capabilities

### Primary Features

- **Autonomous Code Generation**: Generate code from natural language descriptions
- **Multi-Mode Operations**: Plan (Architect), Code (Coder), Debug (Debugger), Custom Modes
- **Task Automation**: Automate repetitive coding workflows
- **Browser Automation**: Control and automate browser interactions
- **Terminal Command Execution**: Run and monitor terminal commands
- **MCP Server Integration**: Extensible through Model Context Protocol servers
- **Multi-Model Support**: Access to 400+ AI models (Gemini, Claude, GPT, etc.)
- **Code Refactoring**: Automated code improvement and restructuring
- **Self-Checking**: Agent validates its own work

### Technical Architecture

- **Extension Host Process**: Runs in isolated Node.js process
- **Webview UI**: React-based frontend for user interaction
- **RPC Communication**: Message passing between extension host and webview
- **Service-Oriented**: Modular service architecture for scalability
- **Cloud Integration**: Optional cloud synchronization and telemetry
- **Internationalization**: Support for 20+ languages

## Project Structure

```
kilocode/
├── src/                        # Main extension source code
│   ├── core/                   # Core functionality and business logic
│   ├── services/               # Service implementations
│   ├── api/                    # API providers and transformers
│   ├── integrations/           # IDE integrations (editor, terminal, etc.)
│   ├── activate/               # Extension activation and registration
│   ├── i18n/                   # Internationalization resources
│   ├── shared/                 # Shared utilities and types
│   ├── utils/                  # Utility functions
│   ├── workers/                # Background worker processes
│   ├── walkthrough/            # First-run walkthrough content
│   └── extension.ts            # Main extension entry point
│
├── webview-ui/                 # React-based UI components
├── apps/                       # Additional applications
│   ├── vscode-e2e/            # End-to-end tests
│   ├── playwright-e2e/        # Playwright test suite
│   ├── storybook/             # UI component documentation
│   ├── kilocode-docs/         # Documentation site
│   └── web-roo-code/          # Web version
│
├── packages/                   # Shared packages
│   ├── cloud/                 # Cloud service implementation
│   ├── telemetry/             # Telemetry and analytics
│   ├── ipc/                   # Inter-process communication
│   ├── types/                 # Shared TypeScript types
│   └── build/                 # Build utilities
│
├── jetbrains/                  # JetBrains IDE plugin
├── cli/                        # Command-line interface
├── benchmark/                  # Performance benchmarks
├── scripts/                    # Build and utility scripts
└── deps/                       # External dependencies

```

## Key Technologies

### Core Stack

- **TypeScript**: Primary language for type-safe development
- **Node.js**: Runtime environment (v20.19.2)
- **pnpm**: Package manager and monorepo management
- **esbuild**: Fast bundling for production builds
- **Vitest**: Testing framework for unit tests

### VS Code Integration

- **VS Code Extension API**: Core extension framework
- **Webview API**: For custom UI rendering
- **Language Server Protocol**: For code intelligence
- **Debug Adapter Protocol**: For debugging integration

### Frontend

- **React**: UI component library
- **Vite**: Development server and build tool
- **VS Code Webview UI Toolkit**: Native VS Code-styled components

### AI & ML Integration

- **Anthropic Claude**: Primary AI model provider
- **OpenRouter**: Multi-model API gateway
- **Model Context Protocol (MCP)**: Extensibility framework
- **LangChain**: AI workflow orchestration

### Development Tools

- **ESLint**: Code linting and style enforcement
- **Prettier**: Code formatting
- **Husky**: Git hooks for quality checks
- **Changesets**: Version management and changelog generation

## Extension Activation Flow

1. **Environment Loading**: Load `.env` configuration files
2. **Telemetry Initialization**: Setup PostHog and cloud telemetry
3. **Cloud Service Setup**: Initialize cloud authentication and sync
4. **Internationalization**: Load localized strings based on user language
5. **Terminal Registry**: Setup terminal command execution handlers
6. **Context Proxy**: Initialize configuration context manager
7. **Code Index Managers**: Setup code intelligence for workspace folders
8. **Provider Initialization**: Create main `ClineProvider` webview
9. **Command Registration**: Register VS Code commands and actions
10. **MCP Integration**: Initialize Model Context Protocol servers
11. **First-Run Experience**: Show walkthrough for new installations

## Development Workflow

### Setup Options

1. **Native Development**: Direct installation on MacOS/Linux/WSL
2. **Devcontainer**: Docker-based environment (Windows recommended)
3. **Nix Flake**: Declarative development environment (NixOS)

### Running the Extension

```bash
# Install dependencies
pnpm install

# Start development mode (F5 in VS Code)
# Launches Extension Development Host with hot reload

# Build production version
pnpm build

# Install built extension
code --install-extension "$(ls -1v bin/kilo-code-*.vsix | tail -n1)"
```

### Testing

```bash
# Unit tests
pnpm test

# E2E tests
pnpm --filter vscode-e2e test

# Type checking
pnpm check-types

# Linting
pnpm lint
```

## Security Considerations

### API Key Management

- API keys stored in VS Code secure storage
- Environment variables loaded from `.env` files
- No hardcoded credentials in source code
- Encrypted cloud synchronization for settings

### Command Execution Safety

- Allowlist for approved commands (`allowedCommands` configuration)
- User confirmation required for potentially destructive operations
- Sandboxed terminal execution environment
- Command output monitoring and logging

### File System Access

- Respects `.gitignore` patterns for file indexing
- Read-only access to virtual documents in diff view
- Workspace-scoped file operations
- Protected paths configurable via `filesNotToEdit`

## Configuration Management

### Settings Hierarchy

1. **Global Settings**: User-wide configuration in VS Code settings
2. **Workspace Settings**: Project-specific configuration
3. **Cloud Settings**: Synchronized settings via cloud service
4. **MCP Settings**: Model Context Protocol server configurations

### Key Configuration Options

- `kilo-code.allowedCommands`: Commands that can auto-execute
- `kilo-code.filesNotToEdit`: Protected file patterns
- `kilo-code.apiProvider`: AI model provider selection
- `kilo-code.customModes`: User-defined agent modes
- `kilo-code.mcpServers`: MCP server configurations

## Performance Optimization

### Code Indexing

- Incremental file indexing for large codebases
- File watcher for real-time index updates
- Lazy loading of code intelligence features
- Debounced index rebuilds on file changes

### Memory Management

- Webview context retained when hidden
- Checkpoint-based conversation history
- Automatic purging of old conversation data
- Streaming responses for large outputs

### Bundle Optimization

- Code splitting for webview and extension host
- Tree shaking to remove unused code
- Minification and compression for production
- Source maps for debugging

## Internationalization (i18n)

### Supported Languages

Arabic (ar), Catalan (ca), Czech (cs), German (de), Spanish (es), French (fr), Hindi (hi), Indonesian (id), Italian (it), Japanese (ja), Korean (ko), Dutch (nl), Polish (pl), Portuguese-BR (pt-BR), Russian (ru), Thai (th), Turkish (tr), Ukrainian (uk), Vietnamese (vi), Chinese-CN (zh-CN), Chinese-TW (zh-TW)

### Translation Structure

- Main translations in `src/i18n/locales/{lang}/`
- Package translations in `package.nls.{lang}.json`
- Runtime language selection based on VS Code locale
- Fallback to English for missing translations

## Telemetry and Analytics

### Data Collection

- Feature usage metrics via PostHog
- Error reporting and crash analytics
- Performance metrics (response times, token usage)
- User journey tracking (onboarding, feature adoption)

### Privacy Controls

- Opt-out available in settings
- Anonymized user identifiers
- No code content transmitted
- GDPR compliant data handling

## Extension Distribution

### VS Code Marketplace

- Published as `kilocode.Kilo-Code`
- Automatic updates via marketplace
- Public visibility for all users
- Version history and rollback support

### Installation Methods

1. **Marketplace**: Direct install from VS Code
2. **VSIX File**: Manual installation from `.vsix`
3. **CI/CD**: Automated builds via GitHub Actions
4. **Development**: Local debugging via Extension Host

## Architecture Patterns

### Service Pattern

- Singleton services for global state management
- Dependency injection for testability
- Event-driven communication between services
- Lifecycle management via VS Code subscriptions

### Provider Pattern

- Custom providers for VS Code features
- Webview view providers for UI panels
- Code action providers for quick fixes
- Completion providers for AI suggestions

### Proxy Pattern

- ContextProxy for configuration access
- API proxies for model provider abstraction
- RPC proxies for webview communication

## Future Considerations

### Scalability

- Multi-workspace support for monorepos
- Distributed code indexing for large projects
- Server-side processing for resource-intensive operations
- Caching strategies for repeated operations

### Extensibility

- Plugin system for custom tools
- Third-party MCP server marketplace
- Custom mode templates and sharing
- Integration with other IDEs (JetBrains, IntelliJ)

### Reliability

- Automatic error recovery mechanisms
- Offline mode for network interruptions
- Backup and restore for conversation history
- Health checks for service availability

## Related Documentation

- [Source Code Documentation](./src/README.md) - Detailed `src/` folder structure
- [Core System Documentation](./src/core/README.md) - Core business logic
- [Services Documentation](./src/services/README.md) - Service implementations
- [API Documentation](./src/api/README.md) - API providers and transformers
- [Architecture Documentation](./architecture/README.md) - System architecture
- [Development Guide](../DEVELOPMENT.md) - Setup and development workflow

## Contributing

Contributions are welcome! See the main repository for contribution guidelines, code of conduct, and development setup instructions.

## License

See the main repository for license information.
