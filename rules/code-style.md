# Code Style Guidelines

## Benchmark

Follow patterns established by [viem](https://github.com/wevm/viem) for blockchain code and [Next.js App Router](https://nextjs.org/docs/app) for framework patterns.

## Formatting: Prettier

Configured in `.prettierrc`:
- Semicolons: yes
- Single quotes: yes
- Tab width: 2
- Trailing commas: ES5

## Naming Conventions

| Type | Convention | Example |
|------|------------|---------|
| Functions | camelCase | `signLink`, `fetchActiveLinks` |
| Variables | camelCase | `dappUserAddress`, `identityOwner` |
| Constants | SCREAMING_SNAKE_CASE | `GATEWAY_API_URLS`, `EARLIEST_BLOCK` |
| Types/Interfaces | PascalCase | `IdentityData`, `LinkedUnlinkedLog` |
| Enums | PascalCase | `NewtonVcPayloadMethod` |
| Enum values | PascalCase | `NewtonVcPayloadMethod.LinkApp` |
| Components | PascalCase | `ConnectSignatureScreen` |
| Hooks | camelCase with `use` prefix | `useTurnkeyAuth` |
| Files (utils/lib) | camelCase | `linkSignature.ts`, `resolvePopUp.ts` |
| Files (components) | PascalCase | `ConnectPrimerScreen.tsx` |
| Files (pages) | kebab-case | `page.tsx` (Next.js convention) |

## TypeScript Patterns

### Strict Mode

All code runs with `strict: true`. Minimize `any` — use `unknown` + type guards.

### React Patterns

```typescript
// Good: 'use client' directive for client components
'use client';

// Good: Hooks at top level, not inside conditions
const { isLoading, isLoggedIn, userInfo } = useTurnkeyAuth();

// Good: Early returns for loading/auth states
if (isLoading) return <LoadingScreen />;
if (!isLoggedIn) return <LoginForm onLogin={login} />;
```

### Path Aliases

Use the configured `@/` alias for `src/`:

```typescript
// Good
import { getIdentityRegistryAddress } from '@/lib/identityRegistry';
import { useTurnkeyAuth } from '@/hooks';

// Bad
import { getIdentityRegistryAddress } from '../../../../lib/identityRegistry';
```

### Hex Types

Use viem's `0x${string}` type for Ethereum addresses and hashes:

```typescript
// Good
const owner = userInfo.wallets.ethereum.publicAddress as `0x${string}`;

// Good: Type-safe contract address
function getIdentityRegistryAddress(): `0x${string}` { ... }
```

### Session Storage Pattern

Popup parameters flow through sessionStorage:

```typescript
// Set in /handle router
sessionStorage.setItem('dappUserAddress', appWalletAddress);
sessionStorage.setItem('chainId', chainId);

// Read in RPC pages
const chainId = Number(sessionStorage.getItem('chainId')) || 11155111;
```

## Tailwind CSS v4

- Use utility classes directly, no custom CSS unless necessary
- Responsive: mobile-first (`md:` for desktop overrides)
- Max width containers: `max-w-[400px]` or `max-w-[445px]` for popup content
- Consistent spacing: `p-4`, `p-5`, `p-8` for page padding

## Imports

Group in order:
1. React/Next.js (`react`, `next/navigation`, `next/link`)
2. External packages (`viem`, `@turnkey/*`)
3. Internal modules (`@/lib/...`, `@/utils/...`, `@/hooks/...`)
4. Components (`@/components/...`)
5. Types

## Comments

- No emojis in comments
- Comment **why**, not **what**
- No commented-out code — delete it
- No AI-generated context comments
- Remove debug `console.log` before committing (currently in `linkSignature.ts`, `activeLinksHydration.ts`, `ConnectKycCard.tsx`, `useKYCStatus.ts`)

## Anti-Patterns

| Pattern | Problem | Solution |
|---------|---------|----------|
| `any` type | Loss of type safety | Use `unknown` + type guards |
| Hardcoded RPC URLs | Breaks in different environments | Use env vars |
| Hardcoded API keys | Security risk | Pass at runtime via sessionStorage |
| `console.log` in prod | Leaks to user's console | Remove or use debug flag |
| `window.opener.postMessage('*')` | No origin validation | Restrict target origin |
| Duplicate type definitions | Maintenance burden | Single source of truth |
