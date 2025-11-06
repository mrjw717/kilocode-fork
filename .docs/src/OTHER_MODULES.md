# Additional Modules Documentation

## Internationalization (`i18n/`)

### Overview

Provides multi-language support for the extension UI and messages.

### Supported Languages

- **English (en)**: Default language
- **Spanish (es)**: español
- **French (fr)**: français
- **German (de)**: Deutsch
- **Japanese (ja)**: 日本語
- **Chinese Simplified (zh-CN)**: 简体中文
- **Chinese Traditional (zh-TW)**: 繁體中文
- **Korean (ko)**: 한국어
- **Portuguese (pt-BR)**: português brasileiro
- **Russian (ru)**: русский
- **Arabic (ar)**: العربية
- **Hindi (hi)**: हिन्दी
- **Italian (it)**: italiano
- **Dutch (nl)**: Nederlands
- **Polish (pl)**: polski
- **Czech (cs)**: čeština
- **Turkish (tr)**: Türkçe
- **Ukrainian (uk)**: українська
- **Vietnamese (vi)**: Tiếng Việt
- **Thai (th)**: ไทย
- **Indonesian (id)**: Bahasa Indonesia
- **Catalan (ca)**: català

### Structure

```
i18n/
├── index.ts                # i18n initialization
├── locales/               # Translation files
│   ├── en/               # English (base)
│   │   ├── common.json
│   │   ├── kilocode.json
│   │   └── ...
│   ├── es/               # Spanish
│   ├── fr/               # French
│   └── ...
└── types.ts              # Translation types
```

### Usage

```typescript
import { t } from "@/i18n"

// Simple translation
const message = t("common.welcome")

// With interpolation
const greeting = t("common.hello", { name: "Alice" })

// Pluralization
const count = t("common.itemCount", { count: 5 })
```

### Translation Keys

```json
{
	"common": {
		"welcome": "Welcome to Kilo Code",
		"error": "An error occurred",
		"success": "Operation completed successfully"
	},
	"task": {
		"creating": "Creating task...",
		"running": "Task is running",
		"completed": "Task completed"
	}
}
```

### Adding New Translations

1. Add keys to `en/` folder (base language)
2. Translate to other languages
3. Use translation function `t(key)`
4. Test in different locales

### Language Detection

```typescript
// Detect from VS Code settings
const userLang = vscode.env.language // e.g., 'en', 'es-ES'
const baseLang = formatLanguage(userLang) // 'en', 'es'

// Initialize i18n
initializeI18n(baseLang)
```

### Fallback Behavior

- Missing translations fall back to English
- Graceful degradation for unsupported languages
- Logs warning for missing keys (development)

---

## Shared Utilities (`shared/`)

### Overview

Common utilities, types, and constants used across the extension.

### Key Modules

#### Types (`shared/types.ts`)

```typescript
export interface RooCodeSettings {
	apiProvider: ProviderName
	apiModelId: string
	customInstructions?: string
	maxTokens?: number
	// ... more settings
}

export type TaskStatus = "idle" | "running" | "waiting" | "completed" | "failed"

export interface ClineMessage {
	role: "user" | "assistant"
	content: MessageContent[]
	timestamp: number
}
```

#### Constants (`shared/constants.ts`)

```typescript
export const DEFAULT_MAX_TOKENS = 8192
export const DEFAULT_TEMPERATURE = 0.7
export const MAX_FILE_SIZE_MB = 10
export const SUPPORTED_IMAGE_FORMATS = ["png", "jpg", "jpeg", "gif", "webp"]
```

#### Package Info (`shared/package.ts`)

```typescript
export const Package = {
	name: "kilo-code",
	version: "4.115.0",
	displayName: "Kilo Code",
	publisher: "kilocode",
}
```

#### Array Utilities (`shared/array.ts`)

```typescript
// Find last matching element
export function findLast<T>(array: T[], predicate: (item: T) => boolean): T | undefined

// Find last index
export function findLastIndex<T>(array: T[], predicate: (item: T) => boolean): number

// Chunk array
export function chunk<T>(array: T[], size: number): T[][]
```

#### Modes (`shared/modes.ts`)

```typescript
export interface Mode {
	slug: string
	name: string
	roleDefinition: string
	groups: string[]
}

export const DEFAULT_MODES: Mode[] = [
	{
		slug: "code",
		name: "Coder",
		roleDefinition: "Expert programmer focused on implementation",
		groups: ["file", "command", "browser"],
	},
	{
		slug: "architect",
		name: "Architect",
		roleDefinition: "System designer focused on architecture",
		groups: ["planning", "design"],
	},
	{
		slug: "debug",
		name: "Debugger",
		roleDefinition: "Problem solver focused on fixing issues",
		groups: ["diagnostic", "fix"],
	},
]
```

#### Extension Messages (`shared/ExtensionMessage.ts`)

```typescript
export interface ExtensionMessage {
	type: "state" | "action" | "partialMessage" | "invoke" | "selectedImages"
	// ... message-specific data
}

export interface ExtensionState {
	version: string
	apiConfiguration?: ApiConfiguration
	currentTask?: TaskProperties
	shouldShowAnnouncement: boolean
	// ... more state
}
```

#### Webview Messages (`shared/WebviewMessage.ts`)

```typescript
export type WebviewMessage =
	| { type: "newTask"; text: string; images?: string[] }
	| { type: "askResponse"; response: ClineAskResponse }
	| { type: "clearTask" }
	| { type: "abortTask" }
	| { type: "exportTask" }
// ... more message types
```

---

## Utilities (`utils/`)

### Overview

Helper functions for common operations.

### Key Utilities

#### File System (`utils/fs.ts`)

```typescript
// Check if file exists
export async function fileExistsAtPath(path: string): Promise<boolean>

// Read file safely
export async function readFileContent(path: string, encoding?: string): Promise<string>

// Write file with directory creation
export async function writeFileContent(path: string, content: string): Promise<void>

// Get file size
export async function getFileSize(path: string): Promise<number>
```

#### Path Utilities (`utils/path.ts`)

```typescript
// Cross-platform path normalization
String.prototype.toPosix = function () {
	return this.split(path.sep).join("/")
}

// Get workspace path
export function getWorkspacePath(): string | undefined

// Resolve relative path
export function resolveWorkspacePath(relativePath: string): string
```

#### Git Operations (`utils/git.ts`)

```typescript
// Get git info for workspace
export async function getWorkspaceGitInfo(workspacePath: string): Promise<GitProperties | undefined>

// Check if path is git repo
export function isGitRepository(path: string): Promise<boolean>

// Get current branch
export async function getCurrentBranch(repoPath: string): Promise<string>
```

#### Logging (`utils/outputChannelLogger.ts`)

```typescript
// Create logger
export function createOutputChannelLogger(channel: vscode.OutputChannel): Logger

// Log with timestamp
export function log(level: "info" | "warn" | "error", message: string): void

// Dual logger (console + output channel)
export function createDualLogger(logger: Logger): DualLogger
```

#### Settings Migration (`utils/migrateSettings.ts`)

```typescript
// Migrate old settings to new format
export async function migrateSettings(
	context: vscode.ExtensionContext,
	outputChannel: vscode.OutputChannel,
): Promise<void>

// Migration steps
const migrations = [migrateApiKeyStorage, migrateModeSettings, migrateCustomInstructions]
```

#### Auto-import (`utils/autoImportSettings.ts`)

```typescript
// Auto-import settings from URL
export async function autoImportSettings(
	outputChannel: vscode.OutputChannel,
	managers: {
		providerSettingsManager: ProviderSettingsManager
		contextProxy: ContextProxy
		customModesManager: CustomModesManager
	},
): Promise<void>
```

#### Text-to-Speech (`utils/tts.ts`)

```typescript
// Enable/disable TTS
export function setTtsEnabled(enabled: boolean): void

// Set TTS speed
export function setTtsSpeed(speed: number): void

// Speak text
export async function speak(text: string): Promise<void>

// Stop speaking
export function stopSpeaking(): void
```

---

## Workers (`workers/`)

### Overview

Background worker processes for resource-intensive operations.

### Structure

```
workers/
├── tts-worker/           # Text-to-speech worker
│   ├── index.ts
│   └── engines/
└── index-worker/         # Code indexing worker (future)
```

### TTS Worker

Handles text-to-speech synthesis in background thread.

```typescript
interface TtsWorkerMessage {
	type: "speak" | "stop" | "setSpeed"
	text?: string
	speed?: number
}

// In main thread
const worker = new Worker("./tts-worker/index.js")

worker.postMessage({
	type: "speak",
	text: "Hello, world!",
	speed: 1.0,
})

worker.on("message", (msg) => {
	if (msg.type === "complete") {
		console.log("TTS finished")
	}
})
```

### Benefits of Workers

- Non-blocking main thread
- Parallel processing
- Better performance
- Isolated crashes
- Memory isolation

### Future Workers

- **Index Worker**: Code indexing in background
- **Parse Worker**: Tree-sitter parsing
- **Compression Worker**: File compression
- **Diff Worker**: Large diff calculations

---

## Walkthrough (`walkthrough/`)

### Overview

First-run tutorial for new users.

### Structure

```
walkthrough/
├── step1.md              # Introduction
├── step2.md              # Basic usage
├── step3.md              # Advanced features
├── step4.md              # MCP servers
└── step5.md              # Tips & tricks
```

### Walkthrough Configuration

Defined in `package.json`:

```json
{
	"contributes": {
		"walkthroughs": [
			{
				"id": "kiloCodeWalkthrough",
				"title": "5 ways Kilo Code helps you code",
				"description": "Learn how to use Kilo Code effectively",
				"steps": [
					{
						"id": "welcome",
						"title": "Write code for you",
						"description": "Generate code from descriptions",
						"media": {
							"markdown": "./dist/walkthrough/step1.md"
						}
					}
				]
			}
		]
	}
}
```

### Step Content Example

**step1.md:**

```markdown
# Welcome to Kilo Code!

Kilo Code is your AI coding assistant that can:

- 🚀 Write code from natural language
- 🔍 Understand your entire codebase
- 🐛 Debug and fix errors
- ✨ Generate tests
- 📝 Add documentation

## Get Started

1. Open the Kilo Code sidebar
2. Describe what you want to build
3. Review and approve the AI's suggestions
4. Your code is ready!

[Open Kilo Code](#command:kilo-code.plusButtonClicked)
```

### Triggering Walkthrough

```typescript
// Show on first install
if (!context.globalState.get("firstInstallCompleted")) {
	await vscode.commands.executeCommand(
		"workbench.action.openWalkthrough",
		"kilocode.kilo-code#kiloCodeWalkthrough",
		false,
	)

	await context.globalState.update("firstInstallCompleted", true)
}
```

### Interactive Elements

- **Command Links**: Click to execute commands
- **Settings Links**: Jump to settings
- **External Links**: Documentation, videos
- **Code Snippets**: Copy-paste examples

### Customization

Organizations can customize walkthrough:

- Custom steps for team workflows
- Internal documentation links
- Company-specific examples
- Branding and messaging

---

## Testing Infrastructure (`__tests__/`, `__mocks__/`)

### Test Organization

```
src/
├── __tests__/            # Top-level tests
│   ├── integration/      # Integration tests
│   └── e2e/             # End-to-end tests
├── core/__tests__/       # Core module tests
├── services/__tests__/   # Service tests
└── __mocks__/           # Mock implementations
    ├── vscode.ts        # VS Code API mock
    ├── fs.ts            # File system mock
    └── api.ts           # API mock
```

### Test Framework

**Vitest Configuration:**

```typescript
export default defineConfig({
	test: {
		globals: true,
		environment: "node",
		setupFiles: "./vitest.setup.ts",
		coverage: {
			provider: "v8",
			reporter: ["text", "json", "html"],
			exclude: ["**/__tests__/**", "**/__mocks__/**"],
		},
	},
})
```

### Mock VS Code API

```typescript
// __mocks__/vscode.ts
export const window = {
	showInformationMessage: jest.fn(),
	showErrorMessage: jest.fn(),
	createOutputChannel: jest.fn(() => ({
		appendLine: jest.fn(),
		show: jest.fn(),
	})),
	activeTextEditor: undefined,
}

export const workspace = {
	getConfiguration: jest.fn(() => ({
		get: jest.fn(),
		update: jest.fn(),
	})),
	workspaceFolders: [],
}

export const commands = {
	registerCommand: jest.fn(),
	executeCommand: jest.fn(),
}
```

### Example Test

```typescript
import { describe, it, expect, beforeEach } from "vitest"
import { Task } from "../task/Task"

describe("Task", () => {
	let task: Task

	beforeEach(() => {
		task = new Task({
			provider: mockProvider,
			apiConfiguration: mockConfig,
		})
	})

	it("should initialize correctly", () => {
		expect(task.status).toBe("idle")
		expect(task.messages).toHaveLength(0)
	})

	it("should execute task", async () => {
		await task.execute("Create a test file")

		expect(task.status).toBe("completed")
		expect(task.messages.length).toBeGreaterThan(0)
	})
})
```

### Coverage Goals

- **Core Logic**: >80% coverage
- **Services**: >70% coverage
- **Integration**: Critical paths covered
- **E2E**: Major user flows tested

---

## Assets (`assets/`)

### Overview

Static assets used by the extension.

### Structure

```
assets/
├── icons/               # Extension icons
│   ├── logo.png        # Main logo
│   ├── logo-outline.png
│   └── ...
├── images/             # UI images
│   ├── walkthrough/    # Walkthrough images
│   └── examples/       # Example screenshots
└── sounds/             # Audio files (future)
```

### Icon Requirements

- **Extension Icon**: 128x128 PNG
- **Marketplace Icon**: 256x256 PNG
- **Status Bar Icons**: SVG or PNG
- **Codicons**: VS Code icon font

### Image Optimization

- Compress PNG files
- Use WebP when possible
- Lazy load large images
- Provide multiple resolutions

### Best Practices

- Use relative paths
- Version control binary assets
- Optimize file sizes
- Provide alt text
- Support light/dark themes

---

## Configuration Files

### TypeScript Configuration (`tsconfig.json`)

```json
{
	"compilerOptions": {
		"module": "commonjs",
		"target": "ES2020",
		"outDir": "./dist",
		"lib": ["ES2020"],
		"sourceMap": true,
		"rootDir": ".",
		"strict": true,
		"esModuleInterop": true,
		"skipLibCheck": true,
		"forceConsistentCasingInFileNames": true
	},
	"exclude": ["node_modules", ".vscode-test"]
}
```

### ESLint Configuration (`eslint.config.mjs`)

```javascript
export default [
	{
		files: ["**/*.ts"],
		languageOptions: {
			parser: "@typescript-eslint/parser",
			parserOptions: {
				project: "./tsconfig.json",
			},
		},
		plugins: {
			"@typescript-eslint": typescriptEslint,
		},
		rules: {
			"@typescript-eslint/no-explicit-any": "warn",
			"@typescript-eslint/no-unused-vars": "error",
			"no-console": "warn",
		},
	},
]
```

### Build Configuration (`esbuild.mjs`)

```javascript
await esbuild.build({
	entryPoints: ["./extension.ts"],
	bundle: true,
	outfile: "./dist/extension.js",
	external: ["vscode"],
	format: "cjs",
	platform: "node",
	target: "node20",
	sourcemap: true,
	minify: process.env.NODE_ENV === "production",
})
```

---

## Best Practices Summary

### Code Organization

- Logical folder structure
- Clear module boundaries
- Minimal dependencies
- Single responsibility

### Error Handling

- Try-catch all async operations
- User-friendly error messages
- Proper logging
- Graceful degradation

### Performance

- Lazy loading
- Caching
- Debouncing
- Resource cleanup

### Testing

- Unit tests for logic
- Integration tests for APIs
- E2E tests for workflows
- Mock external dependencies

### Documentation

- README in each folder
- TSDoc comments
- Usage examples
- Architecture decisions

---

## Related Documentation

- [Main README](../README.md)
- [Core Documentation](./core/README.md)
- [Services Documentation](./services/README.md)
- [API Documentation](./api/README.md)
- [Architecture Overview](../architecture/README.md)
