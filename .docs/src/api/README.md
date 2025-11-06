# API Documentation (`src/api/`)

## Overview

The `api/` directory provides an abstraction layer for interacting with various AI model providers. It handles request formatting, response parsing, streaming, error handling, and provider-specific quirks.

## Directory Structure

```
api/
├── index.ts                  # Main API handler factory
├── providers/                # Provider implementations
│   ├── anthropic.ts         # Anthropic (Claude)
│   ├── openai.ts            # OpenAI (GPT)
│   ├── openrouter.ts        # OpenRouter (multi-model gateway)
│   ├── bedrock.ts           # AWS Bedrock
│   ├── vertex.ts            # Google Vertex AI
│   ├── gemini.ts            # Google Gemini
│   ├── openai-native.ts     # OpenAI-compatible endpoints
│   ├── ollama.ts            # Ollama (local models)
│   ├── lmstudio.ts          # LM Studio (local models)
│   ├── glama.ts             # Glama API
│   └── fetchers/            # Model list fetchers
└── transform/               # Request/response transformers
    ├── stream.ts            # Streaming response handling
    ├── image-cleaning.ts    # Image data processing
    └── ...
```

## Core Concepts

### API Handler (`buildApiHandler`)

The main factory function that creates provider-specific API handlers.

```typescript
function buildApiHandler(settings: ProviderSettings, context: vscode.ExtensionContext): ApiHandler {
	// Returns appropriate handler based on settings.apiProvider
}
```

### ApiHandler Interface

All providers implement this common interface:

```typescript
interface ApiHandler {
	// Create a chat completion
	createMessage(
		systemPrompt: string,
		messages: ApiMessage[],
		options?: CreateMessageOptions,
	): AsyncGenerator<ApiStream, void, unknown>

	// Get cost estimate
	getEstimatedCost(tokens: TokenUsage): number

	// Validate configuration
	validateConfiguration(): Promise<ValidationResult>

	// Get available models
	getModels(): Promise<ModelInfo[]>
}
```

### Message Format

Standardized message format across all providers:

```typescript
interface ApiMessage {
	role: "user" | "assistant" | "system"
	content: MessageContent[]
}

type MessageContent = TextContent | ImageContent | ToolUseContent | ToolResultContent

interface TextContent {
	type: "text"
	text: string
}

interface ImageContent {
	type: "image"
	source: {
		type: "base64"
		media_type: string
		data: string
	}
}
```

## Provider Implementations

### Anthropic (Claude)

**Models:**

- Claude 3.5 Sonnet (latest, most capable)
- Claude 3 Opus (previous flagship)
- Claude 3 Sonnet
- Claude 3 Haiku (fast, economical)

**Features:**

- Extended context window (200K tokens)
- Vision capabilities (image input)
- Tool use (function calling)
- Streaming responses
- System prompts
- Prompt caching (beta)

**Configuration:**

```typescript
{
  apiProvider: "anthropic",
  anthropicApiKey: "sk-ant-...",
  apiModelId: "claude-3-5-sonnet-20241022",
  anthropicBaseUrl?: "https://api.anthropic.com" // Optional
}
```

**Special Considerations:**

- Requires explicit tool_choice for forced tool use
- Supports thinking blocks for reasoning
- Has strict content filtering
- Rate limits vary by tier

### OpenAI (GPT)

**Models:**

- GPT-4 Turbo
- GPT-4
- GPT-3.5 Turbo
- GPT-4 with vision

**Features:**

- Function calling
- JSON mode
- Streaming
- Vision (GPT-4V)
- DALL-E integration
- Whisper transcription

**Configuration:**

```typescript
{
  apiProvider: "openai",
  openAiApiKey: "sk-...",
  apiModelId: "gpt-4-turbo",
  openAiBaseUrl?: "https://api.openai.com/v1"
}
```

**Special Considerations:**

- Uses different message format than Anthropic
- Function calling has different structure
- No native system message in older models
- Content filtering can block responses

### OpenRouter

**Purpose:**
Multi-model gateway providing access to 100+ models from various providers.

**Supported Models:**

- Claude (all variants)
- GPT (all variants)
- Llama
- Mixtral
- Gemini
- Many more

**Features:**

- Model fallback
- Cost optimization
- Rate limit management
- Model comparison
- Usage analytics

**Configuration:**

```typescript
{
  apiProvider: "openrouter",
  openRouterApiKey: "sk-or-...",
  apiModelId: "anthropic/claude-3.5-sonnet",
  openRouterBaseUrl?: "https://openrouter.ai/api/v1"
}
```

**Benefits:**

- Single API for multiple providers
- Automatic fallback on failures
- Transparent pricing
- No rate limit concerns
- Model experimentation

### AWS Bedrock

**Purpose:**
Enterprise-grade AI model hosting on AWS infrastructure.

**Supported Models:**

- Anthropic Claude (via Bedrock)
- Amazon Titan
- Mistral
- Llama 2
- Cohere

**Features:**

- AWS IAM authentication
- VPC support
- Compliance certifications
- Model customization
- Usage tracking via CloudWatch

**Configuration:**

```typescript
{
  apiProvider: "bedrock",
  awsRegion: "us-east-1",
  awsAccessKey: "AKIA...",
  awsSecretKey: "...",
  apiModelId: "anthropic.claude-3-sonnet-20240229-v1:0"
}
```

**Special Considerations:**

- Requires AWS credentials
- Region-specific model availability
- Different pricing than Anthropic direct
- IAM permissions required

### Google Vertex AI

**Purpose:**
Google Cloud's managed AI platform.

**Supported Models:**

- Gemini Pro
- Gemini Ultra
- PaLM 2
- Codey (code-specific)

**Features:**

- GCP authentication
- Multimodal capabilities
- Tuning and customization
- Integration with GCP services

**Configuration:**

```typescript
{
  apiProvider: "vertex",
  gcpProjectId: "my-project",
  gcpRegion: "us-central1",
  vertexCredentials: {...},
  apiModelId: "gemini-pro"
}
```

### Ollama (Local Models)

**Purpose:**
Run AI models locally on your machine.

**Supported Models:**

- Llama 2 (all sizes)
- Mistral
- CodeLlama
- Phi-2
- Custom models

**Features:**

- Complete privacy
- No API costs
- Offline operation
- Custom model fine-tuning
- Fast local inference

**Configuration:**

```typescript
{
  apiProvider: "ollama",
  ollamaBaseUrl: "http://localhost:11434",
  apiModelId: "llama2:13b"
}
```

**Requirements:**

- Ollama installed locally
- Sufficient RAM (8GB+ for 7B models)
- GPU recommended but optional

### LM Studio (Local Models)

**Purpose:**
User-friendly GUI for running local models.

**Features:**

- Easy model downloads
- OpenAI-compatible API
- Model comparison
- Performance tuning
- Chat interface

**Configuration:**

```typescript
{
  apiProvider: "lmstudio",
  lmStudioBaseUrl: "http://localhost:1234/v1",
  apiModelId: "local-model"
}
```

**Benefits:**

- Simpler than Ollama for beginners
- Visual model management
- Built-in benchmarking
- OpenAI API compatibility

## Request/Response Transformation

### Streaming (`transform/stream.ts`)

Handles streaming responses from providers:

```typescript
interface ApiStream {
	type: "text" | "usage" | "error" | "done"
	text?: string
	usage?: TokenUsage
	error?: Error
}

async function* streamResponse(response: Response): AsyncGenerator<ApiStream> {
	// Parse SSE or streaming JSON
	// Yield chunks as they arrive
	// Handle errors
	// Report final usage
}
```

**Benefits:**

- Real-time UI updates
- Better perceived performance
- Can interrupt long responses
- Progressive rendering

### Image Processing (`transform/image-cleaning.ts`)

Handles image data in messages:

```typescript
// Remove images when provider doesn't support vision
function maybeRemoveImageBlocks(messages: ApiMessage[], supportsVision: boolean): ApiMessage[]

// Compress images to reduce token usage
function compressImage(imageData: string, maxSize: number): string

// Convert image formats
function convertImageFormat(data: string, from: string, to: string): string
```

### Token Counting

Estimate token usage before making requests:

```typescript
function estimateTokens(text: string, model: string): number {
	// Model-specific tokenization
	// Rough estimates for speed
	// Conservative (overestimate)
}
```

## Provider-Specific Features

### Tool Use (Function Calling)

Different providers have different tool calling formats:

**Anthropic Format:**

```json
{
	"type": "tool_use",
	"id": "toolu_123",
	"name": "read_file",
	"input": {
		"path": "/path/to/file.ts"
	}
}
```

**OpenAI Format:**

```json
{
	"type": "function",
	"function": {
		"name": "read_file",
		"arguments": "{\"path\":\"/path/to/file.ts\"}"
	}
}
```

The API layer normalizes these differences.

### Vision Capabilities

Handling images varies by provider:

**Anthropic:**

- Base64 encoded images in message content
- Supports multiple images per message
- PNG, JPEG, GIF, WebP

**OpenAI:**

- URL or base64 in special format
- Single image per message (GPT-4V)
- PNG, JPEG

**Gemini:**

- Native multimodal
- Multiple images
- Video support (future)

### System Prompts

System prompt handling:

**Anthropic:**

- Native `system` parameter
- Separate from messages

**OpenAI:**

- System message in messages array
- Role: "system"

**Others:**

- Prepended to first user message
- Or not supported (ignored)

## Error Handling

### Error Types

```typescript
enum ApiErrorType {
	AUTHENTICATION = "authentication",
	RATE_LIMIT = "rate_limit",
	CONTEXT_LENGTH = "context_length",
	CONTENT_FILTER = "content_filter",
	MODEL_NOT_FOUND = "model_not_found",
	NETWORK = "network",
	TIMEOUT = "timeout",
	UNKNOWN = "unknown",
}
```

### Error Recovery

**Rate Limiting:**

- Exponential backoff
- Respect Retry-After headers
- Queue requests
- Provider rotation

**Context Length:**

- Automatic compression
- Remove old messages
- Summarization
- Retry with smaller context

**Network Errors:**

- Retry with timeout
- Exponential backoff
- Max retry count
- Fallback providers

### Error Responses

Standardized error format:

```typescript
interface ApiError {
	type: ApiErrorType
	message: string
	statusCode?: number
	retryable: boolean
	retryAfter?: number
	details?: any
}
```

## Cost Calculation

Track and estimate costs:

```typescript
interface TokenUsage {
	inputTokens: number
	outputTokens: number
	cacheCreationTokens?: number
	cacheReadTokens?: number
}

interface CostEstimate {
	inputCost: number
	outputCost: number
	totalCost: number
	currency: string
}

function calculateCost(usage: TokenUsage, model: string): CostEstimate
```

### Pricing Data

Pricing information stored per model:

```typescript
interface ModelPricing {
	inputCostPerMillion: number
	outputCostPerMillion: number
	cacheCreationCostPerMillion?: number
	cacheReadCostPerMillion?: number
}
```

Updated regularly from provider APIs.

## Model Selection

### Model Discovery

Fetch available models:

```typescript
async function getModels(provider: string, apiKey: string): Promise<ModelInfo[]> {
	// Query provider API
	// Parse response
	// Cache results
	// Return model list
}
```

### Model Metadata

```typescript
interface ModelInfo {
	id: string
	name: string
	provider: string
	contextWindow: number
	maxOutputTokens: number
	supportsVision: boolean
	supportsToolUse: boolean
	costPerMillionInputTokens: number
	costPerMillionOutputTokens: number
	description: string
}
```

### Model Recommendations

Suggest models based on use case:

- **Code Generation**: Claude 3.5 Sonnet, GPT-4
- **Chat**: Claude 3 Haiku, GPT-3.5 Turbo
- **Vision**: GPT-4V, Claude 3 Opus
- **Long Context**: Claude 3 Opus (200K)
- **Speed**: Claude 3 Haiku, GPT-3.5
- **Cost**: Local models (Ollama), Haiku

## Configuration Management

### Provider Settings

```typescript
interface ProviderSettings {
	apiProvider: ProviderName
	apiModelId: string

	// API Keys
	anthropicApiKey?: string
	openAiApiKey?: string
	openRouterApiKey?: string

	// Endpoints
	anthropicBaseUrl?: string
	openAiBaseUrl?: string

	// Advanced
	maxTokens?: number
	temperature?: number
	topP?: number
	streaming?: boolean
}
```

### Validation

Validate settings before use:

```typescript
async function validateSettings(
  settings: ProviderSettings
): Promise<ValidationResult> {
  // Check API key format
  // Test connection
  // Verify model exists
  // Check permissions

  return {
    valid: boolean,
    errors: string[],
    warnings: string[]
  }
}
```

## Performance Optimization

### Request Batching

Batch multiple requests when possible:

- Reduces API calls
- Lower latency
- Better rate limit usage

### Response Caching

Cache identical requests:

```typescript
const cacheKey = hash(systemPrompt + messages)
if (cache.has(cacheKey)) {
	return cache.get(cacheKey)
}
```

### Connection Pooling

Reuse HTTP connections:

- Keep-alive connections
- Connection limits
- Timeout management

### Compression

Compress request/response data:

- gzip compression
- Reduces bandwidth
- Faster transmission

## Testing

### Unit Tests

Test each provider independently:

- Mock HTTP responses
- Test error handling
- Verify transformations
- Check cost calculations

### Integration Tests

Test with real APIs:

- Use test API keys
- Small test prompts
- Verify streaming
- Check tool use

### Mock Providers

Mock implementations for testing:

```typescript
class MockApiHandler implements ApiHandler {
	async *createMessage() {
		yield { type: "text", text: "Mock response" }
		yield { type: "done" }
	}
}
```

## Security

### API Key Storage

- VS Code secure storage
- Never logged
- Encrypted at rest
- Cleared on logout

### Request Sanitization

- Remove sensitive data
- Validate inputs
- Escape special characters
- Limit payload size

### Response Validation

- Verify response structure
- Check for malicious content
- Validate tool calls
- Sanitize outputs

## Future Enhancements

### Planned Features

- Smart model switching based on task
- Automatic fallback chains
- Cost optimization algorithms
- Response quality monitoring
- A/B testing framework

### Provider Additions

- Azure OpenAI
- Cohere
- Hugging Face Inference API
- Together AI
- Replicate

## Troubleshooting

### Common Issues

**"Invalid API Key"**

- Verify key format
- Check expiration
- Confirm provider selected
- Test with curl

**"Rate Limited"**

- Wait and retry
- Use different provider
- Upgrade plan
- Implement queuing

**"Context Too Long"**

- Enable compression
- Reduce custom instructions
- Use smaller context model
- Summarize history

**"Model Not Found"**

- Check model ID
- Verify provider access
- Update model list
- Use default model

## Related Documentation

- [Provider Settings Guide](../../configuration/providers.md)
- [Cost Management](../../guides/cost-management.md)
- [Model Selection Guide](../../guides/model-selection.md)
- [Local Model Setup](../../guides/local-models.md)

## Contributing

When adding new providers:

1. Implement ApiHandler interface
2. Add provider-specific tests
3. Document configuration
4. Update pricing data
5. Add to buildApiHandler factory
6. Update UI model selector
