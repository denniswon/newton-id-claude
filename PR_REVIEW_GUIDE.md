# PR Review Workflow

## Quick Start

```bash
# Review current branch changes
/review-pr

# Re-review after author addresses feedback
/re-review-pr https://github.com/newt-foundation/newton-identity/pull/123
```

## What Gets Reviewed

Blockers — EIP-712 signing correctness, popup security (postMessage origin), Turnkey provider adapter changes, leaked secrets, XSS vectors, encryption correctness, broken popup flow.

Concerns — error handling (missing catches, swallowed errors), missing auth gating on pages, session storage misuse, unnecessary re-renders, chain ID validation gaps.

Minor notes — naming clarity, component organization, Tailwind class ordering.

## Comment Style Guidelines

### General Principles

- Be **direct and concise** - state the issue immediately without preamble
- No bold category prefixes like "**Critical:**" or "**Warning:**"
- No formal headers like "Suggested fix:" - just provide the fix
- No markdown headings in the review body — use **bold** sparingly
- Reference existing patterns in the codebase when applicable
- Use conversational directives: "Let's not do this way" or "something like below can work"
- Use "ditto:" for repeated issues in different files
- Use "nit:" for minor/cosmetic issues
- For follow-up reviews: reference previous comments casually ("still not addressed from previous review")
- Keep the review body focused on what's wrong, not what's been fixed
- Don't comment on things that are correctly done
- Bold direct asks to the author: **Document this in the PR description.**
- Escalate clearly when blocking: "A blocker for this PR considering [reason]."

### Voice and Framing

- **Use "we/us" not "you"** — collaborative framing. "Let's do X", "if we update", "we should"
- **Direct imperatives for clear issues** — "DO NOT", "Shouldn't do this", "Let's do X"
- **Prefix labels for non-blocking items:**
  - `Opinion:` — subjective feedback, architectural preferences
  - `FYI:` — informational, good to know
  - `Suggestion:` — code improvement with example
  - `nit:` — minor/cosmetic
- **No praise sections** — review body is for issues only
- **No severity grouping** — flat numbered list with prefix labels
- **Short punchy comments preferred**
- **End with clear merge criteria** — "LGTM once above are addressed."

### Formatting Rules — DO NOT USE:

- Emoji headers or category prefixes
- Bold-label patterns like "**Problem:**", "**Risk:**", "**Fix:**"
- Markdown headings in the review body
- Severity grouping headers
- Long code blocks in the review body (code suggestions go in inline comments)
- "What is working well" or praise sections
- Using "you" — use "we/us/let's"

### React/Next.js-Specific Concerns

When reviewing newton-identity PRs, additionally check:

- **Popup protocol**: Does the change break the postMessage flow? Are new methods handled in `/handle`?
- **Auth gating**: Every page must gate on `useTurnkeyAuth` (loading → login → content)
- **EIP-712 correctness**: Do types match the IdentityRegistry contract exactly?
- **Session storage**: Are popup params properly stored/read? Cleared when appropriate?
- **Turnkey provider**: Any changes to the signing adapter must preserve r/s/v normalization
- **Encryption**: KMS key from env var, not hardcoded? Output format correct for gateway?
- **Client components**: `'use client'` directive present where needed?
- **Event sourcing**: Active link computation still correct after changes?

### Inline Comment Format

Good:

```text
This hardcodes a testnet URL. Shouldn't do this — we have `getSupportedChainById` for this.
```

```text
The EIP-712 type here doesn't match what the contract expects. Let's verify against the deployed IdentityRegistry.
```

### Code Suggestions

Only include code examples when the fix is non-obvious. Introduce casually: "something like below can work."

### Cross-References

Always use GitHub permalinks with commit SHA and line range:

- "We already do this in https://github.com/newt-foundation/newton-identity/blob/abc123/src/lib/identityRegistry.ts#L17-L24"
