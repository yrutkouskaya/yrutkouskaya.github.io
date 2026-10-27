# CLAUDE.md

Instructions for Claude when working in this repository.

## About this site

- Jekyll site using the [Just the Docs](https://just-the-docs.com) theme (gem-based, `just-the-docs` pinned in `Gemfile`).
- Hosted on GitHub Pages at https://yrutkouskaya.github.io/.
- Every push to `main` triggers the workflow in `.github/workflows/pages.yml`, which builds and deploys the site (usually 1-2 minutes).

## Skills

Read the matching skill before doing the task:

- `.claude/skills/add-page/SKILL.md`: adding or changing pages, sections, navigation, and routes.
- `.claude/skills/commit-and-push/SKILL.md`: committing and pushing any change to `main`.
- `.claude/skills/new-slide-flow/SKILL.md`: adding a new slide, step by step (slide and photos, description, final look, then one commit at the end).
- TBA, not written yet (do not look for the file): `.claude/skills/create-slide/SKILL.md`, which will turn uploaded photos into a slide. The new-slide flow will use it in step 2.

## General rules

- Follow the skills above; they describe how routes and navigation work on this site.
- Ask before deleting or restructuring existing pages.
- Use short, imperative commit messages (for example, "Add contact page").
- After pushing, tell the user to check the Actions tab, and give the expected page URLs.
- `CLAUDE.md` and `.claude/` are excluded from the published site in `_config.yml`. Keep them there.

## Coding and content rules

(To be added by the user.)
