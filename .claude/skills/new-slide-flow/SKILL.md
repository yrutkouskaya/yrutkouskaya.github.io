---
name: new-slide-flow
description: Use when the user wants to add a new slide to this site. Runs a step-by-step flow (slide, photos, description, final look) and commits everything only at the end.
---

# New slide: step-by-step flow

Work through the steps in order, one at a time. Ask one question per step and wait for the answer before moving on.

**Core rule: nothing is committed or pushed until the user approves in step 4.** Keep all drafts (slide files, text, images) in the conversation. Tell the user once, at the start, that drafts only live in this chat until the final commit.

## Step 1. The slide and the photos

1. Ask what the new slide is about and what it should be called.
2. Ask the user to upload the photos for it.
3. Do not start building until both are given.

## Step 2. Build the slide

- Use the slide-creation skill to turn the photos into a good slide.
- **TBA:** that skill does not exist yet (planned: `.claude/skills/create-slide/SKILL.md`). Until it exists, tell the user it is not written yet and ask how they want the slide made. Do not invent the layout or file structure.
- Show the user the draft and apply their changes until they are happy with it. Do not commit.

## Step 3. Description

1. Ask the user for the description text, or offer to draft it from what they said in step 1.
2. Add it to the slide. Fix wording and typos together with the user until they are happy with it.
3. Do not commit.

## Step 4. Final look, then commit

1. Show the complete result: every file that will be added or changed, the final slide, and the description.
2. Ask the user to approve it or ask for more changes. Go back to the right earlier step for any change.
3. Only after explicit approval, follow `.claude/skills/commit-and-push/SKILL.md` to commit everything in **one** commit to `main`.
4. If the slide includes image files, check before committing whether they can be pushed. The GitHub connector takes text content, so binary photos may not go through. Tell the user about this before the final commit, not after.
5. After the push, give the user the Actions tab link and the expected page URL, as the commit-and-push skill describes.
