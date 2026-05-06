---
name: Do not add Claude as Co-Authored-By in git commits
description: User doesn't want Claude attributed in git commit messages — always omit the Co-Authored-By trailer.
type: feedback
originSessionId: 0cce7df7-ada9-40dd-8f6b-fd6d5c53ecb7
---
Never append `Co-Authored-By: Claude ...` to commit messages. Keep the subject line only (+ body if genuinely warranted), no AI attribution trailer.

**Why:** User explicitly asked to remove "claude committed" signatures from commits (session 2026-04-22). The public commit history is reviewed by their team and external parties; they want it to look like human-authored work.

**How to apply:** When running `git commit -m`, pass only the message the user approved. Do NOT append the Co-Authored-By line that the default system prompt suggests. Applies to amends and new commits alike, across all repos the user works in on this machine.
