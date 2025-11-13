# Deployment Fix: Cloudflare Containers Issue

## Problem

The deployment was failing with a 403 Forbidden error:

```
ApiError: Forbidden
  at catchErrorCodes
  url: 'https://api.cloudflare.com/client/v4/accounts/1a481f7cdb7027c30174a692c89cbda1/containers/me',
  status: 403,
  body: { error: 'Authentication error' }
```

## Root Cause

The `wrangler.jsonc` configuration included a **Cloudflare Containers** section that requires:
- Special API permissions for Containers
- Beta/preview access to Cloudflare Containers feature
- Elevated API token permissions

Not all Cloudflare accounts have access to Containers, causing the 403 error.

## Solution

Disabled Cloudflare Containers and configured the project to use an **external sandbox service** instead.

### Changes Made

**1. Commented out Containers configuration** (`wrangler.jsonc`):
- Disabled the `containers` section (lines 58-77)
- Removed `UserAppSandboxService` from Durable Objects binding
- Removed `UserAppSandboxService` from migrations

**2. Made Sandbox binding optional** (`worker-configuration.d.ts`):
- Changed `Sandbox` binding type to optional (`Sandbox?`)

**3. Using External Sandbox Service**:
The project already has built-in support for external sandbox services via the factory pattern in `worker/services/sandbox/factory.ts`:

```typescript
if (env.SANDBOX_SERVICE_TYPE == 'runner') {
    return new RemoteSandboxServiceClient(sessionId);
}
```

## Configuration Required

Ensure these environment variables are set in your Cloudflare Workers settings:

```bash
# Sandbox Service Configuration
SANDBOX_SERVICE_TYPE=runner
SANDBOX_SERVICE_URL=your-sandbox-service-url
SANDBOX_SERVICE_API_KEY=your-sandbox-service-api-key
```

### Where to Set These Variables

**Option 1: Cloudflare Dashboard**
1. Go to Workers & Pages
2. Select your worker
3. Go to Settings > Variables and Secrets
4. Add the variables above

**Option 2: Deploy Script** (`.prod.vars` file)
```bash
SANDBOX_SERVICE_TYPE=runner
SANDBOX_SERVICE_URL=https://your-sandbox-service.com
SANDBOX_SERVICE_API_KEY=your-api-key
```

**Option 3: wrangler.jsonc vars section**
Add to the `vars` section:
```jsonc
"vars": {
    "SANDBOX_SERVICE_TYPE": "runner",
    // Other vars...
}
```

## Verification

After making these changes, the deployment should succeed without the 403 error. The worker will use the external sandbox service instead of attempting to create Cloudflare Containers.

## Re-enabling Containers (Future)

If you later gain access to Cloudflare Containers:

1. Uncomment the containers section in `wrangler.jsonc`
2. Uncomment the UserAppSandboxService bindings
3. Uncomment UserAppSandboxService in migrations
4. Change `Sandbox?` back to `Sandbox` in `worker-configuration.d.ts`
5. Ensure your API token has containers permissions
6. Set `SANDBOX_SERVICE_TYPE` to something other than 'runner'

## Additional Resources

- [Cloudflare Workers Containers Docs](https://developers.cloudflare.com/workers/runtime-apis/bindings/service-bindings/)
- [vibesdk Sandbox Architecture](./docs/architecture-diagrams.md)
- Project sandbox factory: `worker/services/sandbox/factory.ts`
