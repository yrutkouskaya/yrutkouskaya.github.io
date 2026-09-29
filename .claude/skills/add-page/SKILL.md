---
name: add-page
description: Use when adding, moving, or renaming pages or sections on this Just the Docs site, or when changing navigation order, parents, or URLs.
---

# Adding pages and routes (Just the Docs)

## Front matter

Every page is a `.md` file that starts with front matter, otherwise Jekyll does not build it as a page.

```yaml
---
title: Page Title
nav_order: 2
---
```

- `layout: default` is applied to all pages by `_config.yml`. Only the home page sets `layout: home` explicitly. Do not remove the defaults block from `_config.yml`: without a layout the page renders as bare HTML with no styles or sidebar.
- `title` is required and becomes the sidebar label. Keep titles unique, because children refer to parents by title.
- `nav_order` controls sidebar order among pages at the same level.

## Top-level page

Create `page-name.md` in the repo root, with front matter. Served at `/page-name/`.

## Section with child pages

Use a folder. The folder's `index.md` is the parent page and is served at `/folder-name/`.

```
folder-name/
  index.md      (parent page)
  child-one.md  (served at /folder-name/child-one/)
  child-two.md
```

Parent (`folder-name/index.md`):

```yaml
---
title: Folder Title
nav_order: 2
---
```

Child (`folder-name/child-one.md`):

```yaml
---
title: Child One
parent: Folder Title
nav_order: 1
---
```

- `parent` must exactly match the parent page's `title`.
- `has_children` is redundant in current versions of the theme (0.10.0 and later), so do not add it.
- Deeper levels work the same way: a page sets `parent` to its own parent's title, and the folder gets its own `index.md`.

## URLs and file names

- `permalink: pretty` is set in `_config.yml`, so `folder/page.md` is served at `/folder/page/`, not `page.html`.
- Use lowercase file and folder names with hyphens, no spaces.
- Do not put a page named `index.md` anywhere except as a folder's parent page or the home page.

## Links

- Link between pages with relative paths, for example `[text](child-one/)` or `[text](/folder-name/child-one/)`.
- Link to the trailing-slash URL, not to `.md` or `.html`.

## After changing pages

1. Commit to `main`. The Pages workflow deploys automatically.
2. Tell the user to check the Actions tab, then open the new URLs.
3. If a page looks unstyled, check its front matter first, then `_config.yml`.
