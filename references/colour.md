# Colour

The figures in this skill are drawn in one ink. That is the default, and it is deliberate: a monochrome figure is read as a drawing of a machine, and the reader's eye goes to the one lit part instead of to a palette. `references/craft.md` puts the rule as: **keep it monochrome unless colour is the subject.**

Colour is still available, and it is not paint. It is the same four-tone shading, solved from a different base hue. A coloured figure is exactly as strict as a grey one: same parts, same painter's order, same audits, same one accent.

## What the kit actually colours

Every fill in the kit is a CSS custom property, evaluated per element:

| token | what it fills |
|---|---|
| `--anatomy-paper` | the opaque back face of a solid (hides what is behind it) |
| `--anatomy-top` | a top face |
| `--anatomy-shade-0` … `--anatomy-shade-3` | the four side tones, darkest to lightest |
| `--anatomy-lit-top`, `--anatomy-lit-shade` | the top and sides of a `lit` solid |
| `--anatomy-hi` … `--anatomy-faint` | ink: outlines, creases, rules, text |
| `--anatomy-lit` | the one accent, and the outline of a `lit` solid |

So a colour is a scope, not a property. Set the fill tokens on a `<g>` and every solid under it inherits that material. The ink tokens stay unresolved at the root, so the whole drawing keeps sharing one ink — that is what stops a coloured figure from turning into an illustration.

## The recipe

One group per material, one base hue each, and a ramp derived with `color-mix`, so the four tones stay ordered and the material can be re-hued by changing one value:

```js
const wrap = (cls, body) => `<g class="${cls}">${body}</g>`;

// In the page body, after the kit CSS (a <style> in body wins on equal specificity):
const COLOUR_CSS = `
.mat-base,.mat-steel,.mat-carriage,.mat-dial,.mat-accent{
  --anatomy-top:color-mix(in oklab,var(--b) 80%,white);
  --anatomy-shade-3:color-mix(in oklab,var(--b) 92%,white);
  --anatomy-shade-2:var(--b);
  --anatomy-shade-1:color-mix(in oklab,var(--b) 84%,black);
  --anatomy-shade-0:color-mix(in oklab,var(--b) 68%,black);
  --anatomy-paper:var(--anatomy-shade-2);
  --anatomy-face:var(--anatomy-top);
  --anatomy-lit-top:var(--anatomy-top);
  --anatomy-lit-shade:var(--anatomy-shade-1);
  --anatomy-lit:color-mix(in oklab,var(--b) 55%,black);
}
.mat-base{--b:#dcd0b8}
.mat-steel{--b:#b4c0cd}
.mat-carriage{--b:#3f4a56}
.mat-dial{--b:#f0e6d2}
.mat-accent{--b:#c4452f}
.iso[data-theme="dark"] .mat-base{--b:#4a4335}
/* …one dark value per material… */
`;
```

and wrap the parts: `wrap("mat-base", plate())`, `wrap("mat-steel", fixture())`, `wrap("mat-accent", plunger())`. A moving part keeps its `data-part` group and its transform; the material group can sit either side of it.

`examples/dial-indicator-colour/` is the worked example: the same figure as `dial-indicator/`, in five materials, with the needle and plunger on `mat-accent`. It builds byte-for-byte like any other example and passes the same `capture.mjs` checks.

## Rules that still hold

- **The ramp is not free.** Never set a single fill and leave the other tones global: the part will read as a flat cutout. Derive all of `-0` to `-3` and the top from one `--b`, as above; the ordering (top > shade-3 > shade-2 > shade-1 > shade-0) is what makes the machine read as solid.
- **One accent.** Accent parts (`mat-accent`) earn their colour by being the thing that moves or the thing being explained. If everything is coloured, nothing is.
- **One ink.** Outlines, creases, tick marks and text stay `--anatomy-hi`/`mid`/`lo`/`faint` from the root. A second ink is how a technical drawing becomes a cartoon.
- **Light theme first.** Colour is designed for the light theme, as every figure is; a dark value per material is a separate, darker base, not the same hex on a black ground. Check both themes before shipping.
- **Shaders read tokens.** A WebGL layer takes its ink from the same tokens (`references/webgl.md`, 8.3). A colour typed into GLSL is a theme bug waiting to happen.
- **The kit still owns the drawing.** A material changes fills only; tone choice (`lo`/`mid`/`hi`/`lit` per part), painter's order, audit and line ends are unchanged. Run the same verification: `node build.mjs --audit` where it applies, `scripts/lines.mjs`, and 4× close-ups in both themes (`references/verify.md`).
- **Community precedent.** Gallery figures such as `entity/gumball-machine` use the same pattern — a per-material class overriding `--anatomy-*` with `color-mix`. The gallery accepts any theme or colours.

## When a request asks for colour

Say what is being coloured and why, in the same breath as the mechanism: "the body is enamel red, the mechanism stays grey metal, the one moving part is the accent". Then:

1. Write the materials out as a list before geometry, next to the parts list (workflow step 4). Each part gets a material name, not a hex.
2. Pick one base hue per material; keep them in one family and one value range, or the figure will separate into unrelated objects.
3. Keep the subject on the accent and the base/feet on the darkest material, so the tone hierarchy survives colour.
4. Verify in light and dark, at 1× and 4×, exactly like a monochrome figure.

If the subject is not about colour, do not colour it: a grey figure is the house style, and the answer to "why is it not colourful" is that the colour would be decoration.
