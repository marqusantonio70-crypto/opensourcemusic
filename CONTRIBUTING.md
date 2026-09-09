# Contributing to Openitify

Openitify is one HTML file. That is the point: anyone can read the whole app in a sitting.

## Ways to help
- **Themes** - one block of CSS custom properties per theme (see `:root` in `index.html`).
- **Genre profiles** - add a style to the engine catalogue (bpm, kit, chord progression, bass/lead patterns).
- **Lyrics** - improve the auto-writer, or ship `.lrc` packs.
- **A11y + i18n** - labels, focus order, translations.

## Rules
1. No build step, no dependencies, no network calls at runtime.
2. No copyrighted audio or artwork in the repo.
3. Keep it working from `file://`.
4. Test in Chrome, Safari and Firefox before opening a PR; include a screenshot or clip.
