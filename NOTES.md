# Notes for future edits

Internal design notes on `index.html` — the "why" behind values that aren't obvious from the code alone. Read this before tuning difficulty, adding a relationship type, or touching the wheel.

`index.html` holds the whole toolkit: the quiz, the tools, and the hash router that gives each screen a URL.

## Wheel geometry

The wheel is drawn with two stacked CSS gradients on `.wheel-disc` (see `buildWheelGradient`): a `conic-gradient` for hue, with a white `radial-gradient` on top that fades out toward the edge.

That radial gradient **must** say `circle closest-side`. The CSS default is `farthest-corner`, and on a square box the corner is at 0.707×width while the disc edge is at 0.5×width — so the default ramp is only ~71% finished at the rim and leaves roughly 29% white lying over the entire wheel, washing it out. With `closest-side` the ramp ends exactly at the disc edge and saturation equals radius, which is both the correct HSB geometry and what Adobe's wheel measures as. Since HSB saturation at brightness=100 is exactly a linear mix toward white, this reproduces a true HSB wheel without a canvas or per-pixel math. The hues fed into that gradient are warped — see the next section.

## The wheel is RYB, not raw HSL

**Both the quiz and the picker plot Adobe's artist wheel, where red sits opposite green and cyan opposite orange** — not the raw HSL circle, where red sits opposite cyan. `WHEEL_HUE` is a 36-entry lookup table (one entry per 10° of wheel angle) giving the RGB/HSL hue drawn at that angle; `wheelToHue()` and `hueToWheel()` interpolate in each direction.

The table was produced by sampling the Adobe Express colour-wheel widget from a screenshot at 0.25° resolution around the rim and converting each sample to its true RGB hue. It checks out against Adobe's own output: the palette pair `#FC0F35` / `#0FFC3C` that Adobe generates as complements lands 179° apart on this wheel.

The consequence for the quiz: **every hue value in the question generator is a wheel angle, not an HSL hue.** All the scheme math (`genShape`, `angDiff`, targets, tolerances) stays in wheel-angle space, and the conversion happens only at the paint step, via `wheelRgb(wheelAngle, s, b)`. If you add a code path that turns a quiz hue into a colour, call `wheelRgb`, never `hsbToRgb` directly — the latter would silently paint the raw-HSL colour at that angle and the dot would no longer match the wheel underneath it.

Angle convention: **0° (red) is at the top, increasing clockwise** — matches `conic-gradient(from 0deg, ...)`'s default direction. `hueToXY()` and the click-handler's inverse math in `attachWheelClickHandler` must stay in sync with this; if you ever change one, change the other.

Everything else (given colours, option dots, target rings) is plotted in an SVG overlay (`viewBox="0 0 100 100"`, wheel angle mapped to angle, saturation mapped to radius as a 0–1 fraction).

## Difficulty tuning (`DIFFICULTIES`)

- `tol` is the angular slop (in degrees) allowed before a relationship stops counting — this is the "good enough" rule the unit teaches (e.g. complementary at 170° instead of 180° still counts). Easy uses `tol:0` (exact angles only, no ambiguity for beginners).
- `allowNA` / `naProb` control the "No relationship applies" trick questions, only enabled on Medium (and lightly on Mixed). See below — this needed careful dead-zone tuning to avoid false positives.
- Hard's `sat`/`bri` used to be deliberately low (to make the option dots hard to distinguish in a multiple-choice layout). That mechanic is gone now that Hard is free-click — saturation is cosmetic there, so its range was bumped back up to match Medium for visibility. Don't lower it again without a reason; it was raised on purpose.

## Monochromatic: spread, not independent jitter

`genShape`'s Monochromatic case caps the **whole set's** spread at `MONO_MAX_SPREAD` (10°) directly, rather than jittering each hue independently by `±tol`. Independent jitter can double the pairwise spread (two hues each drifting the full `tol` in opposite directions), which produced sets that blew past the 10° threshold that actually defines "monochromatic" for this unit. If you touch this again: pick one spread value first, then place hues inside it — never jitter each point independently.

Saturation/brightness for monochromatic points is drawn from a wide, fixed range (`MONO_SAT_RANGE`, `MONO_BRI_RANGE`) using `stratifiedSpread()`, not the difficulty's own (often narrow) `sat`/`bri` range. This guarantees the dots land at visibly different radii along the same angle instead of clustering on top of each other — narrow ranges caused real overlap in testing.

## Analogous vs. "no relationship" ambiguity

Two hues that are one analogous step apart (roughly 15–45°, `ANALOGOUS_SPACING` is 26°) are legitimately analogous, even as a pair. The N/A generator's "skewed pair" variant (`genNAShape`) must avoid landing in that range, or a valid analogous-looking answer gets marked N/A. Current dead zone for the 2-point N/A case starts at 50°, well clear of the analogous step range and clear of 120°/150°/180° (±`tol`+8°) too.

The other N/A variant ("broken fan") makes 3 evenly-spaced hues plus a 4th outlier. The outlier jump must be large enough to read as unambiguously *not* a continuation of the fan — this was bumped from 70–130° to 110–170° after a 102° jump tested as visually borderline.

## Split-complementary always shows two given colours

Originally Hard's split-complementary question showed only the base colour and asked for "a" split-complementary partner — but there are two valid answers (either flank), which doesn't work for a single free-click target. Fixed by always showing the base **and** one randomly-chosen flank, with the target being the other flank — this makes every split-complementary question have exactly one correct answer, same pattern as triadic/tetradic.

## Adding a new relationship type

If you add a 7th relationship type, you need to touch all of:
1. `RELATIONSHIPS` array
2. `genShape()` — Easy/Medium's "identify" shape generator
3. `explainIdentify()` — the plain-English explanation text
4. `genPlaceQuestion()`'s switch — Hard's free-click shown/target logic
5. Swatch preview hues in `renderDiffGrid()` if you want the start-screen cards to reflect it (they currently don't reference relationship types directly, so this is optional)

## Known trade-offs (not bugs, just choices)

- No persistence — closing the tab loses your streak. Deliberate; this is meant to be a disposable drill, not a tracked gradebook.
- No keyboard-accessible way to answer Hard-mode questions (the wheel is a plain click target, no arrow-key placement). Fine for a classroom laptop/tablet context; would need work for a fully accessible deploy.
- Explanations are template strings, not proper i18n. Fine for a single-language classroom tool; would need restructuring for translation.


## Naming: "wheel angle" vs "H"

These are two different numbers for the same colour, and the rule is stronger than careful labelling: **no screen ever states both.**

- **The picker states only `H`** — sliders, `hsl()`, `rgb()`, hex. Its wheel carries degree ticks (a protractor scale on the drawing), but nothing is labelled as the colour's angle. Don't reintroduce an angle readout here; that's what the removed `wheel-readout` block was, and it forced the "which number is the real one?" question onto every student.
- **The quiz states only wheel angles** — it never renders an `H` value, so degrees carry no ambiguity there and "complements are 180° apart" holds exactly.

Never write `°` after an `H` value, and avoid the phrase "hue angle" entirely — it reads as a physical angle on the wheel, which is what caused the confusion originally. Say "wheel angle" or "H".


## Image export

`renderExport()` draws the picker onto an offscreen canvas at 2x and `downloadImage()` hands it over as a JPG. It is **not** a screenshot of the DOM — nothing is rasterised from the page, so there's no html2canvas-style dependency and no tainted-canvas risk, and the output is deterministic: always the light palette at a fixed size, whatever theme or screen the user is on.

The trade-off is that the export duplicates the layout rather than deriving it, so a visual change to the picker does not reach the image. If you restyle the picker, update `renderExport()` to match or the two will drift apart.

The hue ring is drawn as 720 half-degree wedges with a 0.75° sweep each, the slight overlap hiding seams between them; `createConicGradient` would be tidier but is less widely supported. Wedge hues go through `wheelToHue()`, so the exported wheel is the same RYB wheel as the screen.

Text that could overflow is measured first: the header subtitle is placed at the measured width of the wordmark, and a footer URL wider than the content box falls back to the bare `#picker/<hex>` route.
