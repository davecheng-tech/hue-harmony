# Hue Harmony

**Colour & graphic design toolkit** — a single-file, self-study resource built for a TAS1O communications technology unit.

The landing page splits into two sections:

- **Quiz** — practice naming and placing colour-scheme relationships on the wheel: monochromatic, analogous, complementary, split-complementary, triadic and tetradic.
- **Tools** — interactive colour utilities, starting with an HSL picker and hue wheel.

Both plot the same wheel: Adobe's artist (RYB-style) wheel, where red sits opposite green. See [The wheel](#the-wheel) below.

No build step, no dependencies, no backend. It's one `index.html` file that runs entirely in the browser.

## Levels

- **Easy** — a full palette is plotted on the wheel; name the relationship. Angles are exact.
- **Medium** — same task, but angles drift slightly (a "good enough" complementary pair might be 170° apart), and some examples deliberately fit no clean scheme — you pick "No relationship applies."
- **Hard** — given one or two colours and a named scheme, click anywhere on the wheel to place the missing colour. Only the wheel angle is graded; saturation is ignored.
- **Mixed** — a shuffled round pulling from all three levels above.

Each round is 10 questions, randomly generated, with immediate feedback and a short explanation after every answer. Replay is unlimited — nothing is saved between sessions.

## Tools

### HSL colour picker

Sliders and numeric inputs for hue, saturation and lightness, with live `hsl()` / `rgb()` / hex readouts and a hex input for going the other way. Alongside the sliders, the same hue wheel the quiz uses plots the current colour:

- The hue pointer always runs the **full radius**, so the angle stays readable even for near-grey, low-saturation colours.
- A marker sits along that line at the saturation distance from the centre.
- The wheel's shading tracks the current lightness.
- Clicking or dragging on the wheel sets hue and saturation.

**Copy link** puts a URL for the current colour on the clipboard. **Download image** saves a JPG of the whole picker — wheel, swatch, `hsl()`/`rgb()`/hex values and sliders — named after the colour (`hue-harmony-ffe600.jpg`), with the colour's link along the bottom so the image points back at the live tool. The image is drawn on a canvas rather than screenshotting the page, so it's always the light theme at a fixed 1800×1652, identical from any machine.

## The wheel

The wheel is aligned to the **Adobe Express colour wheel**, not a raw HSL circle. That means it's the artist's (RYB-style) wheel: red sits opposite green, and cyan opposite orange. Red is at 0°/360° at the top, running clockwise through orange (~30°), yellow (~120°), green (180°), cyan (~216°), blue (~270°) and magenta (~330°).

Because of that warp, a position on the wheel is *not* the same number as the HSL `H` value of the colour sitting there — the wheel is a layout, not a restatement of `H`.

The UI keeps those apart by never putting both numbers on one screen:

- **The picker states only `H`**, in the sliders and the `hsl()` / `rgb()` / hex rows. Its wheel is marked in degrees like a protractor, so you can *see* the angle, but no number is labelled as the colour's angle.
- **The quiz states only wheel angles.** It never shows an `H` value, so degrees are unambiguous there, and "complements are 180° apart" is exactly true.

The teaching line: **degrees describe positions on the wheel; `H` describes the colour.**

The mapping was derived by sampling the Adobe Express wheel widget pixel by pixel; it's the `WHEEL_HUE` lookup table in `index.html`, with `wheelToHue()` / `hueToWheel()` converting in each direction.

Both sections use this wheel, so "complementary" means red/green in the quiz too. The quiz's scheme maths all runs in wheel-angle space; see `NOTES.md`.

## Links

Every screen has a URL, using a hash so the whole thing stays a static file with no server rewrites and no 404 on reload — it works on Netlify and straight off disk alike.

| URL | Opens |
| --- | --- |
| `#home` | Main menu |
| `#quiz` | Level menu |
| `#quiz/easy` (`medium`, `hard`, `mixed`) | Starts a round at that level |
| `#tools` | Tools menu |
| `#picker` | The picker |
| `#picker/3a7fc1` | The picker, opened on that colour |

The picker keeps the address bar on the current colour as you drag, via `replaceState` so it doesn't fill the back button with every intermediate shade. **Copy link to this colour** puts that URL on the clipboard. An unrecognised route falls back to the main menu, and a malformed hex is ignored rather than throwing.

Handy for a handout or a slide: `…/#quiz/hard` drops students straight into a round, and `…/#picker/3a7fc1` opens a specific colour for discussion.

## Running it

Just open `index.html` in a browser. That's it.

## Deploying

This is a static site, so any static host works. For Netlify:

1. Connect this GitHub repo via Netlify's "Import from Git" flow (recommended — future pushes to `master` auto-deploy).
2. Build command: none. Publish directory: `/` (repo root).

Or drag the repo folder straight into Netlify's manual deploy dropzone if you'd rather not connect Git (note: manual deploys don't update automatically on future pushes — see `NOTES.md`).

## Editing

See `NOTES.md` for how the question generator works, the reasoning behind the difficulty tuning, where the wheel mapping came from, and the naming rule that keeps wheel angles and HSL `H` values apart.
