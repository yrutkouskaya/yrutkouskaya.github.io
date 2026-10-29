---
name: new-slide-flow
description: Use when the user wants to add a new slide to this site. Runs a step-by-step flow (slide, photos, description, final look) and commits everything only at the end.
---

# New slide: step-by-step flow

Work through the steps in order, one at a time. Ask one question per step and wait for the answer before moving on.

**Core rule: nothing is committed or pushed until the user approves in step 4.** Keep all drafts (slide files, text, images) in the conversation. Tell the user once, at the start, that drafts only live in this chat until the final commit.

## Step 1. The slide and the photos

1. Ask what the new slide is about and what it should be called (the page title, which becomes the sidebar label, and where it goes in the navigation).
2. Ask the user to upload the photos for it.
3. Do not start building until both are given.

## Step 2. Build the slide

- Follow `.claude/skills/create-slide/SKILL.md` to turn the photos into the slide.
- Show the user the draft and apply their changes until they are happy with it. Do not commit.
- Finish this step by asking the user to approve the slide image. Do not ask for the description yet.

## Step 3. Description

Only start this step after the user has approved the slide image in step 2.

1. Ask the user for the description text, or offer to draft it from what they said in step 1.
2. Add it to the page, using the user's exact wording. Fix wording and typos together with the user until they are happy with it.
3. Do not commit.

## Step 4. Final look, then commit

1. Show the complete result: every file that will be added or changed, the final slide, and the description.
2. Ask the user to approve it or ask for more changes. Go back to the right earlier step for any change.
3. Only after explicit approval, follow `.claude/skills/commit-and-push/SKILL.md` to commit everything in **one** commit to `main`.
4. If the slide includes image files, tell the user before the final commit that images are pushed through the GitHub API with a token, because the GitHub connector takes text only. Section "Pushing" of the create-slide skill describes how, and ask for the token at that point.
5. After the push, give the user the Actions tab link and the expected page URL, as the commit-and-push skill describes.
