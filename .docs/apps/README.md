# Applications Documentation (`apps/`)

## Overview

The `apps/` directory contains standalone applications within the Kilo Code monorepo. These applications serve different purposes from documentation to testing to web versions of the extension.

## Application Structure

```
apps/
├── kilocode-docs/           # Documentation website
├── web-roo-code/            # Web version of the extension
├── vscode-e2e/              # End-to-end tests for VS Code extension
├── playwright-e2e/          # Browser automation tests
├── storybook/               # UI component documentation
├── web-evals/               # Web-based evaluation dashboard
└── vscode-nightly/          # Nightly build configuration
```

## Applications

### Documentation Website (`kilocode-docs/`)

**Purpose:** Public-facing documentation site for users and developers.

#### Technology Stack

- **Framework**: Astro or Next.js
- **Styling**: TailwindCSS
- **Content**: MDX (Markdown + JSX)
- **Hosting**: Vercel or Netlify
- **Search**: Algolia DocSearch

#### Structure

```
kilocode-docs/
├── src/
│   ├── pages/              # Documentation pages
│   │   ├── index.mdx      # Homepage
│   │   ├── getting-started/
│   │   ├── guides/
│   │   ├── api-reference/
│   │   └── examples/
│   ├── components/         # React components
│   ├── layouts/           # Page layouts
│   └── styles/            # Global styles
├── public/
│   ├── images/
│   └── videos/
└── package.json
```

#### Content Categories

**Getting Started**

- Installation guide
- Quick start tutorial
- First task walkthrough
- Configuration basics

**User Guides**

- Using different modes
- Working with MCP servers
- Managing API keys
- Advanced features
- Troubleshooting

**Developer Guides**

- Extension architecture
- Adding tools
- Creating modes
- Building MCP servers
- Contributing guide

**API Reference**

- Provider API
- Tool API
- Configuration API
- Extension API

**Examples**

- Code generation examples
- Debugging workflows
- Testing patterns
- Integration examples

#### Development

```bash
# Start dev server
pnpm --filter kilocode-docs dev

# Build for production
pnpm --filter kilocode-docs build

# Preview production build
pnpm --filter kilocode-docs preview
```

#### Deployment

```bash
# Deploy to production
pnpm --filter kilocode-docs deploy

# Preview deployment
pnpm --filter kilocode-docs deploy:preview
```

---

### Web Version (`web-roo-code/`)

**Purpose:** Browser-based version of Kilo Code that doesn't require VS Code.

#### Technology Stack

- **Framework**: React + Vite
- **State Management**: Zustand or Redux
- **Styling**: TailwindCSS + shadcn/ui
- **Editor**: Monaco Editor (VS Code's editor)
- **File System**: Browser FS API or virtual FS
- **Backend**: Optional cloud backend

#### Features

**Core Functionality**

- Chat interface with AI
- Code editing with Monaco
- File browser (virtual or cloud storage)
- Terminal emulation
- Diff viewer
- Settings management

**Limitations vs. VS Code Extension**

- No native file system access
- Limited terminal capabilities
- No VS Code extension API
- Browser-based storage only

#### Architecture

```
web-roo-code/
├── src/
│   ├── app/               # Next.js app or main app logic
│   ├── components/        # React components
│   │   ├── chat/         # Chat interface
│   │   ├── editor/       # Code editor
│   │   ├── files/        # File browser
│   │   └── terminal/     # Terminal emulator
│   ├── lib/              # Utilities
│   │   ├── api/          # API clients
│   │   ├── fs/           # File system abstraction
│   │   └── terminal/     # Terminal implementation
│   ├── hooks/            # React hooks
│   ├── stores/           # State management
│   └── styles/           # CSS/TailwindCSS
├── public/
└── package.json
```

#### State Management

```typescript
interface AppState {
	// Current task
	task: Task | null

	// Files
	files: Map<string, FileContent>
	openFiles: string[]
	activeFile: string | null

	// Chat
	messages: ClineMessage[]
	streaming: boolean

	// Settings
	settings: ProviderSettings

	// UI state
	sidebarOpen: boolean
	terminalOpen: boolean
}
```

#### File System

```typescript
// Virtual file system
class VirtualFS {
	private files = new Map<string, VirtualFile>()

	async readFile(path: string): Promise<string>
	async writeFile(path: string, content: string): Promise<void>
	async deleteFile(path: string): Promise<void>
	async listDir(path: string): Promise<string[]>
}

// Browser FS API (modern browsers)
class BrowserFS {
	private handle: FileSystemDirectoryHandle

	async requestAccess(): Promise<void>
	async readFile(path: string): Promise<string>
	async writeFile(path: string, content: string): Promise<void>
}
```

#### Development

```bash
# Start dev server
pnpm --filter web-roo-code dev

# Build for production
pnpm --filter web-roo-code build

# Run tests
pnpm --filter web-roo-code test
```

---

### VS Code E2E Tests (`vscode-e2e/`)

**Purpose:** End-to-end testing of the VS Code extension in a real VS Code environment.

#### Technology Stack

- **Test Framework**: @vscode/test-electron
- **Test Runner**: Mocha or Jest
- **Utilities**: VS Code Test Utilities

#### Test Structure

```
vscode-e2e/
├── src/
│   ├── extension.test.ts     # Extension activation
│   ├── commands.test.ts      # Command execution
│   ├── task.test.ts          # Task creation and execution
│   ├── tools.test.ts         # Tool functionality
│   ├── ui.test.ts            # UI interactions
│   └── integration.test.ts   # Full workflows
├── fixtures/                 # Test fixtures
│   ├── workspaces/
│   └── sample-files/
└── package.json
```

#### Test Examples

**Extension Activation**

```typescript
suite("Extension Activation", () => {
	test("should activate without errors", async () => {
		const ext = vscode.extensions.getExtension("kilocode.kilo-code")
		await ext?.activate()

		assert.ok(ext?.isActive)
	})

	test("should register commands", async () => {
		const commands = await vscode.commands.getCommands()

		assert.ok(commands.includes("kilo-code.plusButtonClicked"))
		assert.ok(commands.includes("kilo-code.newTask"))
	})
})
```

**Task Execution**

```typescript
suite("Task Execution", () => {
	test("should create and run task", async () => {
		await vscode.commands.executeCommand("kilo-code.newTask", "Create a test file")

		// Wait for task to complete
		await waitFor(() => {
			const state = getExtensionState()
			return state.task?.status === "completed"
		}, 30000)

		// Verify file was created
		const files = await vscode.workspace.findFiles("**/*.test.ts")
		assert.ok(files.length > 0)
	})
})
```

#### Running Tests

```bash
# Run all tests
pnpm --filter vscode-e2e test

# Run specific suite
pnpm --filter vscode-e2e test -- --grep "Task Execution"

# Run with coverage
pnpm --filter vscode-e2e test:coverage
```

---

### Playwright E2E Tests (`playwright-e2e/`)

**Purpose:** Browser automation tests for web version and browser integration features.

#### Technology Stack

- **Framework**: Playwright
- **Test Runner**: Playwright Test
- **Browsers**: Chromium, Firefox, WebKit

#### Test Structure

```
playwright-e2e/
├── tests/
│   ├── browser-automation.spec.ts
│   ├── web-app.spec.ts
│   ├── mcp-integration.spec.ts
│   └── performance.spec.ts
├── fixtures/
├── utils/
└── playwright.config.ts
```

#### Test Examples

**Browser Automation**

```typescript
test("should automate browser navigation", async ({ page }) => {
	// Start task with browser action
	await page.goto("http://localhost:3000")
	await page.fill('[data-testid="chat-input"]', 'Navigate to google.com and search for "AI coding"')
	await page.click('[data-testid="send-button"]')

	// Wait for browser to open
	const browserFrame = page.frameLocator('[data-testid="browser-frame"]')
	await browserFrame.waitFor()

	// Verify navigation
	await expect(browserFrame.locator('input[name="q"]')).toBeVisible()
})
```

**Performance Testing**

```typescript
test("should load chat interface quickly", async ({ page }) => {
	const startTime = Date.now()

	await page.goto("http://localhost:3000")
	await page.waitForSelector('[data-testid="chat-ready"]')

	const loadTime = Date.now() - startTime
	expect(loadTime).toBeLessThan(2000) // < 2 seconds
})
```

#### Running Tests

```bash
# Run all tests
pnpm --filter playwright-e2e test

# Run headed mode
pnpm --filter playwright-e2e test -- --headed

# Run specific browser
pnpm --filter playwright-e2e test -- --project=chromium

# Debug mode
pnpm --filter playwright-e2e test -- --debug
```

---

### Storybook (`storybook/`)

**Purpose:** UI component documentation and development environment.

#### Technology Stack

- **Framework**: Storybook 7+
- **Builder**: Vite
- **Addons**:
    - Controls
    - Actions
    - Docs
    - Accessibility
    - Interactions

#### Structure

```
storybook/
├── stories/
│   ├── components/
│   │   ├── Button.stories.tsx
│   │   ├── ChatMessage.stories.tsx
│   │   ├── FileTree.stories.tsx
│   │   └── Terminal.stories.tsx
│   ├── layouts/
│   └── pages/
├── .storybook/
│   ├── main.ts
│   ├── preview.ts
│   └── theme.ts
└── package.json
```

#### Story Examples

**Button Component**

```typescript
export default {
  title: 'Components/Button',
  component: Button,
  argTypes: {
    variant: {
      control: 'select',
      options: ['primary', 'secondary', 'danger']
    }
  }
} as Meta<typeof Button>

export const Primary: Story = {
  args: {
    variant: 'primary',
    children: 'Click me'
  }
}

export const WithIcon: Story = {
  args: {
    variant: 'primary',
    icon: <SendIcon />,
    children: 'Send'
  }
}
```

**Chat Message Component**

```typescript
export const UserMessage: Story = {
	args: {
		message: {
			role: "user",
			content: [
				{
					type: "text",
					text: "Create a login form",
				},
			],
			timestamp: Date.now(),
		},
	},
}

export const AssistantWithTool: Story = {
	args: {
		message: {
			role: "assistant",
			content: [
				{
					type: "text",
					text: "I'll create a login form for you.",
				},
				{
					type: "tool_use",
					name: "write_to_file",
					input: { path: "LoginForm.tsx" },
				},
			],
		},
	},
}
```

#### Running Storybook

```bash
# Start dev server
pnpm --filter storybook dev

# Build static site
pnpm --filter storybook build

# Preview built site
pnpm --filter storybook preview
```

---

### Web Evals Dashboard (`web-evals/`)

**Purpose:** Web-based dashboard for viewing and analyzing evaluation results.

#### Technology Stack

- **Framework**: Next.js or React + Vite
- **Charts**: Recharts or Chart.js
- **Data**: Fetched from eval runs
- **Styling**: TailwindCSS

#### Features

**Dashboard Views**

- Overview metrics
- Model comparison
- Task success rates
- Cost analysis
- Performance trends
- Error patterns

**Visualizations**

```typescript
// Model comparison chart
<ComparisonChart
  models={['claude-3-5-sonnet', 'gpt-4', 'gpt-3.5-turbo']}
  metrics={['accuracy', 'speed', 'cost']}
  data={evalResults}
/>

// Success rate over time
<TrendChart
  metric="success_rate"
  timeRange="30d"
  data={historicalData}
/>

// Cost breakdown
<PieChart
  data={[
    { name: 'Input tokens', value: 45 },
    { name: 'Output tokens', value: 35 },
    { name: 'API calls', value: 20 }
  ]}
/>
```

#### Data Structure

```typescript
interface EvalDashboardData {
	summary: {
		totalRuns: number
		successRate: number
		avgCost: number
		avgTime: number
	}
	modelComparison: ModelComparisonData[]
	tasks: TaskResultData[]
	trends: TrendData[]
	errors: ErrorData[]
}
```

---

### Nightly Builds (`vscode-nightly/`)

**Purpose:** Configuration and automation for nightly extension builds.

#### Features

- Automated nightly builds
- Pre-release channel
- Latest features testing
- Feedback collection

#### Build Configuration

```json
{
	"name": "kilo-code-nightly",
	"displayName": "Kilo Code (Nightly)",
	"version": "4.116.0-nightly.20240115",
	"icon": "assets/icons/logo-nightly.png",
	"repository": {
		"type": "git",
		"url": "https://github.com/Kilo-Org/kilocode"
	}
}
```

#### Automation

```yaml
# .github/workflows/nightly.yml
name: Nightly Build

on:
    schedule:
        - cron: "0 2 * * *" # 2 AM UTC daily
    workflow_dispatch:

jobs:
    build:
        runs-on: ubuntu-latest
        steps:
            - uses: actions/checkout@v3
            - uses: pnpm/action-setup@v2
            - run: pnpm install
            - run: pnpm build:nightly
            - run: pnpm publish:nightly
```

---

## Development Workflow

### Running All Apps

```bash
# Run all apps in dev mode
pnpm dev

# Build all apps
pnpm build

# Test all apps
pnpm test
```

### Working with Specific App

```bash
# Navigate to app
cd apps/web-roo-code

# Install dependencies
pnpm install

# Run dev server
pnpm dev

# Or use filter from root
pnpm --filter web-roo-code dev
```

## Deployment

### Documentation Site

```bash
# Deploy to production
pnpm --filter kilocode-docs deploy

# Domain: https://kilocode.ai
```

### Web App

```bash
# Deploy to Vercel/Netlify
pnpm --filter web-roo-code deploy

# Domain: https://app.kilocode.ai
```

### Nightly Extension

```bash
# Publish to VS Code Marketplace
pnpm --filter vscode-nightly publish
```

## Best Practices

### Shared Components

- Extract common components to packages
- Use consistent design system
- Share utilities across apps
- Maintain component library

### Testing

- E2E tests for critical flows
- Visual regression tests for UI
- Performance benchmarks
- Accessibility testing

### Performance

- Code splitting
- Lazy loading
- Image optimization
- Bundle size monitoring

### CI/CD

- Automated testing
- Preview deployments
- Staging environments
- Rollback capabilities

## Related Documentation

- [Monorepo Guide](../guides/monorepo.md)
- [Testing Strategy](../guides/testing.md)
- [Deployment Guide](../guides/deployment.md)
- [Web Development](../guides/web-development.md)
