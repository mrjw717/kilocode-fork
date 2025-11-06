# Integrations Documentation (`src/integrations/`)

## Overview

The `integrations/` directory contains modules that bridge Kilo Code with VS Code's native features, external tools, and system capabilities. These integrations enable the AI agent to interact with the IDE, file system, terminal, version control, and more.

## Directory Structure

```
integrations/
├── editor/                  # VS Code editor integration
│   ├── DiffViewProvider.ts # Side-by-side diff viewer
│   ├── EditorUtils.ts      # Editor manipulation utilities
│   └── DecorationManager.ts # Text decorations and highlights
├── terminal/                # Terminal integration
│   ├── Terminal.ts         # Terminal instance management
│   ├── TerminalRegistry.ts # Global terminal registry
│   └── types.ts            # Terminal-related types
├── workspace/               # Workspace and file system
│   ├── WorkspaceTracker.ts # Track workspace changes
│   └── FileWatcher.ts      # File system event monitoring
├── misc/                    # Miscellaneous utilities
│   ├── export-markdown.ts  # Export conversations
│   └── diagnostics.ts      # VS Code diagnostics integration
└── theme/                   # Theme detection
    └── getTheme.ts         # Detect light/dark theme
```

## Editor Integration (`editor/`)

### DiffViewProvider

Provides side-by-side diff view for reviewing AI-proposed changes.

#### Purpose

- **Visual Comparison**: See original vs. proposed changes
- **Review Changes**: Accept or reject modifications
- **Syntax Highlighting**: Proper language highlighting
- **Read-Only Original**: Prevent accidental edits to left side

#### Implementation

```typescript
class DiffViewProvider implements vscode.TextDocumentContentProvider {
	provideTextDocumentContent(uri: vscode.Uri): string {
		// Return original content for diff view
	}
}

// Usage
await vscode.commands.executeCommand(
	"vscode.diff",
	originalUri, // Left side (read-only)
	modifiedUri, // Right side (editable)
	"Original ↔ Modified",
)
```

#### URI Scheme

Uses custom URI scheme for virtual documents:

```
kilo-code-diff://<file-path>?original=<content-hash>
```

#### Features

- Inline diff annotations
- Navigate between changes
- Accept/reject individual hunks
- Preserve file formatting
- Undo/redo support

### EditorUtils

Helper functions for editor manipulation.

#### Common Operations

```typescript
// Get active editor
const editor = vscode.window.activeTextEditor

// Get selection
const selection = editor.selection
const selectedText = editor.document.getText(selection)

// Replace text
await editor.edit((editBuilder) => {
	editBuilder.replace(range, newText)
})

// Insert text
await editor.edit((editBuilder) => {
	editBuilder.insert(position, text)
})

// Delete text
await editor.edit((editBuilder) => {
	editBuilder.delete(range)
})

// Move cursor
editor.selection = new vscode.Selection(position, position)

// Reveal position
editor.revealRange(range, vscode.TextEditorRevealType.InCenter)
```

#### Batch Edits

```typescript
const workspaceEdit = new vscode.WorkspaceEdit()
workspaceEdit.replace(uri, range, newText)
workspaceEdit.insert(uri, position, text)
await vscode.workspace.applyEdit(workspaceEdit)
```

### DecorationManager

Manages text decorations for highlighting and annotations.

#### Decoration Types

```typescript
// Highlight changed lines
const changedDecoration = vscode.window.createTextEditorDecorationType({
	backgroundColor: "rgba(255, 200, 0, 0.2)",
	isWholeLine: true,
})

// Inline annotations
const annotationDecoration = vscode.window.createTextEditorDecorationType({
	after: {
		contentText: " ← AI suggested",
		color: "gray",
		fontStyle: "italic",
	},
})

// Error highlighting
const errorDecoration = vscode.window.createTextEditorDecorationType({
	borderColor: "red",
	borderWidth: "1px",
	borderStyle: "solid",
})
```

#### Application

```typescript
editor.setDecorations(decorationType, ranges)
```

## Terminal Integration (`terminal/`)

### Terminal

Manages individual terminal instances for command execution.

#### Features

- **Command Execution**: Run shell commands
- **Output Capture**: Capture stdout/stderr
- **Process Management**: Track PIDs and status
- **Input Streaming**: Send input to running processes
- **Exit Code Handling**: Detect success/failure

#### Usage

```typescript
const terminal = new Terminal(name, workingDir)

// Execute command
const result = await terminal.run("npm install", {
	timeout: 60000,
	captureOutput: true,
})

// Check result
if (result.exitCode === 0) {
	console.log("Success:", result.output)
} else {
	console.error("Failed:", result.error)
}

// Send input to running process
await terminal.sendText("y\n")

// Kill process
await terminal.kill()
```

#### Process Tracking

```typescript
interface RooTerminalProcess {
	id: string
	command: string
	pid: number
	status: "running" | "completed" | "failed"
	output: string
	exitCode?: number
	startTime: number
	endTime?: number
}
```

### TerminalRegistry

Global registry managing all terminal instances.

#### Purpose

- **Centralized Management**: Track all terminals
- **Reuse Terminals**: Avoid creating duplicates
- **Cleanup**: Dispose unused terminals
- **State Persistence**: Remember terminal state

#### API

```typescript
class TerminalRegistry {
	static initialize(): void
	static getTerminal(name: string): Terminal | undefined
	static createTerminal(name: string): Terminal
	static disposeTerminal(name: string): void
	static getAllTerminals(): Terminal[]
	static clear(): void
}
```

#### Integration with VS Code

```typescript
// Get VS Code terminal
const vscodeTerminal = vscode.window.terminals.find((t) => t.name === name)

// Show terminal
vscodeTerminal?.show()

// Send command to terminal
vscodeTerminal?.sendText(command)
```

### Terminal Actions

Quick actions for terminal-related tasks.

#### Available Actions

- **Run in Terminal**: Execute selected code
- **Explain Command**: AI explains terminal output
- **Fix Error**: AI suggests fixes for errors
- **Repeat Last Command**: Rerun previous command
- **Clear Terminal**: Clear terminal output

#### Trigger Methods

- Context menu in terminal
- Command palette
- Keyboard shortcuts
- Code lens above errors

## Workspace Integration (`workspace/`)

### WorkspaceTracker

Monitors workspace state and file changes.

#### Tracked Information

```typescript
interface WorkspaceState {
	folders: WorkspaceFolder[]
	openFiles: string[]
	changedFiles: Set<string>
	deletedFiles: Set<string>
	createdFiles: Set<string>
	gitStatus: GitStatus
}
```

#### Events

```typescript
workspaceTracker.on("fileChanged", (file: string) => {
	// Handle file change
})

workspaceTracker.on("fileCreated", (file: string) => {
	// Handle new file
})

workspaceTracker.on("fileDeleted", (file: string) => {
	// Handle deletion
})
```

#### Use Cases

- Track files modified by AI
- Detect external file changes
- Monitor git status changes
- Refresh file index
- Update context awareness

### FileWatcher

Low-level file system event monitoring.

#### Watched Events

- File created
- File changed
- File deleted
- File renamed
- Directory created
- Directory deleted

#### Pattern Matching

```typescript
const watcher = vscode.workspace.createFileSystemWatcher(
	"**/*.{ts,tsx,js,jsx}",
	false, // ignoreCreateEvents
	false, // ignoreChangeEvents
	false, // ignoreDeleteEvents
)

watcher.onDidCreate((uri) => {
	console.log("Created:", uri.fsPath)
})

watcher.onDidChange((uri) => {
	console.log("Changed:", uri.fsPath)
})

watcher.onDidDelete((uri) => {
	console.log("Deleted:", uri.fsPath)
})
```

#### Performance Optimization

- Debounce rapid changes
- Filter by patterns
- Batch processing
- Respect .gitignore

## Theme Integration (`theme/`)

### Theme Detection

Detect current VS Code theme for UI adaptation.

```typescript
function getTheme(): "light" | "dark" | "high-contrast" {
	const theme = vscode.window.activeColorTheme

	if (theme.kind === vscode.ColorThemeKind.Light) {
		return "light"
	} else if (theme.kind === vscode.ColorThemeKind.Dark) {
		return "dark"
	} else {
		return "high-contrast"
	}
}

// Listen for theme changes
vscode.window.onDidChangeActiveColorTheme((theme) => {
	updateUIColors(getTheme())
})
```

#### Use Cases

- Adapt webview colors
- Choose appropriate icons
- Adjust syntax highlighting
- Match VS Code aesthetics

## Miscellaneous Integrations (`misc/`)

### Export to Markdown

Export conversations to markdown files.

```typescript
async function exportToMarkdown(taskId: string, outputPath: string): Promise<void> {
	const messages = await loadTaskMessages(taskId)
	const markdown = formatAsMarkdown(messages)
	await fs.writeFile(outputPath, markdown)
}
```

#### Format

````markdown
# Conversation: [Task Name]

**Date**: 2024-01-15
**Model**: claude-3-5-sonnet
**Total Tokens**: 15,234

## User

Create a login form with email and password

## Assistant

I'll create a login form component...

### Tool Use: write_to_file

```json
{
	"path": "components/LoginForm.tsx",
	"content": "..."
}
```
````

### Tool Result

File created successfully

````

### Diagnostics Integration

Access VS Code diagnostics (errors, warnings).

```typescript
// Get diagnostics for file
const diagnostics = vscode.languages.getDiagnostics(uri)

// Format for AI
const problemsText = diagnostics.map(d =>
  `${d.severity}: ${d.message} at line ${d.range.start.line}`
).join('\n')

// Watch for changes
vscode.languages.onDidChangeDiagnostics(event => {
  event.uris.forEach(uri => {
    const problems = vscode.languages.getDiagnostics(uri)
    updateProblemsContext(uri, problems)
  })
})
````

#### Use Cases

- Include errors in context
- Auto-fix on save
- Explain error messages
- Suggest fixes
- Navigate to problems

## Git Integration

### Git Status

Get repository information:

```typescript
interface GitInfo {
	branch: string
	commit: string
	hasChanges: boolean
	unstagedFiles: string[]
	stagedFiles: string[]
	remote?: string
}

async function getGitInfo(workspaceRoot: string): Promise<GitInfo | undefined>
```

### Git Operations

Common git operations:

```typescript
// Stage files
await git.add(files)

// Commit
await git.commit(message)

// Push
await git.push()

// Pull
await git.pull()

// Create branch
await git.createBranch(name)

// Checkout
await git.checkout(branch)

// Get diff
const diff = await git.diff()
```

### Commit Message Generation

Integrate with commit message service:

```typescript
vscode.commands.registerCommand("kilo-code.generateCommitMessage", async () => {
	const diff = await git.diff()
	const message = await generateCommitMessage(diff)

	// Set in source control input box
	scm.inputBox.value = message
})
```

## Language Server Protocol (LSP)

### Language Features

Access language intelligence:

```typescript
// Hover information
const hover = await vscode.commands.executeCommand<vscode.Hover[]>("vscode.executeHoverProvider", uri, position)

// Definitions
const definitions = await vscode.commands.executeCommand<vscode.Location[]>(
	"vscode.executeDefinitionProvider",
	uri,
	position,
)

// References
const references = await vscode.commands.executeCommand<vscode.Location[]>(
	"vscode.executeReferenceProvider",
	uri,
	position,
)

// Rename
const workspaceEdit = await vscode.commands.executeCommand<vscode.WorkspaceEdit>(
	"vscode.executeDocumentRenameProvider",
	uri,
	position,
	newName,
)

// Code actions
const codeActions = await vscode.commands.executeCommand<vscode.CodeAction[]>(
	"vscode.executeCodeActionProvider",
	uri,
	range,
)
```

## Extension API Integration

### Commands

Register VS Code commands:

```typescript
context.subscriptions.push(
	vscode.commands.registerCommand("kilo-code.myCommand", async (...args) => {
		// Command implementation
	}),
)
```

### Menus

Add menu items:

```json
{
	"contributes": {
		"menus": {
			"editor/context": [
				{
					"command": "kilo-code.explain",
					"when": "editorHasSelection",
					"group": "kilo-code"
				}
			],
			"explorer/context": [
				{
					"command": "kilo-code.generateTests",
					"when": "resourceLangId == typescript"
				}
			]
		}
	}
}
```

### Status Bar

Add status bar items:

```typescript
const statusBarItem = vscode.window.createStatusBarItem(vscode.StatusBarAlignment.Right, 100)

statusBarItem.text = "$(sync~spin) Kilo Code"
statusBarItem.command = "kilo-code.openPanel"
statusBarItem.tooltip = "Click to open Kilo Code"
statusBarItem.show()
```

### Notifications

Show user notifications:

```typescript
// Information
vscode.window.showInformationMessage("Task completed!")

// Warning
vscode.window.showWarningMessage("High token usage")

// Error
vscode.window.showErrorMessage("API request failed")

// With actions
const action = await vscode.window.showInformationMessage("File modified", "View", "Dismiss")

if (action === "View") {
	vscode.window.showTextDocument(uri)
}
```

### Input Collection

Get user input:

```typescript
// Simple text input
const name = await vscode.window.showInputBox({
	prompt: "Enter project name",
	placeHolder: "my-project",
	validateInput: (value) => {
		return value.length > 0 ? null : "Name required"
	},
})

// Quick pick
const choice = await vscode.window.showQuickPick(["Option 1", "Option 2", "Option 3"], {
	placeHolder: "Select an option",
	canPickMany: false,
})

// Multi-step input
const result = await vscode.window.createInputBox()
result.step = 1
result.totalSteps = 3
result.show()
```

## Performance Considerations

### Event Throttling

Throttle high-frequency events:

```typescript
let timeout: NodeJS.Timeout | undefined

watcher.onDidChange((uri) => {
	if (timeout) clearTimeout(timeout)

	timeout = setTimeout(() => {
		handleFileChange(uri)
	}, 500)
})
```

### Lazy Activation

Activate integrations only when needed:

```typescript
let terminalRegistry: TerminalRegistry | undefined

function getTerminalRegistry(): TerminalRegistry {
	if (!terminalRegistry) {
		terminalRegistry = new TerminalRegistry()
	}
	return terminalRegistry
}
```

### Resource Cleanup

Proper disposal of resources:

```typescript
class MyIntegration {
  private disposables: vscode.Disposable[] = []

  activate() {
    this.disposables.push(
      vscode.workspace.onDidChangeTextDocument(...),
      vscode.window.onDidChangeActiveTextEditor(...),
      this.watcher
    )
  }

  dispose() {
    this.disposables.forEach(d => d.dispose())
  }
}
```

## Security

### Sandbox Boundaries

Respect VS Code security boundaries:

- No arbitrary code execution
- File access within workspace
- Network requests validated
- Command execution restricted

### Permission Model

Follow least privilege principle:

- Request only needed permissions
- Explain permission usage
- Allow user control
- Audit sensitive operations

## Testing

### Integration Tests

Test VS Code integration:

```typescript
suite("Editor Integration", () => {
	test("applies text edits correctly", async () => {
		const doc = await vscode.workspace.openTextDocument({
			content: "original text",
		})

		const editor = await vscode.window.showTextDocument(doc)

		await editor.edit((builder) => {
			builder.replace(new vscode.Range(0, 0, 0, 13), "modified text")
		})

		assert.strictEqual(doc.getText(), "modified text")
	})
})
```

### Mock VS Code API

For unit tests:

```typescript
const mockVscode = {
	window: {
		showInformationMessage: jest.fn(),
		activeTextEditor: mockEditor,
	},
	workspace: {
		openTextDocument: jest.fn(),
	},
}
```

## Related Documentation

- [Terminal Usage Guide](../../guides/terminal.md)
- [Editor Commands](../../commands/editor.md)
- [Git Integration](../../guides/git.md)
- [VS Code API Reference](https://code.visualstudio.com/api)

## Contributing

When adding new integrations:

1. Follow VS Code API best practices
2. Handle errors gracefully
3. Dispose resources properly
4. Add comprehensive tests
5. Document public APIs
6. Consider performance impact
