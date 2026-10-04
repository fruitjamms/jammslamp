---
name: quiet-depth
description: A palette-agnostic surface system for clean, minimal, light UIs where depth comes from layered inner and drop shadows instead of borders, originally extracted from the Bitface app. Covers the ring / lit-edge / whisper-shadow stack, pressed-in wells, the elevation ladder, hover-lift and press-sink states, focus halos, colored buttons that look lit, and subtle light gradients. Use when building or restyling any light-mode interface that should feel calm, dense, and tactile, or when the user mentions quiet-depth, Bitface style, inner shadow, drop shadow, soft shadows, elevation, depth, hairline ring, wells, "clean minimal UI", or "make it look like Bitface".
---

# Quiet Depth

A way to make a light UI look calm and expensive with almost no visible
decoration. There are no borders on raised things, no heavy shadows, and no
strong gradients. Every surface is defined by a stack of three or four very
faint shadows, and the stacks change as you interact.

Colors are inputs, not the point. Pick any `--ink` and `--accent` and the rest
derives.

## Why it looks like this

1. **Shadows are tinted with the ink color, never pure black.** A shadow made
   of `#171014` at 5% reads as depth. Black at 5% reads as dirt.
2. **Edges are a 0.5px inset ring, not a border.** It renders as a true
   hairline on retina screens and a faint 1px line on 1x, it doesn't take up
   layout space, and it keeps white-on-near-white surfaces legible where a
   drop shadow alone would vanish.
3. **Drop shadows come in pairs: one tight, one wide.** The tight one sits the
   object on the page; the wide one gives it air. Both stay under ~7% alpha.
4. **Inputs are pressed in, not drawn around.** Wells carry their shadow
   *inside*, so the page has two directions of depth: up (cards, buttons) and
   down (fields, tool groups).
5. **Three materials, one step apart.** Paper (the page), card (raised, pure
   white), well (pressed). The step between paper and card is ~4% of ink. That
   tonal lift is most of what people perceive as a "light gradient."
6. **Interaction changes the shadow, not the color.** Hover lifts, press sinks,
   focus adds a halo. Backgrounds mostly stay put.

## Tokens

Two inputs. Everything else is derived with `color-mix()`.

```css
:root {
  /* Inputs */
  --ink: #171014;      /* darkest text; also tints every shadow */
  --accent: #ff29a7;   /* primary action, selection, focus */

  /* Materials */
  --paper: color-mix(in srgb, var(--ink) 4%, white);
  --card:  #fff;
  --well:  color-mix(in srgb, var(--accent) 1%, var(--paper));

  /* Text weights */
  --ink-2: color-mix(in srgb, var(--ink) 62%, white);   /* secondary */
  --ink-3: color-mix(in srgb, var(--ink) 45%, white);   /* labels */
  --ink-4: color-mix(in srgb, var(--ink) 30%, white);   /* metadata, placeholder */

  /* Lines, for rules between regions only */
  --hairline: color-mix(in srgb, var(--ink) 7%, transparent);

  /* The shadow vocabulary */
  --ring: inset 0 0 0 0.5px color-mix(in srgb, var(--ink) 9%, transparent);
  --lit:  inset 0 1px 0 rgb(255 255 255 / 0.9);
  --shadow-1: 0 1px 1.5px color-mix(in srgb, var(--ink) 4.5%, transparent);
  --shadow-2: 0 1px 2px color-mix(in srgb, var(--ink) 5%, transparent),
              0 3px 8px color-mix(in srgb, var(--ink) 3.5%, transparent);
  --shadow-3: 0 2px 6px color-mix(in srgb, var(--ink) 6%, transparent),
              0 12px 32px color-mix(in srgb, var(--ink) 7%, transparent);
  --sunk: inset 0 1px 2px color-mix(in srgb, var(--ink) 5%, transparent),
          inset 0 0 0 0.5px color-mix(in srgb, var(--ink) 8%, transparent);
  --halo: 0 0 0 3px color-mix(in srgb, var(--accent) 13%, transparent);

  --radius: 0px;   /* 0 for sharp; 6px / 9px / 13px (sm / md / lg) for soft */
  --ease-out: cubic-bezier(0.23, 1, 0.32, 1);
}
```

With the default inputs these land within one shade of the original
hand-picked Bitface values. Derived tokens resolve where they're declared, so
to retheme a subtree, redeclare the whole block on that selector rather than
just `--ink`.

## The elevation ladder

Five levels. Every surface sits on exactly one.

| Level | Stack | Use for |
|---|---|---|
| -1 sunk | `var(--sunk)` | inputs, selects, tool groups, preview boxes, a pressed button |
| 0 flat | `var(--ring)` | disabled controls, chips, quiet containers |
| 1 resting | `var(--ring), var(--shadow-1)` | panels, sidebars, main surfaces |
| 1 resting, touchable | `var(--ring), var(--lit), var(--shadow-1)` | buttons |
| 2 lifted | `var(--ring), var(--lit), var(--shadow-2)` | hovered buttons, the selected item in a group |
| 3 floating | `var(--ring), var(--lit), var(--shadow-3)` | toasts, menus, popovers, and the one hero object |

**Order matters.** In `box-shadow`, the first shadow paints on top. Keep the
order ring, lit, drop so the hairline always sits crisply over everything else.

**One thing per screen floats.** Level 3 is for transient overlays and a single
hero object (a canvas, a preview, a document). If two static things are both
at level 3, drop one.

**Don't stack cards on cards.** Something inside a panel goes *down* a level to
a well, not up to another card. Going up inside up is what makes UIs look
cluttered.

## Interaction states

```css
.btn {
  background: var(--card);
  box-shadow: var(--ring), var(--lit), var(--shadow-1);
  transition: transform 140ms var(--ease-out), box-shadow 140ms var(--ease-out),
              background 140ms var(--ease-out);
}
.btn:hover    { box-shadow: var(--ring), var(--lit), var(--shadow-2); }   /* lift */
.btn:active   { box-shadow: var(--sunk); transform: scale(0.97); }        /* sink */
.btn:disabled { box-shadow: var(--ring); opacity: 0.45; transform: none; } /* flatten */
.btn:focus-visible { outline: none; box-shadow: var(--ring), var(--lit), var(--shadow-1), var(--halo); }
```

- **Press flips the direction of depth.** Going from a raised stack to `--sunk`
  on `:active` is the single most tactile thing in the system. Pair it with
  `scale(0.97)`, or `scale(0.94)` on icon-sized targets.
- **Focus appends; it never replaces.** Add `var(--halo)` to the end of
  whatever stack the element already has.
- **Name the transitioned properties.** Never `transition: all`.
- **Guard hover for touch** with `@media (hover: none)` so taps don't leave
  things stuck lifted.

## Colored surfaces

A filled button drops the ring (the color is its own edge) and uses a
lower-alpha catch-light, because a 90% white line on a saturated color looks
like a scratch:

```css
.btn-primary {
  background: var(--accent);
  color: #fff;
  box-shadow: inset 0 0.5px 0 rgb(255 255 255 / 0.34), var(--shadow-2);
}
.btn-primary:hover {
  background: color-mix(in srgb, white 8%, var(--accent));
  box-shadow: inset 0 0.5px 0 rgb(255 255 255 / 0.34), var(--shadow-3);
}
```

Colored buttons start one level higher than neutral ones (shadow-2 at rest)
so they hold their weight against the page.

## Light gradients

Bitface itself uses no background gradients. What reads as gradient is:

- the paper to card tonal step,
- the two-layer drop shadow falling off softly below each surface,
- the lit edge. Note `--lit` is invisible on pure white; it earns its keep on
  any *tinted* surface, which is where it reads as a bevel.

When you want an actual gradient on top of that, keep it barely there:

```css
/* Neutral raised surface: top 0%, bottom 2% ink. */
.btn, .card-soft {
  background: linear-gradient(180deg, var(--card),
              color-mix(in srgb, var(--ink) 2%, var(--card)));
}
/* Colored surface: top 8% lighter than the fill. */
.btn-primary {
  background: linear-gradient(180deg,
              color-mix(in srgb, white 8%, var(--accent)), var(--accent));
}
```

Rules for gradients: always vertical, always lighter on top (light comes from
above, same as `--lit`), never more than ~3% of ink across a neutral surface,
and always paired with `--lit` so the top edge stays crisp. If you can see the
gradient as a gradient, it's too strong.

## Recipes

```css
/* Page */
body { background: var(--paper); color: var(--ink);
       font: 13px/1.45 system-ui, -apple-system, sans-serif;
       -webkit-font-smoothing: antialiased; }

/* Panel: the default container */
.panel { background: var(--card); border-radius: var(--radius); padding: 12px;
         box-shadow: var(--ring), var(--shadow-1); }

/* Input / select: pressed in. Focus lifts it to white and adds the halo. */
input, select, textarea {
  background: var(--well); border: none; border-radius: var(--radius);
  box-shadow: var(--sunk); padding: 5px 8px; font: inherit; color: var(--ink);
  transition: box-shadow 140ms var(--ease-out), background 140ms var(--ease-out);
}
input:focus, select:focus, textarea:focus {
  outline: none; background: var(--card); box-shadow: var(--sunk), var(--halo);
}

/* Segmented control: a well holding flat items; the selected one is a card. */
.segmented { display: flex; gap: 2px; padding: 2px; background: var(--well);
             box-shadow: var(--ring); border-radius: var(--radius); }
.segmented > button { border: none; background: transparent; color: var(--ink-3); }
.segmented > button[aria-pressed="true"] {
  background: var(--card); color: var(--ink); box-shadow: var(--shadow-1), var(--ring);
}

/* Switch: a sunk track with a raised knob. */
.switch .track { width: 26px; height: 15px; background: var(--well); box-shadow: var(--sunk); }
.switch .knob  { width: 12px; height: 12px; background: var(--card);
                 box-shadow: var(--shadow-1), var(--ring); }
.switch input:checked + .track { background: var(--accent); }

/* Toast / menu / popover: the floating level. */
.toast { background: var(--card); border-radius: var(--radius);
         box-shadow: var(--ring), var(--lit), var(--shadow-3); }

/* Hero object: the one place allowed real elevation. */
.hero { box-shadow: var(--ring),
        0 1px 2px color-mix(in srgb, var(--ink) 5%, transparent),
        0 10px 28px color-mix(in srgb, var(--ink) 6%, transparent); }

/* Dividers are the only real borders. */
.region + .region { border-top: 1px solid var(--hairline); }
```

With soft corners, nested radii follow `inner = outer - padding`, so a 13px
panel with 4px padding holds 9px children.

## Don'ts

- Pure black (`rgba(0,0,0,...)`) in any shadow.
- Any single shadow layer above ~8% alpha, or any blur above ~32px.
- A `border` on a raised surface. Use `--ring`.
- A ring *and* a border on the same element.
- A card inside a card. Go down to a well instead.
- Changing background color on hover when a shadow change would do.
- Replacing a stack on focus instead of appending `--halo`.
- Semi-transparent sticky headers. Make them solid, or content bleeds through
  and reads as clipped.

## Checklist

1. Every surface is on exactly one ladder level.
2. Every shadow is ink-tinted; nothing is black.
3. Raised things have no borders; pressed things have inset shadows.
4. Hover lifts, press sinks, focus appends a halo, disabled flattens.
5. At most one static thing floats.
6. Any gradient is vertical, lighter on top, and not noticeable as a gradient.
