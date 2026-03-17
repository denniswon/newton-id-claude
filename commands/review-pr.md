---
description: Review current PR changes for correctness, security, and code quality
---

Review my changes against the main branch.

Run `git diff main...HEAD` to get the full diff, then analyze all changed files. Prioritize popup security (postMessage origins, XSS), EIP-712 signing correctness, Turnkey provider safety, encryption handling, React hook rules, and auth gating.

Provide feedback as plain prose with specific file:line references. No emoji headers, no bold-label patterns like "**Problem:**", no markdown headings in the output. Write like a senior engineer leaving terse, direct review comments.

Skip files that are auto-generated, formatted (handled by Prettier), or have only trivial changes.

Follow the comment style in `.claude/PR_REVIEW_GUIDE.md`.
