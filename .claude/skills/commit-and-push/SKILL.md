---
name: commit-and-push
description: Use when the user asks to commit, push, publish, save, or deploy changes to this repo, or when finishing any task that changed files here.
---

# Commit and push changes

In the regular Claude app there is no local git. Changes reach the repo through the GitHub connector, so "commit" and "push" happen in one step: the commit is created directly on `main`.

## Before committing

1. Read `CLAUDE.md` and the matching skill in `.claude/skills/` for the task (for example `add-page` for pages and navigation).
2. List the exact files that will be added or changed, and say so to the user in one short sentence.
3. For every existing file you change, fetch its current content from `main` first and edit that version. Never rewrite a file from memory.
4. Run the secret scan on the new content (`run_secret_scanning`). If it finds anything, stop and tell the user. Do not commit it.
5. Ask before deleting, renaming, or restructuring existing pages or folders.

## Committing

- Repo: `yrutkouskaya/yrutkouskaya.github.io`, branch `main`.
- Use `push_files` for one or more files in a single commit. It needs no file SHA.
- Use `create_or_update_file` only for a single file. When the file already exists, pass its current blob SHA from `get_file_contents` with `ref` set to `main`.
- Group related changes into one commit. Do not split one task into many commits.
- Commit message: short and imperative, for example "Add contact page" or "Update navigation order". No trailing period, no emoji.
- Never force-push, and never rewrite history.

## After pushing

1. Confirm the commit landed with `get_commit` (use the branch name `main` as the ref) and check that the changed files match what was intended.
2. Tell the user:
   - the commit message and the files changed,
   - to check the Actions tab at https://github.com/yrutkouskaya/yrutkouskaya.github.io/actions (the deploy usually takes 1-2 minutes),
   - the expected page URLs, for example `https://yrutkouskaya.github.io/page-name/`.
3. If a changed page looks unstyled once deployed, check its front matter first, then `_config.yml`.

Note: every push to `main` triggers a deploy, even when only `CLAUDE.md` or `.claude/` changed. Those paths are excluded from the published site, so the site itself does not change.
