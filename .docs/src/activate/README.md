# Activation Documentation (`src/activate/`)

## Overview

The `activate/` directory contains modules responsible for extension activation, command registration, and IDE integration setup. This is where VS Code commands, code actions, and terminal actions are registered and handled.

## Directory Structure

```
activate/
├── index.ts                    # Main exports
├── registerCommands.ts         # Command registration
├── registerCodeActions.ts      # Code action provider setup
├── registerTerminalActions.ts  # Terminal action setup
├── CodeActionProvider.ts       # Code action implementation
├── handleUri.ts                # Deep link handling
├── handleTask.ts               # Task creation from external sources
└── humanRelay.ts               # Human-in-the-loop relay
```

## Command Registration (`registerCommands.ts`)

Registers all VS Code commands for the extension.

### Registered Commands

#### Primary Commands

- **`kilo-code.plusButtonClicked`**: Open main chat panel
- **`kilo-code.openInNewTab`**: Open chat in editor tab
- **`kilo-code.settingsButtonClicked`**: Open settings
- **`kilo-code.historyButtonClicked`**: View conversation history
- **`kilo-code.mcpButtonClicked`**: Manage MCP servers

#### Task Management

- **`kilo-code.newTask`**: Create new task
- **`kilo-code.clearTask`**: Clear current conversation
- **`kilo-code.abortTask`**: Cancel running task
- **`kilo-code.exportTask`**: Export conversation to markdown

#### Context Actions

- **`kilo-code.explainCode`**: Explain selected code
- **`kilo-code.generateTests`**: Generate unit tests
- **`kilo-code.fixErrors`**: Fix diagnostics in file
- **`kilo-code.improveCode`**: Suggest improvements
- **`kilo-code.addComments`**: Add documentation comments

#### Checkpoint Commands

- **`kilo-code.saveCheckpoint`**: Save conversation state
- **`kilo-code.restoreCheckpoint`**: Restore from checkpoint
- **`kilo-code.listCheckpoints`**: View available checkpoints

#### Integration Commands

- **`kilo-code.generateCommitMessage`**: AI-generated commit message
- **`kilo-code.explainTerminalOutput`**: Explain terminal errors
- **`kilo-code.openDiffView`**: View file comparison

### Command Implementation Pattern

```typescript
context.subscriptions.push(
	vscode.commands.registerCommand("kilo-code.myCommand", async (...args) => {
		try {
			// Implementation
			await performAction(args)

			// Success notification
			vscode.window.showInformationMessage("Action completed")
		} catch (error) {
			// Error handling
			outputChannel.appendLine(`Error: ${error.message}`)
			vscode.window.showErrorMessage("Action failed")
		}
	}),
)
```

### Command Contexts

Commands can be enabled/disabled based on context:

```json
{
	"command": "kilo-code.explainCode",
	"when": "editorHasSelection && !editorReadonly"
}
```

**Available Contexts:**

- `editorHasSelection`: Text is selected
- `resourceLangId`: Specific language
- `kilo-code.taskRunning`: Task is executing
- `kilo-code.apiConfigured`: API key is set

## Code Actions (`registerCodeActions.ts`, `CodeActionProvider.ts`)

Provides quick fixes and refactoring actions in the editor.

### Code Action Provider

```typescript
class CodeActionProvider implements vscode.CodeActionProvider {
	provideCodeActions(
		document: vscode.TextDocument,
		range: vscode.Range | vscode.Selection,
		context: vscode.CodeActionContext,
	): vscode.CodeAction[] {
		const actions: vscode.CodeAction[] = []

		// Add actions based on context
		if (context.diagnostics.length > 0) {
			actions.push(this.createFixAction(context.diagnostics))
		}

		if (range && !range.isEmpty) {
			actions.push(this.createExplainAction(range))
			actions.push(this.createRefactorAction(range))
		}

		return actions
	}
}
```

### Available Code Actions

#### Diagnostic Actions

- **Fix Error**: AI suggests fix for error
- **Fix All**: Fix all errors in file
- **Explain Error**: Explain what's wrong

#### Refactoring Actions

- **Extract Function**: Extract selected code to function
- **Extract Variable**: Extract value to variable
- **Rename Symbol**: AI-suggested rename
- **Simplify Code**: Reduce complexity

#### Documentation Actions

- **Add JSDoc**: Generate documentation
- **Add Type Annotations**: Add TypeScript types
- **Explain Code**: Explain what code does

### Action Kinds

```typescript
// Quick fix
action.kind = vscode.CodeActionKind.QuickFix

// Refactoring
action.kind = vscode.CodeActionKind.Refactor

// Source action
action.kind = vscode.CodeActionKind.Source
```

### Triggering Code Actions

Users trigger via:

- Lightbulb icon (💡)
- `Ctrl+.` / `Cmd+.`
- Right-click → Quick Fix
- Automatically on diagnostics

## Terminal Actions (`registerTerminalActions.ts`)

Provides context-aware actions for terminal output.

### Terminal Action Provider

```typescript
interface TerminalAction {
	id: TerminalActionId
	name: TerminalActionName
	description: string
	prompt: TerminalActionPromptType
}
```

### Available Actions

#### Error Handling

- **Explain Error**: Explain terminal error
- **Fix Command**: Suggest corrected command
- **Search Stack Overflow**: Search for solution

#### Output Analysis

- **Summarize Output**: Condense long output
- **Extract Information**: Pull out key data
- **Save Output**: Save to file

#### Command Suggestions

- **Complete Command**: Suggest command completion
- **Alternative Commands**: Suggest alternatives
- **Add to Script**: Add command to script file

### Implementation

```typescript
vscode.window.registerTerminalLinkProvider({
	provideTerminalLinks(context: vscode.TerminalLinkContext, token: vscode.CancellationToken) {
		// Detect error patterns
		const errorPattern = /ERROR|Error|error|FAIL|fail/

		if (errorPattern.test(context.line)) {
			return [
				{
					startIndex: 0,
					length: context.line.length,
					tooltip: "Click to explain error",
				},
			]
		}
	},

	handleTerminalLink(link: vscode.TerminalLink) {
		// Handle link click
		explainTerminalError(link.data)
	},
})
```

## URI Handling (`handleUri.ts`)

Handles deep links and protocol handlers for the extension.

### URI Format

```
vscode://kilocode.kilo-code/action?param=value
```

### Supported Actions

#### Task Creation

```
vscode://kilocode.kilo-code/task?message=Create%20login%20form
```

#### File Opening

```
vscode://kilocode.kilo-code/file?path=/path/to/file.ts&line=42
```

#### Settings

```
vscode://kilocode.kilo-code/settings?section=apiProvider
```

#### Authentication

```
vscode://kilocode.kilo-code/auth?code=abc123&state=xyz
```

### Implementation

```typescript
export async function handleUri(uri: vscode.Uri) {
	const action = uri.path.substring(1) // Remove leading /
	const params = new URLSearchParams(uri.query)

	switch (action) {
		case "task":
			await createTaskFromUri(params.get("message"))
			break

		case "file":
			await openFileFromUri(params)
			break

		case "auth":
			await handleAuthCallback(params)
			break

		default:
			console.warn("Unknown URI action:", action)
	}
}
```

### Use Cases

- OAuth callback handling
- Share task links
- Integration with external tools
- Deep linking from documentation
- Command-line integration

## Task Handling (`handleTask.ts`)

Creates tasks from various sources.

### Task Sources

#### User Input

```typescript
await createTask({
	text: "Create a login form",
	images: [],
	mode: "code",
})
```

#### Code Selection

```typescript
const editor = vscode.window.activeTextEditor
const selection = editor.selection
const code = editor.document.getText(selection)

await createTask({
	text: `Explain this code:\n\`\`\`\n${code}\n\`\`\``,
	mode: "code",
})
```

#### File Context

```typescript
await createTask({
	text: "Add unit tests",
	mentions: ["@currentFile"],
	mode: "code",
})
```

#### Diagnostics

```typescript
const diagnostics = vscode.languages.getDiagnostics(uri)

await createTask({
	text: `Fix these errors:\n${formatDiagnostics(diagnostics)}`,
	mentions: [`@${uri.fsPath}`],
	mode: "debug",
})
```

### Task Options

```typescript
interface TaskOptions {
	text: string // User message
	images?: string[] // Image paths
	mode?: string // Agent mode
	mentions?: string[] // @mentions
	historyItem?: HistoryItem // Resume from history
	startImmediately?: boolean // Auto-start
}
```

## Human Relay (`humanRelay.ts`)

Enables human-in-the-loop workflows where AI requests human assistance.

### Purpose

Allow AI to ask for human help when:

- Ambiguous requirements
- Design decisions needed
- Sensitive operations
- Information unavailable to AI
- Creative input required

### Implementation

```typescript
async function relayToHuman(question: string, options?: string[]): Promise<string> {
	if (options) {
		// Multiple choice
		const answer = await vscode.window.showQuickPick(options, {
			placeHolder: question,
		})
		return answer || ""
	} else {
		// Free text
		const answer = await vscode.window.showInputBox({
			prompt: question,
			ignoreFocusOut: true,
		})
		return answer || ""
	}
}
```

### Use Cases

- **Clarification**: "Should this be a class or function?"
- **Preference**: "Which UI framework to use?"
- **Confirmation**: "Delete 50 files. Are you sure?"
- **Information**: "What's the API endpoint for production?"
- **Decision**: "Multiple approaches possible. Which do you prefer?"

## Registration Flow

### Activation Sequence

```
1. extension.ts activate() called
2. registerCommands() - Register all commands
3. registerCodeActions() - Setup code action provider
4. registerTerminalActions() - Setup terminal actions
5. handleUri() - Register URI handler
6. Return activation status
```

### Subscription Management

All registrations added to subscriptions for cleanup:

```typescript
function registerCommands({ context, provider }) {
	const commands = [
		vscode.commands.registerCommand("cmd1", handler1),
		vscode.commands.registerCommand("cmd2", handler2),
		vscode.commands.registerCommand("cmd3", handler3),
	]

	context.subscriptions.push(...commands)
}
```

### Deactivation

VS Code automatically disposes all subscriptions when extension deactivates.

## Testing

### Command Testing

```typescript
test("command creates new task", async () => {
	await vscode.commands.executeCommand("kilo-code.newTask")

	assert(provider.currentTask !== undefined)
	assert(provider.currentTask.status === "idle")
})
```

### Code Action Testing

```typescript
test("provides fix action for error", async () => {
	const document = await vscode.workspace.openTextDocument({
		content: 'const x: number = "string"',
	})

	const diagnostic = new vscode.Diagnostic(
		new vscode.Range(0, 0, 0, 26),
		"Type error",
		vscode.DiagnosticSeverity.Error,
	)

	const actions = await codeActionProvider.provideCodeActions(document, diagnostic.range, {
		diagnostics: [diagnostic],
	})

	assert(actions.length > 0)
	assert(actions[0].title.includes("Fix"))
})
```

## Performance

### Lazy Command Registration

Register commands lazily for faster activation:

```typescript
let commandsRegistered = false

function ensureCommandsRegistered() {
	if (!commandsRegistered) {
		registerAllCommands()
		commandsRegistered = true
	}
}
```

### Debounced Actions

Debounce rapid action triggers:

```typescript
let timeout: NodeJS.Timeout

function onSelectionChange() {
	clearTimeout(timeout)
	timeout = setTimeout(() => {
		updateCodeActions()
	}, 300)
}
```

## Best Practices

### Command Naming

- Use `kilo-code.` prefix
- Descriptive names (`generateTests` not `gt`)
- Verb-noun pattern (`createTask` not `taskCreate`)

### Error Handling

- Try-catch in all commands
- User-friendly error messages
- Log to output channel
- Don't crash extension

### User Feedback

- Show progress for long operations
- Provide cancellation for async operations
- Clear success/failure indication
- Contextual help messages

### Testing

- Test all registered commands
- Mock VS Code APIs
- Test error scenarios
- Verify cleanup

## Related Documentation

- [Command Reference](../../commands/README.md)
- [Code Actions Guide](../../guides/code-actions.md)
- [URI Scheme](../../guides/uri-scheme.md)

## Contributing

When adding new commands:

1. Register in `registerCommands.ts`
2. Add to `package.json` contributes
3. Implement handler function
4. Add error handling
5. Write tests
6. Update documentation
