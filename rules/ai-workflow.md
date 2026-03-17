# AI Workflow Guidelines

## Workflow Pattern: Explore, Plan, Code, Commit

For complex tasks, follow this sequence:

1. **Explore**: Understand the codebase and requirements
   - "Read the popup message flow in handle/page.tsx"
   - "Search for all usages of signLink"

2. **Plan**: Create a clear implementation plan
   - "Create a plan for adding HPKE encryption support"
   - Outline steps before coding

3. **Code**: Implement the solution
   - Implement one component at a time
   - Test in the popup flow after each change

4. **Commit**: Create atomic commits
   - Use conventional commit format: `feat:`, `fix:`, `refactor:`, etc.
   - No AI attribution

## Test-Driven Development

When adding new flows or fixing bugs:

```
1. "Write a test for the new unlink-as-user flow"
2. Run test, confirm failure
3. "Implement the code to make the test pass"
4. Run test, confirm pass
5. "Refactor for clarity while keeping tests green"
```

## Extended Thinking

Claude allocates more reasoning effort based on keywords:

| Keyword | When to Use |
|---------|-------------|
| "think" | Simple component changes, styling fixes |
| "think hard" | Multi-file changes, new popup flows |
| "think harder" | EIP-712 signing changes, encryption modifications |
| "ultrathink" | Security analysis, Turnkey provider adapter changes |

## Before Committing

```bash
pnpm build        # Verify build succeeds
```

No linter or test suite is currently configured. When they are added, run those too.

## Incremental Changes

Prefer incremental changes over large rewrites:

```
You: Add a new popup method for unlinking as user
Claude: [creates route, page component, signing logic]

You: Now add the confirmation UI
Claude: [adds UnlinkConfirmModal]

You: Add error handling for failed transactions
Claude: [adds try/catch, error display, user feedback]
```

## Match Existing Patterns

Before generating code:
1. Read the related page/component to understand the pattern
2. Note the auth gating pattern (loading → login → content)
3. Follow the sessionStorage parameter passing convention
4. Use the same UI components (Button, LoadingScreen, LoginForm)
