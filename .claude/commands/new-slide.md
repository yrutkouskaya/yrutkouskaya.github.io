---
description: Add a new slide to the site, step by step, with a stop for the user at every step
argument-hint: [optional slide title]
---

Add a new slide to this site. Work through the steps in order. At every **STOP**, ask exactly one question, then end your turn and wait for the answer. Do not do work from the next step before the answer arrives.

Before step 1, read `CLAUDE.md` and these skills, and follow them:

- `.claude/skills/new-slide-flow/SKILL.md`
- `.claude/skills/create-slide/SKILL.md`
- `.claude/skills/commit-and-push/SKILL.md`
- `.claude/skills/add-page/SKILL.md`

Tell the user once, at the start: drafts only live in this chat, and nothing is committed or pushed until they approve the final result.

If the user gave a title as an argument ($ARGUMENTS), use it as the proposed title in step 1.

## Step 1. Title, place, photos

**STOP 1.** Ask for the slide title (it becomes the sidebar label) and where it goes in the navigation. Propose a title and a position. Then ask for the photos if they are not uploaded yet. Do not build anything until the title, the place and the photos are all there.

## Step 2. Build the slide

Follow `create-slide`: look at the photos, do the privacy check, build the slide, render it at about 1280 px wide, and check the text against the photos.

**STOP 2a (only if needed).** Ask when something is unclear or missing in the photos (covered, cut off, glare), or when the photo cuts off part of the slide and you could complete it. Ask one question, and wait.

**STOP 2b.** Show the render and say what you cropped, masked, or completed. Ask the user to approve the slide image or name changes. Repeat until approved. Do not mention the description, committing, or pushing in this step.

## Step 3. Description

Only after the slide image is approved.

**STOP 3.** Ask for the description text, or offer to draft it from what the user said in step 1. Use the user's exact wording. Show it and fix it with the user until they are happy. If they want no description, skip it.

## Step 4. Final look

**STOP 4.** Show the complete result: every file that will be added or changed, the final slide, and the description. Ask the user to approve it or ask for changes. Go back to the matching step for any change.

## Step 5. Commit and push

Only after explicit approval in STOP 4.

**STOP 5 (only if the slide has image files).** Tell the user that image files are pushed through the GitHub API with a token, and ask for a fine-grained token limited to this repo with Contents: read and write (skip this if a token is already available in the project). Never write the token to a file, a commit, or memory.

Then follow `commit-and-push` and the "Pushing" section of `create-slide`: one commit to `main` with all files, short imperative message. After the push, give the Actions tab link and the expected page URL.
