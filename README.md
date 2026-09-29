# Hue Harmony

A single-file, self-study quiz for practicing colour-scheme relationships on an HSB colour wheel — built for a TAS1O communications technology unit covering monochromatic, analogous, complementary, split-complementary, triadic, and tetradic schemes.

No build step, no dependencies, no backend. It's one `index.html` file that runs entirely in the browser.

## Levels

- **Easy** — a full palette is plotted on the wheel; name the relationship. Angles are exact.
- **Medium** — same task, but angles drift slightly (a "good enough" complementary pair might be 170° apart), and some examples deliberately fit no clean scheme — you pick "No relationship applies."
- **Hard** — given one or two colours and a named scheme, click anywhere on the wheel to place the missing colour. Only the hue (angle) is graded; saturation is ignored.
- **Mixed** — a shuffled round pulling from all three levels above.

Each round is 10 questions, randomly generated, with immediate feedback and a short explanation after every answer. Replay is unlimited — nothing is saved between sessions.

## Running it

Just open `index.html` in a browser. That's it.

## Deploying

This is a static site, so any static host works. For Netlify:

1. Connect this GitHub repo via Netlify's "Import from Git" flow (recommended — future pushes to `master` auto-deploy).
2. Build command: none. Publish directory: `/` (repo root).

Or drag the repo folder straight into Netlify's manual deploy dropzone if you'd rather not connect Git (note: manual deploys don't update automatically on future pushes — see `NOTES.md`).

## Editing

See `NOTES.md` for how the question generator works and the reasoning behind the difficulty tuning, if you want to adjust it.
