# Security Guidelines

## Threat Model

Newton Identity runs as a browser popup handling sensitive identity operations. The threat model focuses on popup security, signing integrity, and data encryption.

| Concern | Risk Level | Mitigation |
|---------|------------|------------|
| Popup hijacking / spoofing | Critical | Validate postMessage origins (currently `'*'` — needs hardening) |
| EIP-712 signature replay | Critical | Nonces from contract + 5-minute deadline |
| XSS via popup parameters | High | Validate and sanitize all URL/sessionStorage inputs |
| KMS key exposure | High | Public key only in env var, never private key client-side |
| Hardcoded secrets in source | High | Move Alchemy API keys and placeholder keys to env vars |
| Session storage tampering | Medium | Validate session data before use in signing |
| Man-in-the-middle on Gateway | Medium | HTTPS only for gateway communication |

## Popup Security

### Origin Validation (MUST FIX)

Currently `resolvePopupRequest` and `rejectPopupRequest` use `'*'` as the target origin:

```typescript
// Current (insecure)
window.opener.postMessage({ ... }, '*');

// Required (restrict to known origins)
const ALLOWED_ORIGINS = ['https://app.newton.xyz', 'https://sdk.newton.xyz'];
window.opener.postMessage({ ... }, ALLOWED_ORIGINS[0]);
```

Also validate incoming messages:

```typescript
window.addEventListener('message', (event) => {
  if (!ALLOWED_ORIGINS.includes(event.origin)) return;
  handleMessage(event.data);
});
```

### Popup Parameters

- Parameters arrive via URL query string (`?request=...`) or `postMessage`
- Always `JSON.parse` inside try/catch — malformed input should not crash the popup
- Validate required fields before storing in sessionStorage
- Never trust `appName` for display without sanitization

## EIP-712 Signing Safety

### Domain Separation

The EIP-712 domain MUST match the on-chain contract exactly:

```typescript
{
  name: 'IdentityRegistry',
  version: '1',
  chainId: CHAIN_ID,
  verifyingContract: identityRegistryAddress,
}
```

Any mismatch causes silent signature verification failure on the gateway/contract.

### Replay Protection

- **Nonce**: Read from `IdentityRegistry.nonces(owner)` before each signature
- **Deadline**: Set to `Date.now() + 300000` (5 minutes)
- **Chain ID**: Included in EIP-712 domain
- **Verifying contract**: Bound to specific IdentityRegistry deployment

### Signature Normalization

The Turnkey provider adapter normalizes `r`, `s`, `v` values:
- `r` and `s` are zero-padded to 64 hex chars
- `v` is adjusted from Turnkey's raw recovery id (0/1) to Ethereum format (27/28)
- This logic in `useTurnkeyAuth.ts` lines 150-169 is critical — do not modify without testing

## Encryption Security

### Current: RSA-OAEP (Phase 1)

- KMS public key loaded from `NEXT_PUBLIC_KMS_PUBLIC_KEY` (base64-encoded PEM)
- Encryption via Web Crypto API: `RSA-OAEP` with `SHA-256`
- Ciphertext hex-encoded for Rust operator `hex::decode()` compatibility
- **Only the KMS service can decrypt** — client never has the private key

### Future: HPKE (Phase 2)

Per the product spec, encryption will migrate to HPKE (RFC 9180):
- X25519 KEM + HKDF-SHA256 + ChaCha20-Poly1305
- AAD binding to prevent cross-context replay
- Already implemented in the Newton SDK privacy module

## Secrets Management

### Never Hardcode

These are currently hardcoded and must be moved to env vars:

| Location | Secret | Fix |
|----------|--------|-----|
| `supportedChains.ts` | Alchemy API key | `NEXT_PUBLIC_ALCHEMY_API_KEY` env var |
| `useTurnkeyAuth.ts:193` | Same Alchemy API key | Use `supportedChains.ts` helper |
| `utils/encrypt.ts` | KMS public key PEM | Use `NEXT_PUBLIC_KMS_PUBLIC_KEY` (already exists in `lib/kms.ts`) |
| `register-user-data/page.tsx` | `admin_key_CHANGE_ME_IN_PRODUCTION` | Require API key from session |
| `ConnectKycCard.tsx` | `admin_key_CHANGE_ME_IN_PRODUCTION` | Require API key from session |

### Session Storage Sensitivity

| Key | Sensitivity | Notes |
|-----|-------------|-------|
| `chainId` | Low | Public chain identifier |
| `apiKey` | Medium | Newton API key — clear on popup close |
| `dappUserAddress` | Low | Public address |
| `dappClientAddress` | Low | Public address |
| `userData` | High | KYC data — encrypt or clear immediately after use |
| `appIdentityDomain` | Low | Public domain identifier |

## Input Validation

### Address Validation

Use viem's type system:

```typescript
import { isAddress, getAddress } from 'viem';

if (!isAddress(address)) {
  throw new Error('Invalid address');
}
const checksummed = getAddress(address);
```

### Chain ID Validation

Reject unsupported chains early:

```typescript
const SUPPORTED_CHAINS = [11155111] as const; // Currently Sepolia only

if (!SUPPORTED_CHAINS.includes(chainId)) {
  throw new Error(`Chain ${chainId} not supported`);
}
```

## Error Handling

- Never swallow errors silently in signing or encryption flows
- Always show user-facing error messages for failed operations
- Log errors to console for debugging but never log sensitive data (signatures, encrypted payloads, PII)
- Return structured errors to parent via `rejectPopupRequest()` with error codes
