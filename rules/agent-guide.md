# Agent Guide

## Agent Behavior Directives

### Verify Before Claiming

All output — code, docs, comments, CLI instructions, config examples — must be verified against the actual codebase before submission. Do NOT assume or guess:

- **Component props and hooks**: Read the component/hook source to confirm exact props, return values, and side effects before using them
- **Route paths and params**: Verify routes exist in `app/` directory structure before referencing them
- **File paths and naming**: Use Glob/ls to confirm files exist where you say they do
- **Environment variables**: Read `.env.example` or `next.config.ts` to confirm variable names and expected formats
- **Contract ABIs and addresses**: Read `lib/identityRegistry.ts` or deployment configs to confirm ABI fields and deployed addresses

If you are uncertain about any claim, ASK the user rather than guessing. A missing fact is better than a false statement.

### Quality Bar

Treat every code change as if it will be reviewed by a staff engineer:
- Match existing patterns in the file you're editing
- Error messages must include context (identifiers, values that caused the error)
- Follow the auth gating pattern: loading state → login form → actual content

### Mid-Task Replanning

If during implementation you discover the approach will not work (wrong assumption about types, missing API, architectural mismatch):
1. STOP implementing
2. Explain what you discovered and why the current approach fails
3. Propose an alternative
4. Get confirmation before continuing

Do NOT push through a failing approach hoping to fix it later.

### Autonomous Bug Fixing

When you encounter a bug or test failure during implementation:
- **Fix autonomously** if: the fix is in code you just wrote, OR it's a clear typo/import error
- **Ask first** if: the fix touches signing logic, encryption, Turnkey provider adapter, or shared interfaces
- **Always explain** what failed and what you changed
