# Quick Start: Cloudflare AI Gateway Setup

This guide will help you configure vibesdk to use Cloudflare AI Gateway with Claude, GPT, and Grok models.

## Prerequisites

1. Cloudflare account
2. API keys from providers you want to use (OpenAI, Anthropic, xAI)
3. vibesdk project

## Step 1: Create Cloudflare AI Gateway

1. Go to [Cloudflare Dashboard](https://dash.cloudflare.com)
2. Navigate to **AI** > **AI Gateway**
3. Click **Create Gateway**
4. Name it (e.g., `voither` or `vibesdk-gateway`)
5. Note your **Account ID** and **Gateway Name**

## Step 2: Get Your Credentials

Your AI Gateway URL will be in this format:
```
https://gateway.ai.cloudflare.com/v1/{account_id}/{gateway_name}/compat
```

Example:
```
https://gateway.ai.cloudflare.com/v1/abc123def456/my-gateway/compat
```

You'll also need:
- **Gateway Token**: Found in the AI Gateway settings (Bearer token for `cf-aig-authorization` header)
- **Provider API Keys**: From each LLM provider you want to use

## Step 3: Configure Environment Variables

Copy `.env.example` to `.env` (or configure in Cloudflare Workers dashboard):

```bash
cp .env.example .env
```

Edit `.env` with your actual values:

```bash
# Required: AI Gateway Configuration
CLOUDFLARE_AI_GATEWAY_URL=https://gateway.ai.cloudflare.com/v1/YOUR_ACCOUNT_ID/YOUR_GATEWAY_NAME/compat
CLOUDFLARE_AI_GATEWAY_TOKEN=your-gateway-bearer-token

# Required: At least one provider API key
OPENAI_API_KEY=sk-proj-your-openai-api-key
ANTHROPIC_API_KEY=sk-ant-api03-your-anthropic-api-key
XAI_API_KEY=xai-your-xai-api-key
GOOGLE_AI_STUDIO_API_KEY=your-google-ai-studio-api-key
```

## Step 4: Choose Your Models

Edit `worker/agents/inferutils/config.ts` to configure which models to use for each operation:

```typescript
export const AGENT_CONFIG: AgentConfig = {
    // Use Grok for fast code generation
    phaseImplementation: {
        name: AIModels.GROK_4_FAST_REASONING,
        max_tokens: 64000,
        temperature: 0.2,
        fallbackModel: AIModels.GEMINI_2_5_PRO,
    },

    // Use Claude for conversations
    conversationalResponse: {
        name: AIModels.CLAUDE_4_SONNET,
        reasoning_effort: 'medium',
        max_tokens: 4000,
        temperature: 0.7,
        fallbackModel: AIModels.GEMINI_2_5_FLASH,
    },

    // Use GPT for blueprints
    blueprint: {
        name: AIModels.OPENAI_5_MINI,
        reasoning_effort: 'medium',
        max_tokens: 16000,
        temperature: 1,
        fallbackModel: AIModels.GEMINI_2_5_FLASH,
    },
};
```

## Step 5: Deploy

Deploy to Cloudflare Workers:

```bash
npm run deploy
```

Or for development:

```bash
npm run dev
```

## Step 6: Verify Configuration

Check that requests are going through the gateway:

1. Open Cloudflare AI Gateway dashboard
2. You should see requests appearing in the analytics
3. Check worker logs for confirmation:
   ```
   baseUrl: https://gateway.ai.cloudflare.com/v1/.../compat
   modelName: xai/grok-4-fast-reasoning
   ```

## Available Models

### OpenAI (GPT)
- `AIModels.OPENAI_5` - GPT-5
- `AIModels.OPENAI_5_MINI` - GPT-5-mini (faster, cheaper)
- `AIModels.OPENAI_O3` - O3 (reasoning)
- `AIModels.OPENAI_O4_MINI` - O4-mini

### Anthropic (Claude)
- `AIModels.CLAUDE_4_SONNET` - Claude 4 Sonnet
- `AIModels.CLAUDE_4_OPUS` - Claude 4 Opus (most capable)
- `AIModels.CLAUDE_3_7_SONNET_20250219` - Claude 3.7 Sonnet

### xAI (Grok)
- `AIModels.GROK_4_FAST_REASONING` - Grok 4 (fast reasoning, recommended)
- `AIModels.GROK_2_1212` - Grok 2
- `AIModels.GROK_2_VISION_1212` - Grok 2 Vision (supports images)
- `AIModels.GROK_BETA` - Grok Beta

### Google AI Studio (Gemini)
- `AIModels.GEMINI_2_5_PRO` - Gemini 2.5 Pro
- `AIModels.GEMINI_2_5_FLASH` - Gemini 2.5 Flash (fast, good fallback)
- `AIModels.GEMINI_2_5_FLASH_LITE` - Gemini 2.5 Flash Lite (fastest)

## Recommended Configurations

### Cost-Optimized (Primarily Gemini with selective premium models)
```typescript
{
    blueprint: AIModels.GEMINI_2_5_PRO,
    phaseGeneration: AIModels.GEMINI_2_5_PRO,
    phaseImplementation: AIModels.GEMINI_2_5_PRO,
    conversationalResponse: AIModels.GEMINI_2_5_FLASH,
    deepDebugger: AIModels.GEMINI_2_5_PRO,
}
```

### Performance-Optimized (Mix of fast models)
```typescript
{
    blueprint: AIModels.OPENAI_5_MINI,
    phaseGeneration: AIModels.OPENAI_5_MINI,
    phaseImplementation: AIModels.GROK_4_FAST_REASONING,
    conversationalResponse: AIModels.CLAUDE_4_SONNET,
    deepDebugger: AIModels.GROK_4_FAST_REASONING,
}
```

### Quality-Optimized (Best models for each task)
```typescript
{
    blueprint: AIModels.OPENAI_5,
    phaseGeneration: AIModels.CLAUDE_4_SONNET,
    phaseImplementation: AIModels.CLAUDE_4_OPUS,
    conversationalResponse: AIModels.CLAUDE_4_SONNET,
    deepDebugger: AIModels.OPENAI_5,
}
```

## Troubleshooting

### Error: 401 Unauthorized
- Check that `CLOUDFLARE_AI_GATEWAY_TOKEN` is correct
- Verify provider API keys are valid
- Ensure gateway URL includes your account ID

### Error: Model not found
- Verify model name includes provider prefix (e.g., `xai/grok-4-fast-reasoning`)
- Check that the provider API key is configured
- Ensure the model is available in your region

### Requests not going through gateway
- Confirm `CLOUDFLARE_AI_GATEWAY_URL` is set
- Check that it's not set to 'none' or empty string
- Verify URL format is correct

### High latency
- Use Flash/Mini models for faster responses
- Enable caching in AI Gateway settings
- Consider using regional endpoints

## Next Steps

1. Monitor usage in Cloudflare AI Gateway dashboard
2. Set up rate limits and caching rules
3. Configure alerts for errors and high usage
4. Optimize model selection based on cost and performance metrics

## Additional Resources

- [Detailed Configuration Guide](./CLOUDFLARE_AI_GATEWAY.md)
- [Environment Variables Reference](./.env.example)
- [Model Configuration](./worker/agents/inferutils/config.ts)
- [Cloudflare AI Gateway Docs](https://developers.cloudflare.com/ai-gateway/)

## Support

For issues or questions:
1. Check the troubleshooting section above
2. Review the detailed configuration guide
3. Check Cloudflare AI Gateway logs
4. Open an issue in the repository
