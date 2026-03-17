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
