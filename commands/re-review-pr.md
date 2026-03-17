---
description: |
  Follow-up PR review after the author has addressed previous review comments.
  Reads all prior review threads and discussions to understand context, then
  re-reviews the updated PR to verify issues were addressed and check for
  new problems introduced by the fixes.

  USE THIS COMMAND:
  - After a previous round of review where you left comments
  - When the author says "comments addressed" or pushes new commits
  - To verify unaddressed items and catch regressions from fixes

  USAGE:
    /re-review-pr <PR_URL>
    /re-review-pr https://github.com/newt-foundation/newton-identity/pull/123

  The PR URL is required. It will be passed as $ARGUMENTS.
---

# Follow-Up PR Re-Review for Newton Identity

You are performing a **follow-up review** of a Next.js identity popup PR. A previous review round was completed, and the author claims to have addressed feedback. Verify that claim and catch anything new.

## Input

The user provides a GitHub PR URL as `$ARGUMENTS`. Extract the owner, repo, and PR number.

## Phase 1: Context Recovery

### Step 1: Get PR metadata and diff

```bash
gh pr view <PR_NUMBER> --repo <OWNER/REPO> --json title,body,baseRefName,headRefName,author,state,commits
gh pr diff <PR_NUMBER> --repo <OWNER/REPO>
```

### Step 2: Read ALL previous review comments

```bash
gh api repos/<OWNER>/<REPO>/pulls/<PR_NUMBER>/comments --paginate
gh api repos/<OWNER>/<REPO>/pulls/<PR_NUMBER>/reviews --paginate
gh api repos/<OWNER>/<REPO>/issues/<PR_NUMBER>/comments --paginate
```

### Step 3: Build a review ledger

For each previous comment, categorize:
- **BLOCKING (unaddressed)** — must-fix, not addressed
- **BLOCKING (claimed addressed)** — must-fix, author says fixed → VERIFY IN CODE
- **NON-BLOCKING (acknowledged)** — suggestion/nit deferred
- **RESOLVED** — clearly fixed

## Phase 2: Verify Fixes

For each "claimed addressed" item:
1. Find the relevant code in the current diff
2. Check if the fix is correct and complete
3. Check for regressions introduced by the fix

Pay special attention to:
- Popup protocol changes (postMessage flow, origin validation)
- EIP-712 signing correctness (types matching IdentityRegistry contract)
- Turnkey provider adapter (r/s/v normalization preserved)
- Encryption handling (KMS key sourcing, output format)
- React patterns (hook rules, auth gating, client directives)

## Phase 3: Review New Changes

Review fix commits with full rigor. Pay attention to:
- Fixes that are too narrow (letter not spirit)
- New code introduced by fixes that itself has issues
- Silent behavior changes beyond what was requested

## Phase 4: Produce the Re-Review

### Review body format

If unaddressed items exist:

```text
From previous review, these are still unaddressed:
- <item 1> — still not fixed
- <item 2> — partially addressed but <what's missing>

New issues:

1. <new issue>

LGTM once above are addressed.
```

If all addressed, no new issues: `Previous comments all addressed. LGTM.`

### Voice and Framing

- Use "we/us/let's" not "you"
- Direct imperatives for clear issues
- Prefix labels: `Opinion:`, `FYI:`, `Suggestion:`, `nit:`
- No praise sections
- No severity grouping — flat numbered list
- End with clear merge criteria

### DO NOT USE:
- Emoji headers or bold-label patterns
- Markdown headings in the review body
- Full checkbox lists of all previous comments
- Formal "Summary" sections

### Important

- Make comments inline to the remote PR using MY GitHub `@denniswon` configured in `~/.gitconfig`
- Follow `.claude/PR_REVIEW_GUIDE.md` for style conventions
- Be fair: if the author addressed a concern differently than suggested, accept it if the underlying issue is resolved
