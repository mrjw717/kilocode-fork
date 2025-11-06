# Packages Documentation (`packages/`)

## Overview

The `packages/` directory contains shared, reusable packages used across the Kilo Code monorepo. These packages provide core functionality, types, configurations, and utilities that multiple applications depend on.

## Monorepo Structure

Kilo Code uses a monorepo architecture managed by **pnpm workspaces** and **Turborepo** for efficient builds and dependency management.

```
packages/
├── types/                    # Shared TypeScript types and interfaces
├── cloud/                    # Cloud service integration
├── telemetry/                # Analytics and telemetry
├── ipc/                      # Inter-process communication
├── build/                    # Build utilities and scripts
├── config-eslint/            # Shared ESLint configuration
├── config-typescript/        # Shared TypeScript configuration
└── evals/                    # Evaluation and testing framework
```

## Core Packages

### Types (`@roo-code/types`)

**Purpose:** Centralized type definitions shared across all packages and applications.

#### Key Type Categories

**API Types**

```typescript
// Provider configurations
export interface ProviderSettings {
	apiProvider: ProviderName
	apiModelId: string
	anthropicApiKey?: string
	openAiApiKey?: string
	openRouterApiKey?: string
	maxTokens?: number
	temperature?: number
}

// Model information
export interface ModelInfo {
	id: string
	name: string
	provider: string
	contextWindow: number
	maxOutputTokens: number
	supportsVision: boolean
	supportsToolUse: boolean
	pricing: ModelPricing
}
```

**Task Types**

```typescript
// Task status
export type TaskStatus = "idle" | "running" | "waiting" | "completed" | "failed" | "aborted"

// Task properties
export interface TaskProperties {
	id: string
	status: TaskStatus
	createdAt: number
	updatedAt: number
	tokenUsage?: TokenUsage
	cost?: number
}

// Task events
export interface TaskEvents {
	"status-changed": (status: TaskStatus) => void
	"message-added": (message: ClineMessage) => void
	"tool-executed": (tool: string, result: any) => void
	error: (error: Error) => void
}
```

**Message Types**

```typescript
// Chat messages
export interface ClineMessage {
	role: "user" | "assistant"
	content: MessageContent[]
	timestamp: number
	metadata?: MessageMetadata
}

export type MessageContent = TextContent | ImageContent | ToolUseContent | ToolResultContent

// Tool definitions
export interface ToolDefinition {
	name: ToolName
	description: string
	input_schema: JSONSchema
}
```

**UI Types**

```typescript
// Extension state
export interface ExtensionState {
	version: string
	apiConfiguration?: ApiConfiguration
	currentTask?: TaskProperties
	shouldShowAnnouncement: boolean
	user?: CloudUserInfo
}

// Webview messages
export type WebviewMessage = NewTaskMessage | AskResponseMessage | ClearTaskMessage | AbortTaskMessage
// ... more message types
```

#### Usage

```typescript
import { type TaskStatus, type ProviderSettings, type ClineMessage } from "@roo-code/types"
```

#### Benefits

- Single source of truth for types
- Prevents type drift across packages
- Easier refactoring
- Better IDE support
- Compile-time type checking

---

### Cloud (`@roo-code/cloud`)

**Purpose:** Cloud service integration for authentication, synchronization, and remote features.

#### Features

**Authentication**

```typescript
class CloudService {
	// Login/logout
	async login(credentials: Credentials): Promise<AuthState>
	async logout(): Promise<void>

	// Token management
	async refreshToken(): Promise<void>
	async validateToken(): Promise<boolean>

	// User info
	getUserInfo(): CloudUserInfo | undefined
	getOrganizations(): CloudOrganizationMembership[]
}
```

**Settings Synchronization**

```typescript
interface SettingsSync {
  // Push local settings to cloud
  async push(): Promise<void>

  // Pull cloud settings to local
  async pull(): Promise<void>

  // Merge strategies
  resolveConflict(local: any, remote: any): any
}
```

**Remote Task Control (Bridge)**

```typescript
class BridgeOrchestrator {
	// Enable remote control
	async enableRemoteControl(): Promise<void>

	// Receive remote commands
	onRemoteCommand(callback: (cmd: RemoteCommand) => void)

	// Send status updates
	async sendStatus(status: TaskStatus): Promise<void>
}
```

**Cloud API Client**

```typescript
class CloudAPI {
	// API endpoints
	async get<T>(path: string): Promise<T>
	async post<T>(path: string, data: any): Promise<T>
	async put<T>(path: string, data: any): Promise<T>
	async delete<T>(path: string): Promise<T>

	// WebSocket connection
	async connectWebSocket(): Promise<void>
	onMessage(callback: (msg: any) => void)
}
```

#### Configuration

```typescript
const cloudService = await CloudService.createInstance(context, logger, {
	"auth-state-changed": handleAuthChange,
	"settings-updated": handleSettingsUpdate,
	"user-info": handleUserInfo,
})
```

#### Security

- JWT-based authentication
- Secure token storage
- Encrypted data transmission
- GDPR compliance

---

### Telemetry (`@roo-code/telemetry`)

**Purpose:** Analytics, metrics, and error tracking for product insights and debugging.

#### Components

**TelemetryService**

```typescript
class TelemetryService {
	private static instance: TelemetryService
	private clients: TelemetryClient[] = []

	// Register telemetry providers
	register(client: TelemetryClient): void

	// Track events
	trackEvent(event: TelemetryEvent): void

	// Track metrics
	trackMetric(name: string, value: number): void

	// Track errors
	trackError(error: Error, context?: any): void

	// Set user properties
	identifyUser(userId: string, properties: UserProperties): void
}
```

**PostHog Integration**

```typescript
class PostHogTelemetryClient implements TelemetryClient {
	constructor(apiKey: string, options?: PostHogOptions)

	capture(event: string, properties?: any): void
	identify(userId: string, properties?: any): void
	alias(userId: string, previousId: string): void
}
```

**Event Types**

```typescript
enum TelemetryEventName {
	// Task events
	TASK_STARTED = "task_started",
	TASK_COMPLETED = "task_completed",
	TASK_FAILED = "task_failed",

	// Tool events
	TOOL_EXECUTED = "tool_executed",
	TOOL_FAILED = "tool_failed",

	// UI events
	SIDEBAR_OPENED = "sidebar_opened",
	SETTINGS_CHANGED = "settings_changed",

	// API events
	API_REQUEST = "api_request",
	API_ERROR = "api_error",
}
```

**Privacy Controls**

```typescript
interface TelemetryOptions {
	enabled: boolean // Master switch
	includeErrorData: boolean // Include error details
	includePerformanceData: boolean // Include timing data
	anonymizeData: boolean // Anonymize user data
}
```

#### Usage

```typescript
import { TelemetryService } from "@roo-code/telemetry"

TelemetryService.instance.trackEvent({
	name: "task_started",
	properties: {
		mode: "code",
		model: "claude-3-5-sonnet",
	},
})
```

#### Data Collection

- Feature usage statistics
- Performance metrics
- Error rates and types
- User journey flows
- A/B test results

---

### IPC (`@roo-code/ipc`)

**Purpose:** Inter-process communication utilities for message passing and RPC.

#### Features

**Message Channel**

```typescript
class MessageChannel<TSend, TReceive> {
	// Send messages
	send(message: TSend): void

	// Receive messages
	onMessage(handler: (message: TReceive) => void): Disposable

	// Request-response pattern
	async request<T>(message: TSend): Promise<T>
}
```

**RPC (Remote Procedure Call)**

```typescript
class RpcChannel {
	// Register methods
	registerMethod<TArgs, TResult>(name: string, handler: (args: TArgs) => Promise<TResult>): void

	// Call remote methods
	async call<TArgs, TResult>(name: string, args: TArgs): Promise<TResult>
}
```

**Event Emitter**

```typescript
class TypedEventEmitter<TEvents> {
	on<K extends keyof TEvents>(event: K, listener: TEvents[K]): Disposable

	emit<K extends keyof TEvents>(event: K, ...args: Parameters<TEvents[K]>): void
}
```

#### Use Cases

- Extension ↔ Webview communication
- Main process ↔ Worker threads
- MCP server communication
- Plugin system messaging

---

### Build (`@roo-code/build`)

**Purpose:** Build utilities, scripts, and tooling for the monorepo.

#### Scripts

**Bundle Analyzer**

```typescript
// Analyze bundle size and composition
async function analyzeBundles(entryPoints: string[]): Promise<BundleAnalysis>
```

**Version Management**

```typescript
// Update version across packages
async function updateVersion(version: string, packages: string[]): Promise<void>
```

**Asset Processing**

```typescript
// Optimize images
async function optimizeImages(inputDir: string, outputDir: string): Promise<void>

// Generate icons
async function generateIcons(source: string, sizes: number[]): Promise<void>
```

**VSIX Packaging**

```typescript
// Create VS Code extension package
async function packageExtension(manifest: PackageJson, outputPath: string): Promise<void>
```

#### Configuration

```typescript
// build.config.ts
export default {
	entryPoints: ["./src/extension.ts"],
	outdir: "./dist",
	platform: "node",
	format: "cjs",
	external: ["vscode"],
	minify: true,
	sourcemap: true,
}
```

---

### Config Packages

#### ESLint Config (`@roo-code/config-eslint`)

**Purpose:** Shared ESLint rules for consistent code quality.

```typescript
// eslint.config.mjs
export default {
	extends: ["@roo-code/config-eslint"],
	rules: {
		"@typescript-eslint/no-explicit-any": "warn",
		"@typescript-eslint/no-unused-vars": "error",
		"no-console": "warn",
	},
}
```

**Included Rules:**

- TypeScript best practices
- Import order enforcement
- Unused code detection
- Security lint rules
- Performance optimizations

#### TypeScript Config (`@roo-code/config-typescript`)

**Purpose:** Shared TypeScript compiler options.

```json
{
	"extends": "@roo-code/config-typescript/base.json",
	"compilerOptions": {
		"outDir": "./dist",
		"rootDir": "./src"
	}
}
```

**Base Configuration:**

- Strict type checking
- Modern ES features
- Source maps
- Declaration files
- Path mappings

---

### Evals (`@roo-code/evals`)

**Purpose:** Evaluation and testing framework for AI agent performance.

#### Components

**Evaluation Runner**

```typescript
class EvalRunner {
	// Run evaluation suite
	async run(suite: EvalSuite, config: EvalConfig): Promise<EvalResults>

	// Compare models
	async compareModels(models: string[], tasks: EvalTask[]): Promise<Comparison>
}
```

**Eval Metrics**

```typescript
interface EvalMetrics {
	accuracy: number // Task completion rate
	efficiency: number // Token usage efficiency
	reliability: number // Error rate
	speed: number // Average completion time
	cost: number // Average cost per task
}
```

**Test Cases**

```typescript
interface EvalTask {
	id: string
	description: string
	initialState: WorkspaceState
	expectedOutcome: Outcome
	timeout: number
	maxTokens: number
}
```

**Result Analysis**

```typescript
interface EvalResults {
	passed: number
	failed: number
	metrics: EvalMetrics
	details: TaskResult[]
	comparison?: ModelComparison
}
```

#### Use Cases

- Model performance comparison
- Regression testing
- Feature validation
- Prompt engineering optimization
- Cost analysis

---

## Package Dependencies

### Dependency Graph

```
@roo-code/types (base)
    ↑
    ├── @roo-code/cloud
    ├── @roo-code/telemetry
    ├── @roo-code/ipc
    └── @roo-code/evals

@roo-code/config-eslint (dev)
@roo-code/config-typescript (dev)
@roo-code/build (dev)
```

### Workspace Configuration

**pnpm-workspace.yaml:**

```yaml
packages:
    - "packages/*"
    - "apps/*"
    - "src"
```

**turbo.json:**

```json
{
	"pipeline": {
		"build": {
			"dependsOn": ["^build"],
			"outputs": ["dist/**"]
		},
		"test": {
			"dependsOn": ["build"]
		},
		"lint": {},
		"check-types": {}
	}
}
```

## Development Workflow

### Building Packages

```bash
# Build all packages
pnpm build

# Build specific package
pnpm --filter @roo-code/types build

# Build with dependencies
pnpm --filter @roo-code/cloud... build
```

### Testing Packages

```bash
# Test all packages
pnpm test

# Test specific package
pnpm --filter @roo-code/telemetry test

# Watch mode
pnpm --filter @roo-code/cloud test -- --watch
```

### Publishing Packages

```bash
# Create changeset
pnpm changeset

# Version packages
pnpm changeset version

# Publish to npm
pnpm changeset publish
```

## Best Practices

### Package Design

- **Single Responsibility**: Each package has one clear purpose
- **Minimal Dependencies**: Reduce coupling between packages
- **Well-Defined Exports**: Clear public API surface
- **Versioning**: Semantic versioning for all packages

### Code Organization

- **Clear Structure**: Consistent folder layout
- **TypeScript First**: All code in TypeScript
- **Comprehensive Tests**: High test coverage
- **Documentation**: README in each package

### Performance

- **Tree Shaking**: Mark side effects for bundlers
- **Code Splitting**: Separate large features
- **Lazy Loading**: Load only when needed
- **Caching**: Cache build outputs

### Maintenance

- **Keep Updated**: Regular dependency updates
- **Breaking Changes**: Document in CHANGELOG
- **Deprecation**: Clear migration paths
- **Support**: Help downstream consumers

## Future Enhancements

### Planned Packages

- **@roo-code/database**: Local database utilities
- **@roo-code/crypto**: Encryption and security
- **@roo-code/testing**: Enhanced testing utilities
- **@roo-code/logging**: Structured logging
- **@roo-code/metrics**: Performance monitoring

### Improvements

- More granular package splitting
- Better tree shaking support
- Enhanced type exports
- Performance optimizations
- Additional utility functions

## Related Documentation

- [Monorepo Guide](../guides/monorepo.md)
- [Publishing Guide](../guides/publishing.md)
- [Package Development](../guides/package-development.md)
- [Types Reference](./types/README.md)
- [Cloud Integration](./cloud/README.md)

## Contributing to Packages

When contributing to packages:

1. Understand package boundaries
2. Maintain backward compatibility
3. Update CHANGELOG.md
4. Add tests for new features
5. Update documentation
6. Create changeset for version management
7. Consider downstream consumers
