# Lessons Learned

Living document. When a mistake recurs or a non-obvious pattern causes confusion, add an entry.

## Format

Each entry:
- **What happened**: Brief description
- **Why**: Root cause
- **Rule**: What to do differently

---

## Entries

### Hardcoded Alchemy API key in source

- **What happened**: Alchemy API key `lnkxjj_rKFZRHqrrIvoerSJtzIdt2ElK` is hardcoded in `supportedChains.ts` and `useTurnkeyAuth.ts`
- **Why**: Quick prototype without env var setup
- **Rule**: All RPC URLs must use environment variables. Never commit API keys to source.

### Duplicate encryption modules

- **What happened**: Two encryption implementations exist: `lib/kms.ts` (env var key, hex output) and `utils/encrypt.ts` (hardcoded key, base64 output)
- **Why**: Different iterations of the encryption approach were not consolidated
- **Rule**: Single source of truth for encryption. Use `lib/kms.ts` (env var-based). Remove `utils/encrypt.ts` or redirect to `lib/kms.ts`.

### Duplicate IdentityData type

- **What happened**: `IdentityData` is defined in both `lib/identityRegistry.ts` and `types/identity-data.ts` with identical fields
- **Why**: Type was added in two places independently
- **Rule**: One canonical type definition. Import from `types/identity-data.ts` everywhere.

### Broad postMessage target origin

- **What happened**: `resolvePopupRequest` and `rejectPopupRequest` use `'*'` as target origin
- **Why**: Development convenience — no origin restriction
- **Rule**: Always specify allowed origins for postMessage. Use an allowlist of known Newton domains.

### EIP-712 domain must match contract exactly

- **What happened**: Signature verification fails silently if any EIP-712 domain field doesn't match the contract's constructor parameters
- **Why**: No runtime validation that domain matches — the signature just doesn't verify
- **Rule**: Never change `identityRegistry.ts` domain/types without verifying against the deployed contract. Test with `recoverAddress` locally before assuming correctness.

### Turnkey v normalization

- **What happened**: Signatures failed because Turnkey returns raw recovery id (0 or 1) but ecrecover expects v = 27 or 28
- **Why**: Different conventions between Turnkey's signing API and Ethereum's signature format
- **Rule**: Always normalize v: `vEthereum = vNum === 0 || vNum === 1 ? vNum + 27 : vNum`. This logic in `useTurnkeyAuth.ts` is critical.

### Only Sepolia supported despite multi-chain config

- **What happened**: `supportedChains.ts` only handles chain 11155111 (Sepolia) but `http.ts` defines gateway URLs for Base Sepolia, Base mainnet, and Ethereum mainnet
- **Why**: Chain support was added to the gateway service but not propagated to the frontend chain config
- **Rule**: When adding chain support, update both `supportedChains.ts` (chain objects, RPC URLs) and `http.ts` (gateway URLs).

### JSON-RPC id field not set by client

- **What happened**: During code review, the missing `id` field in `jsonRpc.ts` was flagged as a spec violation
- **Why**: The Newton gateway populates the JSON-RPC `id` server-side — clients should not set it
- **Rule**: Do not add an `id` field to JSON-RPC payloads in `createJsonRpcRequestPayload`. The gateway assigns it.

### Local `pnpm build` fails but Vercel deploys succeed

- **What happened**: `pnpm build` fails locally on `/_global-error` prerender with `TypeError: Cannot read properties of null (reading 'useContext')`. All Vercel deployments succeed with state READY.
- **Why**: Next.js 16 Turbopack (local) tries to fully execute `'use client'` components during SSG prerendering. `TurnkeyProvider` calls `useContext` which crashes because React's dispatcher isn't initialized during static generation. Vercel's `@vercel/next` builder patches this by rendering client components as empty shells during SSG, so it never hits the crash.
- **Rule**: Do not treat the local `pnpm build` failure as a blocker — Vercel deployments are the source of truth. `pnpm dev` works locally. If local production builds are ever needed (e.g., Docker), each page must be restructured as a server component wrapper with `next/dynamic` client import — but this is not needed for Vercel hosting.

### `force-dynamic` on root layout breaks `/_global-error` prerender

- **What happened**: Adding `export const dynamic = 'force-dynamic'` to `layout.tsx` or adding a `(pages)/layout.tsx` with `force-dynamic` causes `/_global-error` to fail prerendering, even though it fixes all other pages.
- **Why**: `/_global-error` is a special internal Next.js route that is always statically prerendered regardless of `force-dynamic`. Changes to the layout route tree affect how Next.js compiles the error boundary page.
- **Rule**: Do not use `force-dynamic` on the root layout or route group layouts as a fix for the prerender issue. It creates a whack-a-mole between `/_global-error` and the regular pages.
