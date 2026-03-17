# pr

Create a pull request for the current branch against its parent branch.

## Steps

1. Run `git log --oneline $(git merge-base HEAD main)..HEAD` to understand all commits.
2. Run `git diff $(git merge-base HEAD main)..HEAD --stat` for file summary.
3. Run `git diff $(git merge-base HEAD main)..HEAD` to read the actual diff.
4. Determine the parent/base branch. Use `git log --oneline --graph --all -20` if unclear.
5. Write a conventional commit title: `type: subject` (feat, fix, refactor, docs, test, chore, perf, ci, build).
6. Write a PR description as plain prose — short paragraphs like a human developer would write. Cover what changed, why, and anything a reviewer should know.
7. Create the PR using `gh pr create --base <parent-branch> --title "<type>: <subject>" --body "<description>"`.

## Rules

- **Scope to THIS branch only** — diff between merge-base and HEAD.
- Be concise yet informative. No filler.
- No emojis anywhere.
- Use the git user already configured. Do not modify git config.
- **Never** include AI attribution (no "Created by Claude", "Co-authored-by: Claude", etc.)
- Do not add `--reviewer`, `--assignee`, or `--label` flags unless explicitly asked.

## Style (avoid looking AI-generated)

- No markdown headings (`##`, `###`) in the body
- No bold-label patterns like `**What:**`, `**Why:**`
- No bullet-point-heavy structure. Prefer sentences and short paragraphs
- No summary/overview sections that restate the title
- No "This PR..." opener — describe the change directly
- No words like "comprehensive", "robust", "seamless", "enhance", "leverage", "streamline"
- Write like a terse engineer
