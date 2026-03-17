# Newton Identity - AI Agent Guidelines

## Project Overview

Newton Identity is a popup-based identity management UI for the Newton Verifiable Credential (VC) system. It is opened as a browser popup by integrating applications via the Newton SDK, allowing identity owners to manage their identity data linking, unlinking, and registration using their Identity Owner EOA (Owner EOA) through Turnkey's passkey-based wallet infrastructure.

This repo implements the **User Consent Signature** flow from the Newton VC Product Spec — the popup where users authenticate and sign consent for identity operations.

## Tech Stack

| Component       | Technology                                |
|-----------------|-------------------------------------------|
| Framework       | Next.js 16 (App Router)                   |
| UI              | React 19, Tailwind CSS v4                 |
| Language        | TypeScript 5 (strict mode)                |
| Auth            | Turnkey (react-wallet-kit, passkey login) |
| Blockchain      | viem 2.39 (contract reads, tx, EIP-712)   |
| SDK             | @magicnewton/newton-protocol-sdk          |
| Encryption      | Web Crypto API (RSA-OAEP with KMS key)    |
| Formatter       | Prettier                                  |
| Package Manager | pnpm                                      |

## Project Structure

```
newton-identity/
├── src/
│   ├── app/
│   │   ├── layout.tsx              # Root layout with TurnkeyProvider
│   │   ├── providers.tsx           # Turnkey configuration
│   │   ├── api/kyc/route.ts        # Server-side Persona KYC proxy
│   │   └── (pages)/
│   │       ├── handle/page.tsx     # Popup message router (entry point)
│   │       ├── rpc/
│   │       │   ├── connect/        # Identity Owner login + KYC + link
│   │       │   ├── link-app/       # EIP-712 signing for identity linking
│   │       │   ├── unlink/         # On-chain unlink transaction
│   │       │   └── register-user-data/ # Encrypted KYC data submission
│   │       └── user/
│   │           ├── settings/       # Account settings
│   │           ├── manage/         # View/manage linked apps
│   │           │   └── unlink/     # Manual unlink from manage page
│   │           └── verified-data/  # View verified data (disabled)
│   ├── lib/
│   │   ├── popup.ts               # Message types & method enums
│   │   ├── identityRegistry.ts    # EIP-712 domain, types, contract config
│   │   ├── turnkeyProvider.ts     # EIP-1193 provider adapter for Turnkey
│   │   ├── kms.ts                 # RSA-OAEP encryption with KMS public key
│   │   ├── unlinkIdentity.ts      # On-chain unlinkIdentityAsSigner tx
│   │   ├── activeLinksHydration.ts # Fetch on-chain link/unlink events
│   │   └── identityLinkedLogs.ts  # Log deduplication & active link computation
│   ├── hooks/
│   │   ├── useTurnkeyAuth.ts      # Auth hook: login, logout, signing, provider
│   │   └── useKYCStatus.ts        # KYC completion tracking
│   ├── utils/
│   │   ├── linkSignature.ts       # EIP-712 link signing
│   │   ├── resolvePopUp.ts        # postMessage response helpers
│   │   ├── http.ts                # Gateway JSON-RPC client
│   │   └── supportedChains.ts     # Chain configs
│   ├── components/                # UI components
│   ├── abi/                       # IdentityRegistry ABI
│   ├── constants/                 # Contract addresses
│   └── types/                     # TypeScript types
├── docs/                          # Project documentation
├── package.json
└── next.config.ts
```

## Essential Commands

| Command        | Purpose                                   |
|----------------|-------------------------------------------|
| `pnpm install` | Install dependencies                      |
| `pnpm dev`     | Start Next.js dev server (localhost:3000) |
| `pnpm build`   | Build for production                      |
| `pnpm start`   | Start production server                   |

## Core Architecture

### Popup-Based RPC Protocol

The app operates as a browser popup opened by integrating applications:

1. Parent app opens `/handle` as popup via `window.open()`
2. Popup sends `NEWTON_VC_POPUP_READY` to parent
3. Parent sends `NEWTON_VC_HANDLE_REQUEST` with method + params
4. Popup routes to `/rpc/{method}`, user authenticates and signs
5. Popup sends `NEWTON_VC_POPUP_RESPONSE` with result back to parent
6. Popup auto-closes after 500ms

### Popup RPC Methods

| Method                              | Route                     | Purpose                             |
|-------------------------------------|---------------------------|-------------------------------------|
| `newton_vc_user_connect`            | `/rpc/connect`            | Login + optional KYC + link signing |
| `newton_vc_user_link_app`           | `/rpc/link-app`           | EIP-712 identity link signing       |
| `newton_vc_user_unlink`             | `/rpc/unlink`             | On-chain unlink transaction         |
| `newton_vc_user_register_user_data` | `/rpc/register-user-data` | Encrypt + submit KYC data           |

### EIP-712 Signing

Two typed data structures:

**Link Identity Signer** (for linking):
```
Domain: { name: "IdentityRegistry", version: "1", chainId, verifyingContract }
Type: linkIdentitySigner(address identityOwner, address policyClient, address clientUser, bytes32[] identityDomains, uint256 identityOwnerNonce, uint256 deadline)
```

**Encrypted Identity Data** (for data registration):
```
Domain: { name: "IdentityRegistry", version: "1", chainId, verifyingContract }
Type: EncryptedIdentityData(string data)
```

### Identity Registry Contract

The popup interacts with `IdentityRegistry` for:
- **Read**: `nonces(address)` — replay protection for EIP-712 signatures
- **Write**: `unlinkIdentityAsSigner(clientUser, policyClient, identityDomains[])` — revoke links
- **Events**: `IdentityLinked`, `IdentityUnlinked` — compute active link state

### Turnkey Authentication

Users authenticate via Turnkey passkeys (WebAuthn). The `useTurnkeyAuth` hook:
- Manages login/logout via Turnkey SDK
- Auto-creates an Ethereum wallet on first login
- Creates an EIP-1193 provider adapter bridging Turnkey signing with viem
- Handles `eth_signTypedData_v4` by hashing EIP-712 data and calling `signRawPayload`

## Environment Variables

| Variable                                   | Required | Description                       |
|--------------------------------------------|----------|-----------------------------------|
| `NEXT_PUBLIC_TURNKEY_ORGANIZATION_ID`      | Yes      | Turnkey organization ID           |
| `NEXT_PUBLIC_TURNKEY_AUTH_PROXY_CONFIG_ID` | Yes      | Turnkey Auth Proxy config         |
| `NEXT_PUBLIC_IDENTITY_REGISTRY_ADDRESS`    | Yes      | IdentityRegistry contract address |
| `NEXT_PUBLIC_KMS_PUBLIC_KEY`               | Yes      | Base64-encoded PEM RSA public key |
| `NEXT_PUBLIC_GATEWAY_API_URL`              | No       | Gateway RPC URL override          |

## Key Principles

1. **Never modify EIP-712 types without contract coordination** — types must match `IdentityRegistry.sol` exactly or signatures fail silently
2. **Never hardcode secrets** — API keys, RPC URLs, and encryption keys must come from environment variables
3. **Never trust postMessage without origin validation** — validate `event.origin` (currently uses `'*'`, needs hardening)
4. **Always validate chain IDs** — reject unsupported chains early
5. **Never log sensitive data** — no private keys, signatures, or PII in console output
6. **Preserve Turnkey provider adapter r/s/v normalization** — the signature encoding in `useTurnkeyAuth.ts` lines 150-169 is critical
7. **Use `sessionStorage` not `localStorage` for popup params** — popup data is ephemeral, scoped to tab

## Common Pitfalls

| Symptom                                 | Cause                               | Fix                                                                               |
|-----------------------------------------|-------------------------------------|-----------------------------------------------------------------------------------|
| Signature verification fails on gateway | EIP-712 domain or types mismatch    | Verify `identityRegistry.ts` matches contract's `EIP712("IdentityRegistry", "1")` |
| `Turnkey signing not ready` error       | Provider not set up before signing  | Ensure `useTurnkeyAuth` hook completes before accessing `getTurnkeyProvider()`    |
| Active links not showing                | Event fetch starts from wrong block | Check `EARLIEST_BLOCK` in `activeLinksHydration.ts`                               |
| Popup closes without response           | `window.opener` is null             | Popup must be opened via `window.open()`, not navigation                          |
| Encryption fails                        | Missing or invalid KMS public key   | Verify `NEXT_PUBLIC_KMS_PUBLIC_KEY` is valid base64-encoded PEM                   |
| Chain ID mismatch                       | sessionStorage not set by /handle   | Ensure `/handle` page stores chainId before routing                               |

## Newton VC Domain Context

### Entity Model

- **Identity Owner EOA (Owner EOA)**: User's global identity authority wallet (managed via Turnkey)
- **Application EOA (App EOA)**: Per-app embedded wallet (e.g., Echo/Polymarket)
- **Policy Client**: Developer/platform entity that uses identity data
- **Identity Domain**: bytes32 namespace partitioning identity data contexts
- **Client User**: The App EOA address being linked

### Dual-Signature Flow

Identity linking requires two signatures:
1. **User Sig** (this repo) — Identity Owner signs EIP-712 typed data in the popup
2. **Client Sig** (developer backend) — Policy Client Owner signs approval

The Newton SDK combines both signatures to call `IdentityRegistry.linkIdentity()` on-chain.

### Privacy Roadmap

- **Phase 1 (current)**: Explicit address-based identity references
- **Phase 2**: DID-shaped identifiers (reduce global linkage leakage)
- **Phase 3**: Offchain DID resolution for signature verification

## Modular Rules

Detailed guidelines are in `.claude/rules/`:

| File                     | Contents                                                      |
|--------------------------|---------------------------------------------------------------|
| `architecture.md`        | Popup protocol, Turnkey patterns, event sourcing, Gateway RPC |
| `code-style.md`          | TypeScript/React/Next.js conventions, Prettier, Tailwind      |
| `security.md`            | Popup security, postMessage, KMS encryption, EIP-712          |
| `communication-style.md` | PR review tone, comment conventions, AI identity rule         |
| `git-hygiene.md`         | AI attribution ban, conventional commits                      |
| `agent-guide.md`         | Verify-before-claiming, quality bar, mid-task replanning      |
| `ai-workflow.md`         | Explore→Plan→Code→Commit, TDD, extended thinking              |
| `lessons.md`             | Known issues and prevention rules (living doc)                |

## Related Repositories

- [`newton-sdk`](https://github.com/newt-foundation/newton-sdk) — Client SDK that opens this popup and handles responses
- [`newton-prover-avs`](https://github.com/newt-foundation/newton-prover-avs) — AVS backend with IdentityRegistry contract, Gateway RPC, and operator infrastructure
- [Newton VC Product Spec](https://www.notion.so/magiclabs/Newton-Verifiable-Cred-VC-Product-Specification-v0-2e9ad65b2b7080faa8eac75655aaa6b8) — Full product specification
- [Linear: Newton VC Project](https://linear.app/magiclabs/project/newton-verifiable-credential-ed6073c1c056) — Issue tracking
