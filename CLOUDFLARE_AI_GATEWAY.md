# Cloudflare AI Gateway Configuration

This guide explains how to configure vibesdk to use Cloudflare AI Gateway for routing requests to Claude, GPT, and Grok models.

## Overview

Cloudflare AI Gateway provides:
- Centralized API management
- Request caching
- Analytics and monitoring
- Rate limiting
- Cost tracking

The vibesdk already has built-in support for Cloudflare AI Gateway. You just need to configure the environment variables correctly.

## Environment Variables

Configure these environment variables in your Cloudflare Worker settings:

### Required Variables

```bash
# Cloudflare AI Gateway Configuration
CLOUDFLARE_AI_GATEWAY_URL=https://gateway.ai.cloudflare.com/v1/YOUR_ACCOUNT_ID/YOUR_GATEWAY_NAME/compat
CLOUDFLARE_AI_GATEWAY_TOKEN=your-gateway-bearer-token
```

### API Keys

Configure the API keys for each provider you want to use:

```bash
# OpenAI (GPT models)
OPENAI_API_KEY=sk-proj-your-actual-openai-api-key

# Anthropic (Claude models)
ANTHROPIC_API_KEY=sk-ant-api03-your-actual-anthropic-api-key

# xAI (Grok models)
XAI_API_KEY=xai-your-actual-xai-api-key

# Google AI Studio (Gemini models)
GOOGLE_AI_STUDIO_API_KEY=your-google-ai-studio-api-key
```

## How It Works

The vibesdk automatically routes requests through the Cloudflare AI Gateway when configured:

1. **Gateway URL Construction** (`worker/agents/inferutils/core.ts:189`):
   - Uses `CLOUDFLARE_AI_GATEWAY_URL` if set
   - Falls back to AI binding if not configured

2. **Authentication** (`worker/agents/inferutils/core.ts:255`):
   - Uses provider-specific API key (e.g., `OPENAI_API_KEY`)
   - Adds `cf-aig-authorization` header with gateway token

3. **Model Routing**:
   - Model names are prefixed with provider (e.g., `xai/grok-4-fast-reasoning`)
   - Gateway routes to the correct provider API

## Supported Models

### OpenAI (GPT)
```typescript
AIModels.OPENAI_5 = 'openai/gpt-5'
AIModels.OPENAI_5_MINI = 'openai/gpt-5-mini'
AIModels.OPENAI_O3 = 'openai/o3'
AIModels.OPENAI_O4_MINI = 'openai/o4-mini'
```

### Anthropic (Claude)
```typescript
AIModels.CLAUDE_4_SONNET = 'anthropic/claude-sonnet-4-20250514'
AIModels.CLAUDE_4_OPUS = 'anthropic/claude-opus-4-20250514'
AIModels.CLAUDE_3_7_SONNET_20250219 = 'anthropic/claude-3-7-sonnet-20250219'
```

### xAI (Grok)
```typescript
AIModels.GROK_4_FAST_REASONING = 'xai/grok-4-fast-reasoning'
AIModels.GROK_2_1212 = 'xai/grok-2-1212'
AIModels.GROK_2_VISION_1212 = 'xai/grok-2-vision-1212'
AIModels.GROK_BETA = 'xai/grok-beta'
```

### Google AI Studio (Gemini)
```typescript
AIModels.GEMINI_2_5_PRO = 'google-ai-studio/gemini-2.5-pro'
AIModels.GEMINI_2_5_FLASH = 'google-ai-studio/gemini-2.5-flash'
AIModels.GEMINI_2_5_FLASH_LITE = 'google-ai-studio/gemini-2.5-flash-lite'
```

## Configuration Examples

### Using Grok for Code Generation

Edit `worker/agents/inferutils/config.ts`:

```typescript
export const AGENT_CONFIG: AgentConfig = {
    phaseImplementation: {
        name: AIModels.GROK_4_FAST_REASONING,
        reasoning_effort: undefined,
        max_tokens: 64000,
        temperature: 0.2,
        fallbackModel: AIModels.GEMINI_2_5_PRO,
    },
    deepDebugger: {
        name: AIModels.GROK_4_FAST_REASONING,
        reasoning_effort: undefined,
        max_tokens: 8000,
        temperature: 0.5,
        fallbackModel: AIModels.GEMINI_2_5_PRO,
    },
};
```

### Using Claude for Conversation

```typescript
export const AGENT_CONFIG: AgentConfig = {
    conversationalResponse: {
        name: AIModels.CLAUDE_4_SONNET,
        reasoning_effort: 'medium',
        max_tokens: 4000,
        temperature: 0.7,
        fallbackModel: AIModels.GEMINI_2_5_FLASH,
    },
};
```

### Using GPT for Blueprint Generation

```typescript
export const AGENT_CONFIG: AgentConfig = {
    blueprint: {
        name: AIModels.OPENAI_5_MINI,
        reasoning_effort: 'medium',
        max_tokens: 16000,
        temperature: 1,
        fallbackModel: AIModels.GEMINI_2_5_FLASH,
    },
};
```

## Testing Your Configuration

To test if the AI Gateway is working correctly:

1. Set the environment variables
2. Deploy your worker
3. Check the Cloudflare AI Gateway dashboard for request logs
4. Monitor the console logs for:
   ```
   baseUrl: https://gateway.ai.cloudflare.com/v1/.../compat, modelName: xai/grok-4-fast-reasoning
   ```

## Troubleshooting

### Issue: Requests failing with 401 Unauthorized

**Solution**: Verify that:
- `CLOUDFLARE_AI_GATEWAY_TOKEN` is set correctly
- Provider API keys (e.g., `XAI_API_KEY`) are valid
- The gateway URL includes your account ID

### Issue: Models not routing correctly

**Solution**: Ensure model names include the provider prefix:
- Correct: `xai/grok-4-fast-reasoning`
- Incorrect: `grok-4-fast-reasoning`

### Issue: Gateway not being used

**Solution**: Check that `CLOUDFLARE_AI_GATEWAY_URL` is set and is a valid URL. If empty or 'none', the system will fall back to direct API calls.

## Advanced: Custom Gateway Endpoints

You can create multiple gateway endpoints for different use cases:

```bash
# Development gateway
CLOUDFLARE_AI_GATEWAY_URL=https://gateway.ai.cloudflare.com/v1/YOUR_ACCOUNT/dev-gateway/compat

# Production gateway with caching
CLOUDFLARE_AI_GATEWAY_URL=https://gateway.ai.cloudflare.com/v1/YOUR_ACCOUNT/prod-gateway/compat
```

## Security Notes

1. Never commit API keys to version control
2. Use Cloudflare Workers secrets for sensitive values
3. Rotate API keys regularly
4. Monitor gateway usage in Cloudflare dashboard
5. Set up rate limits in the AI Gateway dashboard

## Reference

- Code: `worker/agents/inferutils/core.ts`
- Configuration: `worker/agents/inferutils/config.ts`
- Model definitions: `worker/agents/inferutils/config.types.ts`
- Environment types: `worker-configuration.d.ts`
