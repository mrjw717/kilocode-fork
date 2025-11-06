# Kilo Code Architecture Documentation

## System Overview

Kilo Code is a sophisticated AI coding assistant built as a VS Code extension. The architecture follows a modular, event-driven design with clear separation between the extension host process, webview UI, and external services.

## High-Level Architecture

```
┌─────────────────────────────────────────────────────────────┐
│                        VS Code IDE                          │
│  ┌───────────────────────────────────────────────────────┐  │
│  │           Extension Host Process (Node.js)            │  │
│  │  ┌─────────────────────────────────────────────────┐  │  │
│  │  │              Core System                        │  │  │
│  │  │  - Task Management                             │  │  │
│  │  │  - Tool Execution                              │  │  │
│  │  │  - Prompt Engineering                          │  │  │
│  │  │  - Context Management                          │  │  │
│  │  └─────────────────────────────────────────────────┘  │  │
│  │  ┌─────────────────────────────────────────────────┐  │  │
│  │  │              Services Layer                     │  │  │
│  │  │  - Code Index                                  │  │  │
│  │  │  - MCP Servers                                 │  │  │
│  │  │  - Browser Automation                          │  │  │
│  │  │  - Tree-Sitter Parsing                         │  │  │
│  │  └─────────────────────────────────────────────────┘  │  │
│  │  ┌─────────────────────────────────────────────────┐  │  │
│  │  │              API Layer                          │  │  │
│  │  │  - Provider Abstraction                        │  │  │
│  │  │  - Request/Response Transform                  │  │  │
│  │  │  - Streaming Handler                           │  │  │
│  │  └─────────────────────────────────────────────────┘  │  │
│  │  ┌─────────────────────────────────────────────────┐  │  │
│  │  │              Integrations                       │  │  │
│  │  │  - Editor (DiffView, Decorations)              │  │  │
│  │  │  - Terminal (Execution, Tracking)              │  │  │
│  │  │  - Workspace (File Watching)                   │  │  │
│  │  └─────────────────────────────────────────────────┘  │  │
│  └───────────────────────────────────────────────────────┘  │
│                                                             │
│  ┌───────────────────────────────────────────────────────┐  │
│  │             Webview Panel (React)                     │  │
│  │  - Chat Interface                                    │  │
│  │  - Message History                                   │  │
│  │  - Settings UI                                       │  │
│  │  - File Preview                                      │  │
│  └───────────────────────────────────────────────────────┘  │
└─────────────────────────────────────────────────────────────┘
                              │
                              ▼
         ┌────────────────────────────────────────┐
         │         External Services              │
         │  - AI Model APIs (Claude, GPT, etc.)  │
         │  - Cloud Service (Auth, Sync)         │
         │  - Telemetry (PostHog)                │
         │  - MCP Servers (External Tools)       │
         └────────────────────────────────────────┘
```

## Component Architecture

### Extension Host Process

The main extension runs in Node.js with access to VS Code APIs.

**Key Responsibilities:**

- Manage extension lifecycle
- Handle VS Code integration
- Execute commands and tools
- Maintain conversation state
- Coordinate services

**Process Isolation:**

- Runs separately from VS Code main process
- Can be restarted without reloading VS Code
- Communicates via IPC with VS Code

### Webview UI

React-based user interface running in isolated iframe.

**Key Responsibilities:**

- Render chat interface
- Display conversation history
- Handle user input
- Show real-time updates
- Manage UI state

**Communication:**

- Message passing with extension host
- No direct access to Node.js APIs
- Restricted DOM access
- Content Security Policy enforced

### Service Layer

Singleton services providing specialized functionality.

**Design Pattern:**

```typescript
class ServiceName {
	private static instance: ServiceName

	static getInstance(): ServiceName {
		if (!this.instance) {
			this.instance = new ServiceName()
		}
		return this.instance
	}

	private constructor() {
		// Initialize service
	}

	async initialize(): Promise<void> {
		// Async initialization
	}

	dispose(): void {
		// Cleanup resources
	}
}
```

## Data Flow Architecture

### Request Flow

```
User Input → Webview → Extension Host → Task → API → AI Model
                                         ↓
                                    Tool Execution
                                         ↓
                                    Tool Results
                                         ↓
AI Model → API → Extension Host → Task → Webview → User
```

### Detailed Flow

1. **User Input**

    - User types message in webview
    - Optionally includes images, @mentions
    - Clicks send button

2. **Message Routing**

    - Webview posts message to extension
    - Extension receives via `webview.onDidReceiveMessage`
    - Routes to `ClineProvider.handleWebviewMessage`

3. **Task Creation/Resume**

    - Create new `Task` instance or resume existing
    - Load conversation history
    - Gather context (files, diagnostics, etc.)

4. **Prompt Assembly**

    - Build system prompt
    - Add tool definitions
    - Include conversation history
    - Add environment details
    - Insert user message

5. **API Request**

    - Select appropriate provider
    - Transform to provider format
    - Send HTTP request
    - Handle streaming response

6. **Response Processing**

    - Parse streaming chunks
    - Extract text and tool calls
    - Update UI progressively
    - Track token usage

7. **Tool Execution**

    - Validate tool call
    - Execute tool handler
    - Capture results
    - Format as tool_result

8. **Loop Continue**

    - Send tool results to AI
    - AI processes and decides next action
    - Repeat until completion

9. **Task Completion**
    - AI calls `attemptCompletion` tool
    - Save final state
    - Update conversation history
    - Notify user

### State Management

**Global State** (Persisted across sessions)

```typescript
interface GlobalState {
	apiKeys: Record<string, string> // Encrypted
	userPreferences: UserPreferences
	conversationHistory: HistoryItem[]
	allowedCommands: string[]
	firstInstallCompleted: boolean
	telemetryEnabled: boolean
}
```

**Workspace State** (Per-project)

```typescript
interface WorkspaceState {
	currentTaskId?: string
	workspaceSettings: WorkspaceSettings
	fileIndexCache: IndexCache
	mcpServers: McpServerConfig[]
}
```

**Session State** (In-memory)

```typescript
interface SessionState {
	activeTask?: Task
	openTerminals: Map<string, Terminal>
	browserSessions: Map<string, BrowserSession>
	streamingBuffers: Map<string, string>
}
```

## Communication Patterns

### Extension ↔ Webview

**Message Protocol:**

```typescript
// Extension → Webview
interface ExtensionMessage {
  type: 'state' | 'action' | 'partialMessage' | 'invoke'
  // ... type-specific data
}

// Webview → Extension
interface WebviewMessage {
  type: 'newTask' | 'askResponse' | 'clearTask' | ...
  // ... type-specific data
}
```

**Implementation:**

```typescript
// Extension side
webview.postMessage({
	type: "state",
	state: extensionState,
})

webview.onDidReceiveMessage((message: WebviewMessage) => {
	handleMessage(message)
})

// Webview side
vscode.postMessage({
	type: "newTask",
	text: userInput,
})

window.addEventListener("message", (event) => {
	handleMessage(event.data)
})
```

### Service Communication

**Event-Driven:**

```typescript
class MyService extends EventEmitter {
	async performAction() {
		this.emit("action-started")

		try {
			const result = await doWork()
			this.emit("action-completed", result)
		} catch (error) {
			this.emit("action-failed", error)
		}
	}
}

myService.on("action-completed", (result) => {
	console.log("Action completed:", result)
})
```

**Direct Calls:**

```typescript
// For synchronous operations
const result = serviceA.getData()
serviceB.processData(result)

// For async operations
const data = await serviceA.fetchData()
await serviceB.saveData(data)
```

## Security Architecture

### Threat Model

**Potential Threats:**

1. Malicious code injection via AI responses
2. Unauthorized file access
3. Command execution exploits
4. API key theft
5. Data exfiltration

**Mitigations:**

#### Code Injection Prevention

- Sanitize all AI responses
- Validate tool call parameters
- Escape special characters
- No `eval()` or similar constructs

#### File Access Control

```typescript
// Respect workspace boundaries
function validatePath(path: string): boolean {
	const workspace = vscode.workspace.workspaceFolders[0]
	const resolved = path.resolve(workspace.uri.fsPath, path)

	return resolved.startsWith(workspace.uri.fsPath)
}

// Honor ignore patterns
if (rooIgnore.shouldIgnore(path)) {
	throw new Error("File is ignored")
}

// Protect critical files
if (rooProtect.isProtected(path)) {
	throw new Error("File is protected")
}
```

#### Command Execution Safety

```typescript
// Allowlist mechanism
function canExecuteCommand(command: string): boolean {
	const allowed = context.globalState.get<string[]>("allowedCommands")
	return allowed.includes(command)
}

// User confirmation for new commands
if (!canExecuteCommand(command)) {
	const approved = await vscode.window.showWarningMessage(
		`Allow command: ${command}?`,
		"Allow Once",
		"Allow Always",
		"Deny",
	)

	if (approved === "Allow Always") {
		await addToAllowlist(command)
	}

	return approved !== "Deny"
}
```

#### API Key Protection

```typescript
// Use VS Code secure storage
await context.secrets.store("anthropic-api-key", apiKey)
const apiKey = await context.secrets.get("anthropic-api-key")

// Never log sensitive data
logger.info(`Request to ${provider}`) // ✓ Good
logger.info(`API Key: ${apiKey}`) // ✗ Bad

// Clear on logout
await context.secrets.delete("anthropic-api-key")
```

### Content Security Policy

Webview CSP restricts what the UI can do:

```typescript
const csp = [
	"default-src 'none'",
	"style-src ${webview.cspSource} 'unsafe-inline'",
	"script-src ${webview.cspSource}",
	"img-src ${webview.cspSource} https: data:",
	"font-src ${webview.cspSource}",
	"connect-src https://api.anthropic.com https://api.openai.com",
].join("; ")
```

## Scalability Architecture

### Code Indexing

**Incremental Strategy:**

```
1. Initial full scan (background)
2. Watch file changes
3. Debounce updates (500ms)
4. Process in batches
5. Update index incrementally
```

**Performance Targets:**

- Index 10,000 files in <30 seconds
- Update index in <100ms per file
- Search response in <200ms
- Memory usage <500MB for large projects

### Context Window Management

**Compression Strategy:**

```
1. Calculate total tokens
2. If > 75% of limit:
   a. Summarize old messages
   b. Remove middle messages
   c. Keep system prompt + recent context
3. If still too large:
   a. Force reduction (keep 75%)
   b. Warn user
4. If error occurs:
   a. Retry with smaller context
   b. Max 3 retries
```

**Priority:**

1. System prompt (always included)
2. Last 5 messages (recent context)
3. Tool definitions (required)
4. Environment details
5. Older messages (candidates for removal)

### Memory Management

**Conversation Checkpointing:**

```typescript
// Save checkpoints every N messages
if (messageCount % CHECKPOINT_INTERVAL === 0) {
	await saveCheckpoint(task)
}

// Auto-purge old checkpoints
if (checkpointCount > MAX_CHECKPOINTS) {
	await purgeOldCheckpoints()
}
```

**Resource Limits:**

```typescript
// Limit concurrent operations
const semaphore = new Semaphore(MAX_CONCURRENT)

await semaphore.acquire()
try {
	await performOperation()
} finally {
	semaphore.release()
}

// Timeout long operations
await pTimeout(operation(), {
	milliseconds: TIMEOUT_MS,
	message: "Operation timed out",
})
```

## Error Handling Architecture

### Error Categories

```typescript
enum ErrorCategory {
	API = "api", // API-related errors
	FILESYSTEM = "filesystem", // File operations
	NETWORK = "network", // Network issues
	VALIDATION = "validation", // Input validation
	INTERNAL = "internal", // Internal errors
}
```

### Error Handling Strategy

```typescript
class ErrorHandler {
	handle(error: Error): ErrorResponse {
		// Classify error
		const category = this.classify(error)

		// Determine if retryable
		const retryable = this.isRetryable(error)

		// Format user message
		const userMessage = this.getUserMessage(error)

		// Log for debugging
		this.log(error, category)

		// Send telemetry
		this.sendTelemetry(error, category)

		return {
			category,
			retryable,
			userMessage,
			technicalDetails: error.message,
		}
	}
}
```

### Retry Logic

```typescript
async function withRetry<T>(operation: () => Promise<T>, options: RetryOptions = {}): Promise<T> {
	const { maxAttempts = 3, initialDelay = 1000, maxDelay = 60000, backoffFactor = 2 } = options

	let lastError: Error
	let delay = initialDelay

	for (let attempt = 1; attempt <= maxAttempts; attempt++) {
		try {
			return await operation()
		} catch (error) {
			lastError = error

			if (attempt < maxAttempts && isRetryable(error)) {
				await sleep(Math.min(delay, maxDelay))
				delay *= backoffFactor
			} else {
				break
			}
		}
	}

	throw lastError
}
```

## Performance Architecture

### Optimization Strategies

**1. Lazy Loading**

```typescript
// Load heavy dependencies on demand
let treeSitter: TreeSitter | undefined

function getTreeSitter(): TreeSitter {
	if (!treeSitter) {
		treeSitter = new TreeSitter()
	}
	return treeSitter
}
```

**2. Caching**

```typescript
class CacheManager {
	private cache = new Map<string, CacheEntry>()

	async get<T>(
		key: string,
		fetcher: () => Promise<T>,
		ttl: number = 300000, // 5 minutes
	): Promise<T> {
		const entry = this.cache.get(key)

		if (entry && Date.now() < entry.expiry) {
			return entry.value as T
		}

		const value = await fetcher()

		this.cache.set(key, {
			value,
			expiry: Date.now() + ttl,
		})

		return value
	}
}
```

**3. Debouncing**

```typescript
function debounce<T extends (...args: any[]) => any>(func: T, wait: number): T {
	let timeout: NodeJS.Timeout | undefined

	return ((...args: any[]) => {
		clearTimeout(timeout)
		timeout = setTimeout(() => func(...args), wait)
	}) as T
}

// Usage
const debouncedIndexFile = debounce(indexFile, 500)
watcher.onDidChange(debouncedIndexFile)
```

**4. Batching**

```typescript
class BatchProcessor<T> {
	private batch: T[] = []
	private timeout?: NodeJS.Timeout

	add(item: T) {
		this.batch.push(item)

		if (!this.timeout) {
			this.timeout = setTimeout(() => {
				this.flush()
			}, 100)
		}
	}

	private flush() {
		if (this.batch.length > 0) {
			this.processBatch(this.batch)
			this.batch = []
		}
		this.timeout = undefined
	}
}
```

### Performance Monitoring

```typescript
// Measure operation timing
async function measurePerformance<T>(operation: () => Promise<T>, label: string): Promise<T> {
	const start = Date.now()

	try {
		const result = await operation()
		const duration = Date.now() - start

		telemetry.recordTiming(label, duration)

		if (duration > SLOW_THRESHOLD) {
			logger.warn(`Slow operation: ${label} took ${duration}ms`)
		}

		return result
	} catch (error) {
		const duration = Date.now() - start
		telemetry.recordError(label, duration)
		throw error
	}
}
```

## Extensibility Architecture

### Plugin System (Future)

```typescript
interface Plugin {
	id: string
	name: string
	version: string

	activate(context: ExtensionContext): Promise<void>
	deactivate(): Promise<void>

	registerTools?(): ToolDefinition[]
	registerModes?(): ModeDefinition[]
	registerProviders?(): ProviderDefinition[]
}

class PluginManager {
	private plugins = new Map<string, Plugin>()

	async loadPlugin(path: string): Promise<void> {
		const plugin = await import(path)

		// Validate plugin
		this.validatePlugin(plugin)

		// Activate plugin
		await plugin.activate(this.context)

		// Register plugin
		this.plugins.set(plugin.id, plugin)
	}
}
```

### MCP Integration

Model Context Protocol provides extensibility:

```typescript
// Register MCP server
await mcpManager.registerServer({
	name: "my-server",
	command: "node",
	args: ["./my-mcp-server.js"],
	env: { API_KEY: "xxx" },
})

// Use MCP tool
await mcpHub.callTool("my-server", "my-tool", {
	param1: "value1",
	param2: "value2",
})

// Access MCP resource
const resource = await mcpHub.getResource("my-server", "my-resource")
```

## Future Architecture Considerations

### Distributed Processing

- Offload heavy tasks to server
- Cloud-based code indexing
- Distributed MCP servers
- Edge computing for privacy

### Multi-Agent System

- Specialized agents for different tasks
- Agent collaboration and handoff
- Parallel task execution
- Agent performance comparison

### Advanced Caching

- Semantic caching of AI responses
- Shared cache across users (privacy-preserved)
- Predictive prefetching
- Cache warming strategies

### Real-Time Collaboration

- Multi-user conversation sessions
- Shared context and state
- Conflict resolution
- Presence indicators

## Related Documentation

- [Core System](../src/core/README.md)
- [Services](../src/services/README.md)
- [API Layer](../src/api/README.md)
- [Security Best Practices](./security.md)
- [Performance Optimization](./performance.md)
