# Architecture Guidelines

## Popup-Based Identity Management

Newton Identity is a **popup-based consent UI** — it's opened as a browser popup by integrating applications via the Newton SDK. The popup:

- Isolates the Identity Owner's signing context from the integrating app
- Communicates via `window.postMessage` (not iframes, not redirects)
- Auto-closes after completing or rejecting the action
- Uses `sessionStorage` for ephemeral popup parameters

## Message Flow

```
Parent App (SDK)                    Newton Identity (/handle)
    |                                      |
    |── window.open("/handle") ──────────>|
    |<── NEWTON_VC_POPUP_READY ──────────|
    |── NEWTON_VC_HANDLE_REQUEST ───────>|── route to /rpc/{method}
    |                                      |── Turnkey auth
    |                                      |── EIP-712 sign or tx
    |<── NEWTON_VC_POPUP_RESPONSE ───────|── window.close()
```

## Route Architecture (Next.js App Router)

### Popup Entry Point
- `/handle` — Message listener, routes to appropriate RPC page based on method

### RPC Pages (popup-initiated)
- `/rpc/connect` — Full flow: login → KYC → link signing
- `/rpc/link-app` — EIP-712 signature for identity linking (Owner Sig)
- `/rpc/unlink` — On-chain `unlinkIdentityAsSigner` transaction
- `/rpc/register-user-data` — Encrypt KYC data, submit to Gateway

### User Pages (standalone, not popup-initiated)
- `/user/settings` — Account settings, email, wallet address
- `/user/manage` — View/manage active identity links
- `/user/manage/unlink` — Manual unlink from manage page

## Turnkey Provider Adapter

The `useTurnkeyAuth` hook creates a custom EIP-1193 provider that bridges Turnkey's signing with viem:

- `eth_accounts` → returns Turnkey wallet addresses
- `eth_chainId` → returns current chain ID from sessionStorage
- `eth_signTypedData_v4` → hashes EIP-712 data, calls Turnkey `signRawPayload`, normalizes r/s/v
- `eth_sendTransaction` → signs via Turnkey `signTransaction`, broadcasts via Alchemy RPC
- All other methods → proxied to chain RPC

The provider is stored as a module-level singleton in `turnkeyProvider.ts` and accessed by non-React code (viem clients, signing utilities).

## On-Chain Event Sourcing

Active identity links are computed client-side, not read from contract storage:

1. Fetch `IdentityLinked` events from `EARLIEST_BLOCK` to latest
2. Fetch `IdentityUnlinked` events from same range
3. Deduplicate by `(identityDomain, policyClient, policyClientUser)` key — keep most recent
4. Active = linked events without a newer unlink event
5. Resolve block timestamps for display ("Since Mar 14, 2026")

This pattern avoids needing an off-chain database while providing complete link state.

## Gateway Communication

The `GatewayHttpService` sends JSON-RPC 2.0 requests to the Newton Gateway:

- **HTTP transport** with `X-API-Key` header
- **Method**: `newt_sendIdentityEncrypted` for identity data registration
- **Gateway URLs** resolved per chain ID from `GATEWAY_API_URLS` in `http.ts`

## Client-Side Encryption

Identity data (KYC fields) is encrypted before submission:

1. RSA-OAEP encryption using KMS public key (PEM, base64-encoded in env var)
2. Ciphertext hex-encoded for Rust operator compatibility
3. Encrypted string signed with EIP-712 (`EncryptedIdentityData` typehash)
4. Submitted to Gateway, which stores on-chain via `submitIdentity`

**Note**: Phase 2 will migrate from RSA-OAEP to HPKE (RFC 9180) per the product spec.

## Component Patterns

- **Page components** handle auth gating: show `LoadingScreen` → `LoginForm` → actual content
- **Feature components** (`ConnectSignatureScreen`, `UnlinkConfirmModal`) handle specific UI flows
- **UI primitives** (`Button`, `LabelValueRow`, `InfoBlock`) provide consistent styling
- **No state management library** — React hooks + sessionStorage/localStorage only
