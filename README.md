# Redmine Dark Static

A standalone dark theme for **Redmine 7.x**.

Dark mode is active only when this theme is selected in **Administration →
Settings → Display → Theme**. Nothing is injected into other themes, no plugin
is registered, no cookie is set.

## How it works

Redmine 7 rewrote its stylesheets on top of the [Open Color](https://yeun.github.io/open-color/)
palette, exposed as `--oc-*` custom properties. Across `application.css`,
`dropdown.css`, `responsive.css`, `context_menu.css`, `gantt.css`, `scm.css`
and `jstoolbar.css` there are ~440 references to those variables and only a
handful of hardcoded colours.

So this theme **re-maps the palette semantically** rather than filtering pixels:

* levels `0–2` become dark surfaces (they are the light theme's near-whites),
* levels `3–4` become borders,
* levels `5–9` become text of increasing emphasis,
* `--oc-white` becomes the base surface.

Every sheet that consumes those variables is themed at once, including the
per-page ones. A short list of explicit overrides then handles the cases the
palette cannot express — see below.

Only the levels that actually need to move are overridden. The saturated
mid-tones — the avatar palette, the markdown alert stripes, the current-item
marker, the revision-graph fills — already read correctly on a dark page and
are left at their Open Color defaults rather than being restated.

Typography is untouched: the default theme's Noto Sans stack and all font
sizes are inherited as-is.

One deliberate inversion of upstream's intent: where the light theme treats
`--oc-gray-0` as *recessed* (sidebar, `.box`, `pre`, alternating rows sit
slightly darker than white content), this theme makes it slightly *lighter*
than the base surface, which is the usual convention for elevation on dark
backgrounds.

Upstream also fills the issue-detail panel with `--oc-yellow-0`. That reads as
a muddy tinted block on a dark page, so the fill is dropped and the panel sits
on the page background with only a hairline separating it from the history
below.

## Problems this fixes

The previous version of this theme applied `filter: invert(90%)` to `<body>`
and patched the fallout. That approach caused the following, all of which are
resolved:

| Symptom | Cause |
| --- | --- |
| Links were the same white as body text | The theme forced `a { color: rgb(17,17,17) }`, which inverted to roughly the same value as inverted body text |
| Icons were sometimes yellow, sometimes blue | `.icon { filter: invert(100%) }` nested inside `body { filter: invert(90%) }`. The two filters do not cancel, and Redmine 7 strokes action icons with `--oc-blue-9`, which inverts to orange/yellow. Icons the selector missed kept the single body inversion and stayed blue |
| Avatars, gravatars and attached images looked wrong | They were double-inverted to compensate for the body filter |
| Dead rules | `#top-menu` (Redmine 7 uses `nav.top-menu`) and the `.theme-Rtmaterial` block, which targets an unrelated theme |

Action icons are now a single consistent blue (`#40c4ff`), amber on hover, in
the admin panel and everywhere else — the same rule Redmine itself uses.

## Colour semantics

Upstream's red ramp does two unrelated jobs: interaction (link and icon hover)
and real failure (flash errors, field validation). This theme separates them,
which gives four unambiguous signals:

| Meaning | Colour | Used for |
| --- | --- | --- |
| Navigate | azure `#40c4ff` | links, action icons at rest |
| Attention | amber `#ffc107` | hover, overdue dates, required-field `*`, gantt "late" bars, missing wiki pages, the "private" badge, warnings, diff removals |
| Failure | rose `#ff8fa3` | flash errors, `#errorExplanation`, field validation, `.icon-error`, jQuery UI error state |
| Success | green `#3fdd85` | notice flashes, closed status, diff additions |

Red is gone entirely. The required-field asterisk in particular is amber rather
than red — it marks a field, it does not report a problem.

The amber and the azure sit roughly 154° apart in hue, so the two colours you
see most — a link and the icon beside it going amber under the cursor — read as
a deliberate complementary pair. The amber is exposed as `--rm-amber` and the
warning and "late" ramps derive from it, so retuning it is a one-line change.

Flash messages carry a 3px leading edge in their own colour, which is what
makes the state readable at a glance rather than requiring you to read the icon.
The rose is kept close to the amber in lightness so a validation error never
reads as *less* urgent than an overdue date.

Two side effects worth noting:

* Diffs are now **amber removals against green additions** instead of red
  against green, which is easier to read with deuteranopia — the most common
  form of colour blindness — while staying a clear two-colour pairing.
* Upstream tints warning flashes with `--oc-pink-9`, i.e. pink text on a yellow
  background. Because pink is now reserved for failures, warnings get amber
  text on the amber background instead, so warning and error no longer share a
  hue.

## Additional issues found and fixed

These are not caused by the old theme; they are places where a Redmine 7 dark
theme has to intervene, found by auditing every colour declaration in the
Redmine 7.0-stable stylesheets:

* **jQuery UI 1.13.2** is vendored with hardcoded light colours
  (`#fff`, `#333`, `#f6f6f6`). Without overrides the **date picker renders as a
  white box**. Its icon sprites are dark PNGs and are inverted.
* **Syntax highlighting** ships Rouge's light `colorful` theme — dark inks on
  `#fafafa`, plus a pink `#fff0f0` fill behind every string literal. Re-mapped
  to a dark scheme.
* **`#header`** is painted with the literal `#3A78A3` and **`nav.top-menu`**
  with `#234761`; both also use light `--oc-*` values as *foreground*. Both
  bars are repainted and their text restored.
* **`--oc-white` is dual-purpose** — 24 background uses but also 14 foreground
  uses (header text, top-menu icon strokes, badge text, avatar initials,
  calendar "today"). Each foreground case is restored explicitly.
* **Gantt "todo" milestones** are drawn in `--oc-white` and would have become
  invisible.
* **Context menu** (`context_menu.css`) uses a literal `background: white` plus
  `border: 1px solid white` per item — the latter would have become a visible
  white grid on a dark menu.
* **`search.svg`** is stroked `currentColor`, which resolves to black when
  loaded via `url()`, so the autocomplete search glyph was invisible. Replaced
  with an inline data URI, as are the dark rasters `external.png` and
  `arrow_up.png`.
* **Blame stripes** (`table.annotate`) encode revisions using level-1/2/3
  colour tints as border colours; darkening those levels for flash messages
  would have erased the colour coding, so the twelve stripes are set explicitly.
* **Inline `<code>`** is tinted with `rgba(var(--oc-gray-9-rgb), …)`.
  `--oc-gray-9-rgb` is deliberately kept dark so box-shadows read as depth
  rather than glow, so that one rule is overridden separately.
* **Mobile chrome** (`responsive.css`, ≤899px) uses the literals `#628db6`,
  `#3e5b76` and `#506a83`.
* Page-specific sheets load *after* `heads_for_theme`, so overrides that target
  them are `body`-prefixed to outrank them without `!important`.
* **`redmine_spent_time`** hardcodes three colours that only work on white: the
  under-8h day total is literal `red` (2.3:1 on `#14181d`), the over-8h total is
  literal `blue` (1.4:1), and `.weekend_row` fills with `#ffdfe6`, a light pink
  that keeps the theme's light text. Remapped to `--rm-danger`, `--oc-blue-9`
  and `--oc-gray-2`. Plugin sheets also load after the theme, so these are
  `body`-prefixed.

## Contrast

Checked against WCAG 2.1 on the base surface `#14181d`:

| | ratio |
| --- | --- |
| Body text | 14.5:1 |
| Headings / box text | 8.8:1 |
| Secondary text | 6.0:1 |
| De-emphasised text | 4.6:1 |
| Links and action icons | 9.0:1 |
| Link hover (amber) | 10.9:1 |
| Error text | 8.2:1 |
| Flash messages | 7.8–9.9:1 |

Structural borders stay intentionally quiet (`--oc-gray-4`); interactive
controls use a separate, brighter border that clears 3:1.

Repository statistics use Chart.js with bar colours set in JavaScript. The
canvas is left alone — inverting it, as the old theme did, corrupted those
colours. The bars stay accurate; only the default axis labels are dimmer than
ideal.

## Installation

Copy the theme directory into Redmine's root `themes` directory:

```text
/usr/src/redmine/themes/redmine_dark_static
```

For Docker, persist it with a host mount such as:

```yaml
volumes:
  - /srv/redmine/themes:/usr/src/redmine/themes
```

Then recreate Redmine and select `redmine_dark_static` in the administration UI.

## Important

Remove or move the original `/usr/src/redmine/plugins/redmine_dark` plugin.
Keeping it installed will continue loading `dark.css` and `dark.js` into all
other themes, including the default blue theme.

## License

GNU GPL version 2, matching the source plugin. See `LICENSE`.
