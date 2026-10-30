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
- `.claude/skills/create-slide/SKILL.md`: turning uploaded photos into a slide (photo cleanup, slide markup, image files, pushing images). The new-slide flow uses it in step 2.

## General rules

- Follow the skills above; they describe how routes and navigation work on this site.
- Ask before deleting or restructuring existing pages.
- Use short, imperative commit messages (for example, "Add contact page").
- After pushing, tell the user to check the Actions tab, and give the expected page URLs.
- `CLAUDE.md` and `.claude/` are excluded from the published site in `_config.yml`. Keep them there.

## Coding and content rules

### Writing rules

These rules come from ASD-STE100 Simplified Technical English (STE). They apply to text that Claude writes: chat replies, drafted slide descriptions, commit messages, and changes to `CLAUDE.md` and `.claude/`.

They do not apply to:

- Slide text copied from photos. Copy it exactly (see `create-slide`).
- Descriptions that the user wrote. Use the user's exact wording.
- Medical terms, drug names, doses, units, and quotes. Do not change them.

Write in the language of the user's message or of the page. The STE word list exists for English only. In Dutch or another language, use the rules for verbs, sentences, and paragraphs, and use the word rules as far as they fit.

**Words**

1. Use one word for one meaning. Use the same term for the same thing each time. Do not use synonyms to vary the text.
2. Choose the simple, common word ("use", not "utilize"). Do not use idioms or figurative language.
3. Do not write a noun string of more than three words. Rewrite it with a verb or a preposition.
4. Keep the technical and medical terms of the source. Do not replace them with simpler words.

**Verbs**

5. Use only these forms: infinitive, imperative, simple present, simple past, simple future, and past participle (only as an adjective).
6. Do not use auxiliary verbs to make complex verb constructions.
7. Use the "-ing" form only as a technical noun or as a modifier in a technical noun.
8. Use the active voice. In descriptions, use the passive voice only when the agent is unknown.

**Sentences**

9. Write no more than 20 words in an instruction and no more than 25 words in a description.
10. Write one instruction per sentence.
11. Do not omit the subject, the verb, or the article to make a sentence shorter.
12. Make instructions clear and specific. Give the exact value with its unit.

**Paragraphs and lists**

13. Write one topic per paragraph. Write no more than six sentences in a paragraph.
14. Use a vertical list for complex text.
15. Start a safety or privacy instruction with a clear command or condition. Put it before the step it protects. Example: "If a photo shows a face, stop and ask the user."

STE is a standard of ASD with 53 rules. This list is a short version. Add rules from the full standard (free at https://asd-ste100.org/) when needed.

### Coding rules

(To be added by the user.)
