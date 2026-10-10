---
description: "The official places Spatial speaks from, and the design system that shapes how it looks: mark, colours, typefaces, surfaces, imagery and motion."
---

# Brand, design and channels

## Mark

<figure><img width="200" alt="Spatial's mark: a clear glass sphere containing layered teal, aqua and pale blue folds" src="../.gitbook/assets/logo-mark.png"><figcaption>Spatial's mark on the light icon plate</figcaption></figure>

The mark is a clear glass sphere. Inside it, layered folds in teal, aqua and pale blue sit over one another. The colour gets deeper where folds overlap, and a pale highlight runs over the top. The mark is always shown in full colour on a transparent background, and it holds up at sizes as small as 16 px.

Use it according to these rules:

1. Place it on pale grounds only, meaning Canvas, White or Haze. For app icons and favicons it sits on a light plate graded from near-white down to Ice, with a single soft teal shadow beneath the sphere. Any other background is off limits.
2. Do not recolour it, outline it, flatten it into one colour or add effects. The single soft shadow under it is the only effect allowed.
3. Leave clear space around it equal to at least half its width.

## Colour

There is one accent, Lagoon. Grounds and text come from a family of cool, faintly blue-green neutrals. Four more hues exist only for atmosphere (washes, rings and the sky), and three semantic colours carry meaning: caution, health and loss.

**Grounds**

| Token | Hex | Where it goes |
| --- | --- | --- |
| Canvas | `#f4f9fa` | The page itself, which is never pure white |
| White | `#ffffff` | Cards and raised surfaces |
| Haze | `#edf5f7` | Tinted panels and blocks nested inside cards |
| Haze 2 | `#e2eef1` | A deeper tint where two levels are needed |
| Line | `#dbe8eb` | Borders, rules and dividers |

**Text**

| Token | Hex | Where it goes |
| --- | --- | --- |
| Navy | `#0b2530` | Headings and primary text |
| Tide | `#46626b` | Secondary text |
| Fog | `#87a0a8` | Captions, table heads and labels |
| Pale | `#b7cacf` | The quietest text, counters and placeholders |

**Accent**

| Token | Value | Where it goes |
| --- | --- | --- |
| Lagoon | `#0a8782` | Primary buttons, links, active states and accented figures |
| Lagoon, deep | `#076b67` | Pressed states and accent text such as overlines |
| Lagoon at 40% | `rgb(10 135 130 / 0.4)` | Muted indicators |
| Lagoon at 16% | `rgb(10 135 130 / 0.16)` | Edges of active or hovered cards |
| Lagoon at 10% | `rgb(10 135 130 / 0.1)` | Tag and badge fills, focus rings on inputs |
| Lagoon at 5% | `rgb(10 135 130 / 0.05)` | The faintest accented surfaces |

**Atmosphere only**

| Token | Hex | Where it goes |
| --- | --- | --- |
| Aqua | `#14b7ad` | The turning arc in the rings, overline dots, the far end of accent gradients |
| Mint | `#96e0de` | Soft washes behind hero and closing sections |
| Ice | `#dbf0fa` | Washes, and the lower end of the icon plate |
| Sky | `#0f84b4` | Reserved for sky-toned moments around the hero |

None of these four is used for text, buttons or data.

**Semantic**

| Token | Hex | Deep (for text) | Meaning |
| --- | --- | --- | --- |
| Amber | `#e3922f` | `#a95f12` | Warnings and margin calls |
| Leaf | `#2e9d5f` | `#1d7a45` | Healthy figures and live status |
| Rose | `#dc4f5a` | `#b6343f` | Losses, liquidation and destructive actions |

The primary button has a Lagoon glow beneath it. Gradients appear in exactly three places: the soft Mint and Ice washes, which drift under a fine grain; a single highlighted phrase within a display heading, which runs from Lagoon into Aqua; and the thin accent bar built from the same colours.

## Typefaces

| Family | Used for |
| --- | --- |
| Bricolage Grotesque, weight 600, tightly tracked | Display: the wordmark, headings for pages and sections, and big figures |
| Geist | All other text: body copy, smaller headings (weight 600), labels and buttons, tables, plus all numbers, addresses and hashes, set with tabular figures |

Overlines use Geist at 12 px and weight 600 in Lagoon deep, usually inside a small white pill, led by an Aqua dot. Table heads and figure labels are Geist at 11 px, uppercase with open tracking, in Fog. Those captions are the only text set in capitals.

## Surfaces and shape

Buttons are pills, fully rounded at every size. Cards are white, rounded to 24 px, edged with a Line hairline and lifted by one long, very soft shadow that deepens slightly on hover. Nested blocks inside cards use Haze at 16 px, icon tiles 14 px and inputs 12 px. The header is frosted glass: translucent white with a strong blur, so the page shows through as it scrolls. Chips that sit on the hero photograph use a lighter, clearer pane of the same glass.

## Imagery and ornament

The hero is a photograph of a bright blue sky with clouds, under a gentle veil that keeps white type legible. Away from the hero, three ornaments do the work. Soft Mint and Ice washes drift behind sections with a fine grain laid over them. Sections that read like instruments get a faint grid of dots that fades out before the edges. And thin Lagoon rings, drawn around a common centre to echo the sphere, carry one Aqua arc with a small bead that turns slowly. Product screens and the mark itself complete the set. People never appear in photographs, and the system uses neither clip art nor illustration.

## Motion

Washes drift slowly. Some elements float a few pixels up and down. Content rises gently into place as it appears, and progress bars fill from left to right. The ring arc takes a full minute to turn once. Nothing bounces and nothing flashes.

## Writing the name

Spatial is one word with a single capital, the S. In running text it is never set in capitals, and the wordmark uses the same form, set in Bricolage Grotesque.

## Where Spatial speaks
| Channel | Address |
| --- | --- |
| Website | [onspatial.org](https://onspatial.org) |
| Documentation | [docs.onspatial.org](https://docs.onspatial.org) |
| X | [@onspatial](https://x.com/onspatial) |
| GitHub | [github.com/onspatial](https://github.com/onspatial) |
| Email for everything else | hello@onspatial.org |
| Reporting a vulnerability | security@onspatial.org |

Any other domain, handle or address is not Spatial, whatever it says about itself. Contract addresses appear in exactly two places, the website and these docs.
