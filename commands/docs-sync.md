# docs-sync

Analyze branch changes and ensure all documentation and code comments are accurate.

## Phase 1: Gather Context

1. Read `.claude/rules/` thoroughly.
2. Run `git log --oneline $(git merge-base HEAD main)..HEAD` to list commits.
3. Run `git diff $(git merge-base HEAD main)..HEAD` for code changes.
4. Run `git diff $(git merge-base HEAD main)..HEAD --stat` for file summary.

## Phase 2: Audit Documentation

Locate and read:
- `CLAUDE.md` (root)
- `docs/` folder — all documentation files
- `README.md`
- Any `.md` files relevant to changed code

For each, evaluate:
- Does it accurately reflect the current code?
- Are popup protocol descriptions correct?
- Are EIP-712 types/domain documented correctly?
- Are environment variable references up to date?
- Is new functionality missing documentation?

## Phase 3: Audit Code Comments

Review changed files:
- Are existing comments still accurate?
- Are there complex sections lacking a "why" explanation?
- Remove stale or misleading comments.
- Remove leftover debug `console.log` statements.

Comment principles:
- Only comment where the "why" is non-obvious
- Concise and clear. No filler.
- No TODO comments unless actionable and tracked.

## Phase 4: Apply Updates

1. Update documentation with accurate information.
2. Fix or remove stale code comments.
3. Add minimal necessary comments where "why" context is missing.
4. Ensure terminology consistency (Identity Owner EOA, App EOA, Policy Client, Identity Domain).

## Phase 5: Evolve Agent Instructions

Review `.claude/` folder:

**Rules:** Are there new patterns or lessons learned that should become rules?
**Commands:** Are there workflows that could become reusable commands?
**CLAUDE.md:** Update project context if architecture has changed.

## Rules

- Follow all instructions in `.claude/rules/`.
- Do not add verbose or redundant comments.
- Do not fabricate features — only document what exists.
- No emojis. No AI attribution.

## Output

Summary of: files modified, docs changes, comments updated, `.claude/` contributions, gaps requiring human decision.
