# Kilo Code Quick Start Guide

## For New Contributors

This guide helps you quickly understand and contribute to the Kilo Code codebase.

## 5-Minute Overview

### What is Kilo Code?

Kilo Code is an AI coding assistant VS Code extension that:

- Generates code from natural language
- Debugs and fixes errors autonomously
- Automates repetitive tasks
- Integrates with 400+ AI models
- Extends via Model Context Protocol (MCP) servers

### Key Technologies

- **Language**: TypeScript
- **Framework**: VS Code Extension API
- **UI**: React (in webview)
- **AI**: Anthropic Claude, OpenAI GPT, OpenRouter
- **Build**: esbuild, pnpm
- **Test**: Vitest

## Project Structure (10-Minute Tour)

```
kilocode/
├── src/                          # Main extension code
│   ├── extension.ts             # 🚀 Entry point - START HERE
│   ├── core/                    # 🧠 AI agent logic
│   │   ├── task/                #    Task execution
│   │   ├── tools/               #    Tool implementations
│   │   ├── prompts/             #    Prompt engineering
│   │   └── webview/             #    UI controller
│   ├── services/                # 🔧 Supporting services
│   │   ├── code-index/          #    Code intelligence
│   │   ├── mcp/                 #    MCP integration
│   │   └── browser/             #    Browser automation
│   ├── api/                     # 🌐 AI provider APIs
│   │   └── providers/           #    Model providers
│   └── integrations/            # 🔌 VS Code integration
│       ├── editor/              #    Editor features
│       └── terminal/            #    Terminal features
├── webview-ui/                  # ⚛️ React UI
└── apps/                        # 📦 Additional apps
    └── vscode-e2e/              #    E2E tests
```

## Common Tasks

### 1. Run Extension (Development)

```bash
# Install dependencies
pnpm install

# Open in VS Code
code .

# Press F5 to debug
# This opens Extension Development Host
```

### 2. Make a Change

**Example: Add a new tool**

```typescript
// 1. Create tool file: src/core/tools/myTool.ts
export async function myTool(args: {param: string}) {
  // Implementation
  return { success: true }
}

// 2. Register in prompt: src/core/prompts/tools/index.ts
export const TOOLS = [
  ...existingTools,
  {
    name: 'my_tool',
    description: 'Does something useful',
    input_schema: {
      type: 'object',
      properties: {
        param: { type: 'string', description: 'A parameter' }
      }
    }
  }
]

// 3. Handle in task: src/core/task/Task.ts
case 'my_tool':
  const result = await myTool(toolInput)
  return result
```

### 3. Run Tests

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

### 4. Build Extension

```bash
# Development build
pnpm build:dev

# Production build
pnpm build

# Install locally
code --install-extension "$(ls -1v bin/kilo-code-*.vsix | tail -n1)"
```

## Understanding the Code Flow

### User Creates Task

```
1. User types message in webview (webview-ui/)
   ↓
2. Message posted to extension (src/core/webview/ClineProvider.ts)
   ↓
3. Task created or resumed (src/core/task/Task.ts)
   ↓
4. Prompt assembled with context (src/core/prompts/)
   ↓
5. Sent to AI model (src/api/)
   ↓
6. AI response streamed back (src/api/transform/stream.ts)
   ↓
7. Tools executed (src/core/tools/)
   ↓
8. Results sent to AI (loop continues)
   ↓
9. Task completes (attemptCompletionTool)
   ↓
10. UI updated (webview-ui/)
```

### Tool Execution Example

```typescript
// File: src/core/task/Task.ts

// 1. AI returns tool_use
{
  type: 'tool_use',
  name: 'read_file',
  input: { path: '/path/to/file.ts' }
}

// 2. Task routes to tool handler
const toolHandler = this.getToolHandler(toolName)
const result = await toolHandler(toolInput)

// 3. Tool executes (src/core/tools/readFileTool.ts)
async function readFileTool({ path }) {
  const content = await fs.readFile(path, 'utf-8')
  return { content }
}

// 4. Result formatted as tool_result
{
  type: 'tool_result',
  tool_use_id: 'toolu_123',
  content: '...'
}

// 5. Sent back to AI for next decision
```

## Key Files to Know

### Extension Entry (`src/extension.ts`)

- Activates extension
- Initializes services
- Registers commands
- Sets up providers

### Main Task Controller (`src/core/task/Task.ts`)

- Manages conversation
- Executes AI requests
- Handles tool calls
- Manages state

### UI Controller (`src/core/webview/ClineProvider.ts`)

- Renders webview
- Routes messages
- Updates UI state
- Manages panel

### Tool Definitions (`src/core/tools/`)

- Individual tool files
- Each implements one capability
- Clear input/output contracts

### Prompt Engineering (`src/core/prompts/`)

- System prompt
- Tool descriptions
- Context sections
- Response formatting

## Development Workflow

### 1. Setup

```bash
git clone https://github.com/Kilo-Org/kilocode.git
cd kilocode
pnpm install
```

### 2. Branch

```bash
git checkout -b feature/my-feature
```

### 3. Develop

- Make changes
- Test locally (F5)
- Write tests
- Check types

### 4. Commit

```bash
# Follow conventional commits
git commit -m "feat: add new tool for X"

# Or use changeset
pnpm changeset
```

### 5. Push & PR

```bash
git push origin feature/my-feature
# Create PR on GitHub
```

## Debugging Tips

### Extension Not Loading

```bash
# Check output
View → Output → Kilo Code

# Check developer tools
Help → Toggle Developer Tools

# Reload window
Cmd/Ctrl + Shift + P → Reload Window
```

### Webview Not Updating

```bash
# Check webview console
Right-click webview → Inspect Element

# Check message passing
console.log in both extension and webview

# Verify CSP isn't blocking
Check console for CSP violations
```

### Tool Not Working

```bash
# Check tool registration
src/core/prompts/tools/

# Verify handler exists
src/core/tools/

# Check execution in task
src/core/task/Task.ts

# Review logs
Output channel → Kilo Code
```

## Common Patterns

### Adding a Service

```typescript
// 1. Create service class
export class MyService {
	private static instance: MyService

	static getInstance() {
		if (!this.instance) {
			this.instance = new MyService()
		}
		return this.instance
	}

	async initialize() {
		// Setup
	}

	dispose() {
		// Cleanup
	}
}

// 2. Initialize in extension.ts
const myService = await MyService.getInstance()
await myService.initialize()
context.subscriptions.push(myService)
```

### Adding a Command

```typescript
// 1. Register command
context.subscriptions.push(
  vscode.commands.registerCommand(
    'kilo-code.myCommand',
    async () => {
      // Implementation
    }
  )
)

// 2. Add to package.json
{
  "contributes": {
    "commands": [
      {
        "command": "kilo-code.myCommand",
        "title": "Kilo Code: My Command"
      }
    ]
  }
}
```

### Adding Configuration

```typescript
// 1. Add to package.json
{
  "contributes": {
    "configuration": {
      "properties": {
        "kilo-code.mySetting": {
          "type": "string",
          "default": "value",
          "description": "My setting"
        }
      }
    }
  }
}

// 2. Access in code
const config = vscode.workspace.getConfiguration('kilo-code')
const mySetting = config.get<string>('mySetting')
```

## Resources

### Documentation

- [Complete Documentation](.docs/README.md)
- [Architecture](.docs/architecture/README.md)
- [Core System](.docs/src/core/README.md)
- [Services](.docs/src/services/README.md)

### External Resources

- [VS Code Extension API](https://code.visualstudio.com/api)
- [TypeScript Handbook](https://www.typescriptlang.org/docs/)
- [Anthropic API Docs](https://docs.anthropic.com/)
- [Model Context Protocol](https://modelcontextprotocol.io/)

### Community

- [Discord](https://discord.gg/Ja6BkfyTzJ)
- [GitHub Issues](https://github.com/Kilo-Org/kilocode/issues)
- [Discussions](https://github.com/Kilo-Org/kilocode/discussions)

## Next Steps

### Beginner Tasks

- Fix a small bug
- Add tests for existing code
- Improve documentation
- Add a simple tool

### Intermediate Tasks

- Add a new AI provider
- Implement a code action
- Create a new mode
- Optimize performance

### Advanced Tasks

- Architect a major feature
- Refactor a core system
- Design new APIs
- Implement MCP server

## Getting Help

1. **Check Documentation**: Look in `.docs/` folder
2. **Search Issues**: GitHub issues for similar problems
3. **Ask on Discord**: Active community support
4. **Read Code**: Comments and existing implementations
5. **Debug**: Use VS Code debugger extensively

## Pro Tips

### 🚀 Faster Development

- Use `NODE_ENV=development` for hot reload
- Keep Extension Development Host open
- Use VS Code tasks for common operations
- Set up keyboard shortcuts

### 🧪 Better Testing

- Write tests as you code
- Use VS Code debugger in tests
- Mock external dependencies
- Test error cases

### 📚 Code Organization

- Keep files small and focused
- Follow existing patterns
- Add TSDoc comments
- Group related code

### 🐛 Debugging Tricks

- Use `debugger` statement
- Check Output channel
- Inspect webview console
- Enable verbose logging

## Anti-Patterns to Avoid

❌ **Don't:**

- Modify `node_modules`
- Skip tests
- Ignore TypeScript errors
- Leave console.log in code
- Hardcode API keys
- Circular dependencies
- Synchronous file operations

✅ **Do:**

- Follow TypeScript strict mode
- Write self-documenting code
- Use async/await
- Handle errors properly
- Add tests for new features
- Keep dependencies minimal
- Use VS Code APIs when available

## Checklist for New Features

- [ ] Feature implemented
- [ ] Tests added
- [ ] Types defined
- [ ] Error handling
- [ ] Documentation updated
- [ ] Changeset created
- [ ] Linting passes
- [ ] Builds successfully
- [ ] Tested manually
- [ ] PR description complete

## Welcome to Kilo Code!

You're now ready to contribute. Start small, ask questions, and have fun building the future of AI coding assistants!

Remember: **The best way to learn is to build.** Pick a feature you'd like to add, and dive in!
