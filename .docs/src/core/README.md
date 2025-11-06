# Core System Documentation (`src/core/`)

## Overview

The `core/` directory contains the heart of Kilo Code's AI agent functionality. This is where the autonomous coding assistant's intelligence, decision-making, tool execution, and conversation management are implemented.

## Directory Structure

```
core/
├── assistant-message/        # AI response parsing and formatting
├── checkpoints/              # Conversation state checkpointing
├── condense/                 # Context window compression
├── config/                   # Configuration management
├── context/                  # Context gathering and management
├── context-tracking/         # File and workspace tracking
├── diff/                     # Code diff strategies
├── environment/              # System environment detection
├── ignore/                   # .rooignore pattern matching
├── kilocode/                 # Kilo-specific functionality
├── mentions/                 # @mention parsing and processing
├── message-queue/            # Message queuing system
├── prompts/                  # Prompt engineering and templates
├── protect/                  # File protection mechanisms
├── slash-commands/           # /command parsing
├── sliding-window/           # Context window management
├── task/                     # Task lifecycle and execution
├── task-persistence/         # Task state persistence
├── tools/                    # AI tool implementations
└── webview/                  # Webview UI controller
```

## Core Concepts

### Task (`task/Task.ts`)

The `Task` class is the central execution unit for all AI interactions. Each conversation with the AI agent is managed by a Task instance.

#### Responsibilities

- **Conversation Management**: Maintains message history between user and AI
- **Tool Execution**: Calls appropriate tools based on AI decisions
- **State Persistence**: Saves/restores conversation state
- **Error Handling**: Manages API errors, context window issues, rate limits
- **Token Tracking**: Monitors token usage and costs
- **Checkpoint Integration**: Creates restore points for rollback

#### Lifecycle States

```typescript
enum TaskStatus {
	IDLE = "idle", // No active task
	RUNNING = "running", // Task executing
	WAITING = "waiting", // Awaiting user input
	COMPLETED = "completed", // Successfully finished
	FAILED = "failed", // Error occurred
	ABORTED = "aborted", // User cancelled
}
```

#### Key Methods

- `execute()`: Main execution loop for AI interactions
- `say()`: Send message to user without requiring response
- `ask()`: Request user input or approval
- `recursivelyMakeRooApiRequest()`: Core API interaction with retry logic
- `addToPrompt()`: Queue messages for next AI request
- `attemptCompletion()`: Signal task completion
- `abortTask()`: Cancel running task

### Tools (`tools/`)

Tools are the AI agent's capabilities - functions it can call to interact with the codebase, file system, terminal, and more.

#### Available Tools

##### File Manipulation

- **`readFileTool`**: Read file contents with syntax highlighting
- **`writeToFileTool`**: Create new files or overwrite existing
- **`editFileTool`**: Search-and-replace edits to existing files
- **`searchAndReplaceTool`**: Advanced multi-file editing
- **`applyDiffTool`**: Apply unified diff patches
- **`multiApplyDiffTool`**: Apply multiple diff patches atomically
- **`insertContentTool`**: Insert content at specific line positions

##### Code Intelligence

- **`codebaseSearchTool`**: Semantic search across codebase
- **`listFilesTool`**: List files matching patterns
- **`listCodeDefinitionNamesTool`**: Extract function/class names
- **`searchFilesTool`**: Text search within files

##### Execution

- **`executeCommandTool`**: Run terminal commands
- **`browserActionTool`**: Control browser automation
- **`useMcpToolTool`**: Call MCP server tools

##### Task Management

- **`newTaskTool`**: Create subtasks
- **`updateTodoListTool`**: Manage task checklist
- **`attemptCompletionTool`**: Mark task as complete
- **`askFollowupQuestionTool`**: Request clarification

##### Meta Operations

- **`switchModeTool`**: Change agent mode (Architect/Coder/Debugger)
- **`runSlashCommandTool`**: Execute slash commands
- **`newRuleTool`**: Add custom rules
- **`fetchInstructionsTool`**: Load external instructions
- **`condenseTool`**: Compress conversation history

##### Kilo-Specific

- **`generateImageTool`**: AI image generation
- **`reportBugTool`**: Bug reporting integration

#### Tool Execution Flow

```
1. AI decides to use tool
2. Task validates tool call
3. Tool handler executes
4. Results formatted
5. Sent back to AI as tool result
6. AI processes and decides next action
```

#### Tool Safety

- **Approval Required**: Destructive operations need user confirmation
- **Allowlist**: Commands checked against allowed list
- **File Protection**: `.rooprotect` patterns prevent editing
- **Ignore Patterns**: `.rooignore` excludes files from operations
- **Repetition Detection**: Prevents infinite tool loops

### Prompts (`prompts/`)

Sophisticated prompt engineering to guide AI behavior.

#### Components

##### System Prompt (`system.ts`)

The core instruction set defining the AI's personality, capabilities, and constraints.

**Key Sections:**

- **Identity**: Who the AI is and its purpose
- **Capabilities**: What tools are available
- **Guidelines**: How to interact with users
- **Rules**: Constraints and safety measures
- **Output Format**: Expected response structure

##### Tool Prompts (`prompts/tools/`)

Detailed descriptions of each tool for the AI:

- Function signature and parameters
- Usage examples
- Success/failure scenarios
- Best practices
- Edge cases to avoid

##### Section Prompts (`prompts/sections/`)

Contextual information injected into conversations:

- **Environment Details**: OS, shell, current directory
- **File Context**: Open files, selections, diagnostics
- **Workspace Info**: Project structure, git status
- **Custom Instructions**: User-defined behavior
- **Kilo Rules**: Project-specific rules from `.kilorules/`

##### Response Formatting (`responses.ts`)

Structures for AI responses:

- Text messages to user
- Tool call formatting
- Error message templates
- Completion acknowledgments

#### Prompt Assembly

```typescript
const messages = [
	systemPrompt, // Core instructions
	...contextSections, // Environment and workspace
	...conversationHistory, // Past messages
	userMessage, // Current request
]
```

### Configuration Management (`config/`)

#### ContextProxy

Reactive configuration access layer:

```typescript
class ContextProxy {
	// Get configuration value
	get<T>(key: string): T

	// Watch for changes
	onChange(callback: (key: string) => void)

	// Update configuration
	set(key: string, value: any)
}
```

**Managed Settings:**

- API provider and model selection
- Custom modes and instructions
- MCP server configurations
- File patterns (ignore, protect)
- UI preferences

#### ProviderSettingsManager

Manages AI model provider configurations:

- API keys and endpoints
- Model selection per provider
- Token limits and pricing
- Request timeouts
- Streaming preferences

#### CustomModesManager

Handles user-defined agent modes:

```typescript
interface CustomMode {
	name: string
	slug: string
	description: string
	instructions: string
	roleDefinition?: string
}
```

### Checkpoints (`checkpoints/`)

Save and restore conversation state for rollback capability.

#### Checkpoint Types

- **Manual**: User-initiated save points
- **Automatic**: Created before risky operations
- **Shadow**: Hidden background saves

#### Checkpoint Data

```typescript
interface Checkpoint {
	id: string
	timestamp: number
	taskId: string
	conversationHistory: ClineMessage[]
	fileSystemState: Map<string, FileState>
	terminalState: TerminalSnapshot
	metadata: CheckpointMetadata
}
```

#### Operations

- **Save**: `checkpointSave(task, label)`
- **Restore**: `checkpointRestore(task, checkpointId)`
- **Diff**: `checkpointDiff(task, checkpointId)` - Show changes
- **List**: Get all checkpoints for task

### Context Management (`context/`)

#### Context Window Handling

AI models have limited context windows. Kilo Code manages this through:

**Strategies:**

1. **Sliding Window**: Remove oldest messages first
2. **Summarization**: Compress old messages into summaries
3. **Selective Inclusion**: Prioritize recent and important context
4. **Forced Reduction**: Emergency compression on errors

**Context Budget:**

```
Total Context = System Prompt + Tools + Environment + History
Target: 75% of model's max input tokens
Reserve: 25% for model output
```

#### Context Tracking (`context-tracking/`)

Tracks which files are relevant to current task:

```typescript
class FileContextTracker {
	// Mark file as relevant
	addFile(path: string)

	// Get all tracked files
	getTrackedFiles(): string[]

	// Check if file is tracked
	isTracked(path: string): boolean
}
```

### Diff Strategies (`diff/`)

Multiple strategies for code changes to handle various scenarios.

#### Available Strategies

1. **Multi-Search-Replace**

    - Multiple search-replace operations in single file
    - Best for scattered small changes
    - Validates all searches before applying

2. **Multi-File-Search-Replace**

    - Search-replace across multiple files
    - Atomic (all succeed or all fail)
    - Best for refactoring across files

3. **Unified Diff**

    - Standard diff patch format
    - Best for large changes
    - Requires precise line matching

4. **Insert Content**
    - Insert at specific line numbers
    - Best for adding new code sections
    - Minimal context required

### Assistant Message Parsing (`assistant-message/`)

Parses and validates AI responses.

#### Message Types

- **Text**: Plain text response to user
- **Tool Use**: Request to execute a tool
- **Thinking**: Internal reasoning (shown to user)
- **Completion**: Task completion signal

#### Validation

- Ensures proper message structure
- Validates tool call parameters
- Detects malformed responses
- Provides helpful error messages

### Mentions (`mentions/`)

Processes `@mention` syntax for referencing context.

#### Supported Mentions

- `@file`: Reference specific file
- `@folder`: Include entire folder
- `@url`: Fetch web content
- `@problems`: Include VS Code diagnostics
- `@terminal`: Include terminal output
- `@search`: Code search results

#### Processing

```typescript
const content = await processKiloUserContentMentions(userMessage, workspacePath, contextProxy)
```

### Slash Commands (`slash-commands/`)

Built-in commands triggered with `/` prefix.

#### Available Commands

- `/reset`: Clear conversation history
- `/clear`: Clear visible messages
- `/mode <mode>`: Switch agent mode
- `/task <description>`: Create new task
- `/workflow <name>`: Load workflow from `.kilorules/`

### Message Queue (`message-queue/`)

Manages message queuing and sequencing.

#### Purpose

- Queue multiple user messages
- Process messages in order
- Prevent race conditions
- Handle interruptions

### Ignore & Protect

#### RooIgnoreController (`ignore/`)

Implements `.rooignore` file pattern matching:

- Excludes files from operations
- Prevents unnecessary file reads
- Speeds up search operations
- Respects `.gitignore` patterns

#### RooProtectedController (`protect/`)

Implements `.rooprotect` file protection:

- Prevents AI from editing critical files
- Configuration files (package.json, etc.)
- Production code
- Database migrations

### Webview Communication (`webview/`)

#### ClineProvider

Main controller for webview panel:

**Responsibilities:**

- Render React UI
- Handle user interactions
- Route messages to Task
- Update UI state
- Manage panel visibility

**Message Types:**

```typescript
// Extension → Webview
- state: Current extension state
- action: User action response
- partialMessage: Streaming update

// Webview → Extension
- newTask: Start new conversation
- askResponse: User's response to ask
- cancelTask: Abort running task
- retryTask: Retry failed request
```

## Key Workflows

### Starting a New Task

```
1. User enters message in webview
2. ClineProvider receives message
3. Creates new Task instance
4. Task assembles prompt with context
5. Sends request to AI model
6. Streams response back to UI
7. AI requests tool execution
8. Task executes tools
9. Results sent back to AI
10. Loop continues until completion
```

### Tool Execution Loop

```
1. AI returns tool_use content block
2. Task validates tool name and parameters
3. Tool handler function called
4. Operation performed (file edit, command, etc.)
5. Result captured
6. Formatted as tool_result
7. Added to conversation
8. Sent to AI for next decision
```

### Context Window Management

```
1. Calculate total token usage
2. If approaching limit:
   a. Identify oldest messages
   b. Summarize or remove them
   c. Maintain critical context
3. If hard limit exceeded:
   a. Force compression (remove 25%)
   b. Retry request
   c. Fail if still too large
```

### Error Recovery

```
1. API request fails
2. Check error type:
   - Rate limit: Exponential backoff
   - Context window: Compress and retry
   - Auth error: Prompt re-authentication
   - Network error: Retry with timeout
3. Max retries exceeded: Report to user
4. Allow user to modify and retry
```

## Performance Optimizations

### Caching

- **File Content**: Cache recently read files
- **Search Results**: Cache code search results
- **Token Counts**: Cache prompt token calculations
- **API Responses**: Cache for identical requests (future)

### Lazy Loading

- Defer loading non-critical tools
- Load MCP servers on-demand
- Lazy load large dependencies
- Stream responses instead of buffering

### Debouncing

- File watcher events debounced
- UI updates batched
- Search requests debounced
- Autosave after idle period

## Error Handling

### Error Categories

1. **API Errors**

    - Rate limiting
    - Context window exceeded
    - Invalid API key
    - Model not found
    - Payment required

2. **Tool Errors**

    - File not found
    - Permission denied
    - Command execution failed
    - Invalid parameters

3. **System Errors**
    - Out of memory
    - Disk full
    - Network timeout
    - Extension crash

### Recovery Strategies

- Automatic retry with backoff
- Graceful degradation
- User notification with actionable steps
- Telemetry for debugging

## Security Considerations

### Command Execution

- Allowlist of approved commands
- User approval for new commands
- No arbitrary code execution
- Sandboxed terminal environment

### File Access

- Respects workspace boundaries
- Honors `.gitignore` patterns
- Protection for sensitive files
- No access to system files

### API Key Management

- Stored in secure storage
- Never logged or transmitted
- Encrypted at rest
- Cleared on logout

## Testing

### Unit Tests

Located in `__tests__/` folders:

- Tool execution logic
- Prompt generation
- Message parsing
- Configuration management
- Diff strategies

### Integration Tests

- API interactions (mocked)
- File system operations
- Terminal command execution
- Checkpoint save/restore

### Test Coverage Goals

- Core business logic: >80%
- Tool implementations: >70%
- Error handling: >60%
- UI integration: >50%

## Common Issues and Solutions

### Task Won't Complete

- Check for infinite tool loops
- Review tool repetition detector logs
- Ensure completion tool is available
- Verify AI has enough context

### Context Window Errors

- Enable condensation in settings
- Reduce custom instructions length
- Limit file mentions
- Use smaller model

### Tool Execution Failures

- Check file permissions
- Verify paths are correct
- Review command allowlist
- Check terminal state

### Performance Degradation

- Clear old conversation history
- Disable unnecessary MCP servers
- Reduce file watching scope
- Check memory usage

## Best Practices

### Prompt Engineering

- Be specific in instructions
- Provide examples
- Define success criteria
- Include error handling guidance

### Tool Design

- Single responsibility per tool
- Clear parameter descriptions
- Comprehensive error handling
- Idempotent where possible

### State Management

- Minimize global state
- Use immutable data structures
- Clear boundaries between modules
- Explicit state transitions

### Error Messages

- User-friendly language
- Actionable suggestions
- Link to documentation
- Include error codes

## Future Enhancements

### Planned Features

- Multi-agent collaboration
- Custom tool creation by users
- Improved context summarization
- Parallel tool execution
- Advanced caching strategies

### Research Areas

- Semantic caching
- Predictive context loading
- Automatic mode switching
- Self-healing error recovery
- Context-aware compression

## Related Documentation

- [Task System Deep Dive](./task/README.md)
- [Tool Implementation Guide](./tools/README.md)
- [Prompt Engineering Guide](./prompts/README.md)
- [Configuration Reference](./config/README.md)
- [Checkpoint System](./checkpoints/README.md)
