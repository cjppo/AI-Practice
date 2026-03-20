## Compact Instructions

When compressing, preserve in priority order:

1. Architecture decisions (NEVER summarize)
2. Modified files and their key changes
3. Current verification status (pass/fail)
4. Open TODOs and rollback notes
5. Tool outputs (allow to be deleted, keep pass/fail only)

## Never Ever Do

Things you are not allowed to do ever:

1. Deleting system sensitive files without explicit approval (when you have to, ask firstly and rename it with a `.bak` suffix insteadly and log what you did in file: `Sensitive-File-Changes.md`)
2. Tiptoing around the real issue/problem
3. Modify `.env`, lockfiles, or CI secrets without explicit approval

## Thinking Instructions

Think from first principles. Don’t just follow the old playbook or rely on 'how we've always done it.' Never assume I have the full picture—stay sharp and always go back to the core problem. If the goal is fuzzy, pause and check in with me. If the goal is clear but there's a better way, call it out and show me a faster, leaner path.
