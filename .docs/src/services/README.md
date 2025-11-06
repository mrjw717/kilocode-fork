# Services Documentation (`src/services/`)

## Overview

The `services/` directory contains specialized service implementations that provide infrastructure and supporting functionality to the core AI agent system. Services are typically singleton instances that manage specific domains of functionality.

## Directory Structure

```
services/
├── auto-purge/              # Automatic cleanup of old conversations
├── browser/                 # Browser automation (Playwright, Puppeteer)
├── checkpoints/             # Checkpoint service implementations
├── code-index/              # Code intelligence and semantic search
├── command/                 # Command execution management
├── commit-message/          # AI-powered Git commit messages
├── continuedev/             # Continue.dev integration (legacy)
├── ghost/                   # Ghost text inline suggestions
├── glob/                    # File pattern matching utilities
├── marketplace/             # MCP server marketplace integration
├── mcp/                     # Model Context Protocol server management
├── mdm/                     # Mobile Device Management integration
├── mocking/                 # Test mocking utilities
├── ripgrep/                 # Fast code search integration
├── roo-config/              # Roo Code configuration service
├── search/                  # Search abstraction layer
├── terminal-welcome/        # Terminal welcome messages
└── tree-sitter/             # Code parsing and AST generation
```

## Core Services

### Code Index Service (`code-index/`)

Provides fast, intelligent code search and navigation across the workspace.

#### Purpose

- **Semantic Search**: Find code by meaning, not just keywords
- **Symbol Extraction**: Index functions, classes, variables
- **File Discovery**: Quickly locate files by name or pattern
- **Definition Lookup**: Find where symbols are defined
- **Reference Tracking**: Find all usages of a symbol

#### Architecture

```typescript
class CodeIndexManager {
	private fileIndex: Map<string, FileMetadata>
	private symbolIndex: Map<string, SymbolInfo[]>
	private searchIndex: SearchIndex

	async initialize(workspace: string)
	async indexFile(filePath: string)
	async search(query: string): SearchResult[]
	async findDefinitions(symbol: string): Location[]
}
```

#### Indexing Strategy

1. **Initial Scan**: Full workspace indexing on startup
2. **Incremental Updates**: File watcher for real-time updates
3. **Debounced Rebuilds**: Batch multiple file changes
4. **Priority Queue**: Index open files first
5. **Background Processing**: Non-blocking indexing

#### Search Capabilities

- **Full-Text Search**: Fast grep-like search with ripgrep
- **Fuzzy Matching**: Find symbols with typos
- **Regex Support**: Advanced pattern matching
- **Scope Filtering**: Search within specific directories
- **Language Awareness**: Syntax-aware search

#### Performance Optimizations

- **Incremental Indexing**: Only re-index changed files
- **Memory-Mapped Files**: Efficient large file handling
- **LRU Caching**: Cache frequently accessed files
- **Parallel Processing**: Multi-threaded indexing
- **Index Persistence**: Save/restore index state

#### Integration with AI

The code index powers several AI tools:

- `codebaseSearchTool`: Semantic code search
- `listCodeDefinitionNamesTool`: Symbol extraction
- `listFilesTool`: File discovery
- Context gathering for prompts

### MCP Service (`mcp/`)

Manages Model Context Protocol servers for extensibility.

#### Purpose

Model Context Protocol (MCP) allows external tools and data sources to extend the AI agent's capabilities.

#### Components

**McpServerManager**

- Discovers and registers MCP servers
- Manages server lifecycle (start/stop)
- Handles server configuration
- Monitors server health

**McpHub**

- Central registry of MCP servers
- Routes tool calls to appropriate servers
- Aggregates resources from multiple servers
- Handles inter-server communication

#### MCP Server Types

1. **Tool Servers**: Provide custom tools for the AI
2. **Resource Servers**: Expose data sources (databases, APIs)
3. **Prompt Servers**: Provide prompt templates
4. **State Servers**: Maintain persistent state

#### Configuration

```json
{
	"mcpServers": {
		"filesystem": {
			"command": "npx",
			"args": ["-y", "@modelcontextprotocol/server-filesystem"],
			"env": {
				"ALLOWED_DIRECTORIES": "/workspace"
			}
		},
		"postgres": {
			"command": "npx",
			"args": ["-y", "@modelcontextprotocol/server-postgres"],
			"env": {
				"DATABASE_URL": "postgresql://..."
			}
		}
	}
}
```

#### Server Lifecycle

```
Initialize → Start → Ready → Serving → Stop → Cleanup
```

#### Error Handling

- Automatic restart on crashes
- Fallback to built-in tools
- Health checks every 30 seconds
- User notification on failures

### Browser Service (`browser/`)

Automates browser interactions for web scraping, testing, and automation.

#### Purpose

- **Web Scraping**: Extract data from websites
- **UI Testing**: Automated browser testing
- **Screenshot Capture**: Visual documentation
- **Form Interaction**: Fill and submit forms
- **Navigation**: Multi-page workflows

#### Implementations

**BrowserSession (Playwright)**

- Modern, reliable browser automation
- Cross-browser support (Chrome, Firefox, Safari)
- Network interception
- Mobile emulation
- Video recording

**UrlContentFetcher**

- Lightweight content fetching
- Markdown conversion of web pages
- Screenshot capture
- PDF generation
- Link extraction

#### Browser Actions

```typescript
interface BrowserAction {
	// Navigation
	navigate(url: string): Promise<void>
	goBack(): Promise<void>
	goForward(): Promise<void>

	// Interaction
	click(selector: string): Promise<void>
	type(selector: string, text: string): Promise<void>
	select(selector: string, value: string): Promise<void>

	// Data Extraction
	getText(selector: string): Promise<string>
	getAttribute(selector: string, attr: string): Promise<string>
	screenshot(options?: ScreenshotOptions): Promise<Buffer>

	// Waiting
	waitForSelector(selector: string): Promise<void>
	waitForNavigation(): Promise<void>
}
```

#### Security Considerations

- Sandboxed browser instances
- User data isolation
- Cookie management
- HTTPS enforcement
- Content filtering

### Commit Message Service (`commit-message/`)

Generates intelligent Git commit messages using AI.

#### Purpose

Automatically generate descriptive, conventional commit messages based on staged changes.

#### Features

- **Conventional Commits**: Follows standard format
- **Change Analysis**: Understands what changed and why
- **Multiple Suggestions**: Offers alternatives
- **Context Awareness**: Considers commit history
- **Custom Templates**: User-defined formats

#### Commit Message Format

```
<type>(<scope>): <subject>

<body>

<footer>
```

#### Types Supported

- `feat`: New feature
- `fix`: Bug fix
- `docs`: Documentation changes
- `style`: Code style changes
- `refactor`: Code refactoring
- `test`: Test additions/changes
- `chore`: Build/tooling changes

#### Generation Process

1. **Diff Analysis**: Parse git diff
2. **Change Classification**: Categorize changes
3. **Context Gathering**: Review recent commits
4. **AI Generation**: Request commit message
5. **Validation**: Ensure conventional format
6. **Presentation**: Show to user for approval

#### Integration

Provides VS Code Source Control integration:

```typescript
vscode.commands.registerCommand("kilo-code.generateCommitMessage", async () => {
	const message = await generateCommitMessage()
	scm.inputBox.value = message
})
```

### Tree-Sitter Service (`tree-sitter/`)

Provides robust code parsing using Tree-sitter parsers.

#### Purpose

- **Syntax Parsing**: Parse code into Abstract Syntax Trees (AST)
- **Symbol Extraction**: Find functions, classes, variables
- **Syntax Highlighting**: Accurate highlighting
- **Code Navigation**: Jump to definitions
- **Refactoring Support**: Structural code changes

#### Supported Languages

- JavaScript/TypeScript
- Python
- Rust
- Go
- Java
- C/C++
- Ruby
- PHP
- More via Tree-sitter grammars

#### Capabilities

**Symbol Extraction**

```typescript
interface Symbol {
	name: string
	kind: "function" | "class" | "variable" | "method"
	location: Location
	signature?: string
	docstring?: string
}

const symbols = await treeSitter.extractSymbols(filePath)
```

**Syntax Queries**

```typescript
// Find all function declarations
const query = `
  (function_declaration
    name: (identifier) @name
    parameters: (formal_parameters) @params
  )
`
const matches = treeSitter.query(filePath, query)
```

**Code Modification**
Tree-sitter enables precise code edits:

- Insert at specific AST nodes
- Replace syntax elements
- Maintain formatting
- Preserve comments

### Auto-Purge Service (`auto-purge/`)

Automatically cleans up old conversation data to manage storage.

#### Purpose

- **Storage Management**: Prevent unlimited growth
- **Performance**: Faster searches and loads
- **Privacy**: Remove old sensitive data
- **Compliance**: Data retention policies

#### Purge Strategy

```typescript
interface PurgePolicy {
	maxAge: number // Days to keep
	maxCount: number // Max conversations
	excludeBookmarked: boolean // Keep bookmarked
	excludeIncomplete: boolean // Keep active tasks
}
```

#### Default Policy

- Keep conversations < 30 days old
- Keep last 100 conversations
- Always keep bookmarked
- Always keep incomplete tasks
- Weekly purge schedule

#### Purge Process

1. **Identify Candidates**: Find purgeable conversations
2. **Export Option**: Offer to export before deletion
3. **Soft Delete**: Mark as deleted (recoverable)
4. **Hard Delete**: Remove from storage (after 7 days)
5. **Index Cleanup**: Update search indices

### Checkpoint Service (`checkpoints/`)

Manages conversation state checkpoints for rollback.

#### Purpose

Enable users to save and restore conversation states, allowing exploration of alternative approaches without losing work.

#### Service Implementations

**RepoPerTaskCheckpointService**

- One checkpoint repo per task
- Git-based storage
- Efficient diffs
- Branch-based experimentation

**ShadowCheckpointService**

- Automatic background checkpoints
- No user interaction needed
- Rollback on errors
- Limited retention (last 10)

#### Checkpoint Operations

**Save Checkpoint**

```typescript
await checkpointService.save({
	taskId: task.id,
	label: "Before refactoring",
	includeFileSystem: true,
	includeTerminal: true,
})
```

**Restore Checkpoint**

```typescript
await checkpointService.restore({
	checkpointId: "checkpoint-123",
	restoreFiles: true,
	restoreTerminal: false,
})
```

**Diff Checkpoints**

```typescript
const diff = await checkpointService.diff({
	fromCheckpoint: "checkpoint-123",
	toCheckpoint: "current",
})
```

#### Storage Backend

- Git repositories for file states
- SQLite for metadata
- JSON for conversation history
- Compression for large files

### Ghost Text Service (`ghost/`)

Provides inline code suggestions as ghost text (grayed-out text).

#### Purpose

Real-time code completion suggestions that appear as you type.

#### Features

- **Context-Aware**: Considers surrounding code
- **Multi-Line**: Suggests entire code blocks
- **Smart Triggers**: Activates at appropriate times
- **Fast**: Low-latency suggestions (<200ms)
- **Unobtrusive**: Easy to accept or ignore

#### Trigger Conditions

- After typing a comment
- After opening a bracket/brace
- After a newline in function body
- After import/require statements
- On demand (Ctrl+Space)

#### Acceptance Methods

- **Tab**: Accept entire suggestion
- **Ctrl+→**: Accept word-by-word
- **Escape**: Dismiss suggestion
- **Keep typing**: Auto-dismiss

#### AI Integration

Uses smaller, faster models for low latency:

- GPT-3.5 Turbo
- Claude Instant
- CodeLlama
- Phi-2 (local)

### Marketplace Service (`marketplace/`)

Manages MCP server discovery and installation from marketplace.

#### Purpose

- **Discovery**: Browse available MCP servers
- **Installation**: One-click server setup
- **Updates**: Automatic version checking
- **Ratings**: Community feedback
- **Documentation**: Server usage guides

#### Marketplace Sources

- Official MCP server registry
- Community contributions
- Private organization servers
- Local custom servers

#### Server Metadata

```typescript
interface MarketplaceServer {
	id: string
	name: string
	description: string
	author: string
	version: string
	category: string[]
	tags: string[]
	downloads: number
	rating: number
	repository: string
	documentation: string
	verified: boolean
}
```

### MDM Service (`mdm/`)

Mobile Device Management integration for enterprise deployments.

#### Purpose

- **Policy Enforcement**: Apply organization policies
- **Configuration Management**: Centralized settings
- **Compliance**: Meet regulatory requirements
- **Security**: Enhanced security controls

#### Supported Policies

- Allowed/blocked AI models
- Required MCP servers
- File access restrictions
- Command execution limits
- Network restrictions

## Service Patterns

### Singleton Pattern

Most services are singletons:

```typescript
class MyService {
	private static instance: MyService

	static getInstance(): MyService {
		if (!this.instance) {
			this.instance = new MyService()
		}
		return this.instance
	}

	private constructor() {}
}
```

### Event-Driven Communication

Services communicate via events:

```typescript
class MyService extends EventEmitter {
	start() {
		this.emit("started")
	}

	onError(error: Error) {
		this.emit("error", error)
	}
}

myService.on("started", () => console.log("Started"))
myService.on("error", (error) => console.error(error))
```

### Lifecycle Management

Services follow consistent lifecycle:

```typescript
interface Service {
	initialize(): Promise<void>
	start(): Promise<void>
	stop(): Promise<void>
	dispose(): Promise<void>
}
```

### Dependency Injection

Services receive dependencies via constructor:

```typescript
class MyService {
	constructor(
		private config: Config,
		private logger: Logger,
		private otherService: OtherService,
	) {}
}
```

## Performance Considerations

### Lazy Initialization

- Services initialized on first use
- Reduces startup time
- Lower memory footprint

### Caching

- Cache expensive operations
- TTL-based cache invalidation
- LRU eviction for memory management

### Background Processing

- Offload heavy work to background threads
- Non-blocking operations
- Progress reporting

### Resource Management

- Proper cleanup in dispose()
- Connection pooling
- File handle limits
- Memory leak prevention

## Error Handling

### Service-Level Errors

- Log errors with context
- Emit error events
- Graceful degradation
- User notification

### Recovery Strategies

- Automatic retry with backoff
- Fallback implementations
- Health checks
- Circuit breakers

## Testing Services

### Unit Tests

- Mock dependencies
- Test business logic
- Verify error handling
- Check edge cases

### Integration Tests

- Real dependencies
- End-to-end flows
- Performance benchmarks
- Resource usage monitoring

### Mock Services

Located in `services/mocking/`:

- Mock implementations for testing
- Predictable behavior
- Fast execution
- No external dependencies

## Configuration

### Service Configuration

Each service has configuration options:

```typescript
interface CodeIndexConfig {
	enabled: boolean
	maxFileSize: number
	excludePatterns: string[]
	debounceMs: number
}
```

### Configuration Sources

1. Default values in code
2. Extension settings
3. Workspace settings
4. Environment variables
5. Runtime overrides

## Future Enhancements

### Planned Services

- **Language Server Integration**: LSP support
- **Database Service**: SQL query execution
- **Cloud Storage Service**: S3, Azure Blob, etc.
- **Container Service**: Docker integration
- **Testing Service**: Automated test generation

### Improvements

- Service mesh for inter-service communication
- Distributed tracing
- Metrics collection
- Service health dashboard
- A/B testing framework

## Related Documentation

- [Code Index Deep Dive](./code-index/README.md)
- [MCP Integration Guide](./mcp/README.md)
- [Browser Automation Guide](./browser/README.md)
- [Tree-Sitter Usage](./tree-sitter/README.md)

## Contributing

When adding new services:

1. Follow singleton pattern for stateful services
2. Implement proper lifecycle methods
3. Add comprehensive error handling
4. Include unit and integration tests
5. Document configuration options
6. Update this README
