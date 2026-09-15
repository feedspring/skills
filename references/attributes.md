# FeedSpring Attributes — full reference

The complete attribute model. `SKILL.md` covers the common case; read this for rendering modes, loading states, and the behaviours that cause silent failures.

## Structural attributes

| Attribute | Placed on | Purpose |
|---|---|---|
| `feedspring="FEED_ID"` | Wrapper element | Declares the feed container and which feed to render |
| `feedspring="post"` | Repeating element inside the wrapper | The post template |
| `feed-field="item"` | Same as above | Alias for `feedspring="post"` |
| `feedspring="loading"` | Element inside the wrapper | Placeholder, removed once rendering starts |
| `feed-field="FIELD"` | Any element | Where a value is injected |

A `feed-field` inside the wrapper but outside any post template is a **profile-level** field.

Multiple wrappers per page are fine, including different sources on the same page — load each source's script.

## Rendering modes

### `render:dynamic`

Clones one post template per item. Right for grids, sliders, lists — most feeds.

```html
<section feedspring="inst_..." feed-options="render:dynamic|limit:6">
  <article feedspring="post">
    <img feed-field="img" alt="" />
    <p feed-field="caption"></p>
  </article>
</section>
```

Use exactly one post template in this mode.

### `render:static` (default)

Fills post templates already in the DOM, in document order. Use when slots differ — hero plus grid, magazine layouts.

```html
<section feedspring="inst_...">
  <article feedspring="post" class="hero">
    <img feed-field="img" alt="" />
    <h2 feed-field="caption"></h2>
  </article>
  <article feedspring="post" class="small"><img feed-field="img" alt="" /></article>
  <article feedspring="post" class="small"><img feed-field="img" alt="" /></article>
</section>
```

Renders as many posts as there are templates.

## `feed-options`

Pipe-delimited `name:value` pairs on the wrapper. Order doesn't matter.

| Option | Effect |
|---|---|
| `render:dynamic` | Clone one template per item |
| `render:static` | Fill existing templates by index (default) |
| `limit:N` | Maximum number of items displayed (`0` = no limit) |
| `skip:N` | Skip the first N items in the feed |
| `lang:xx-XX` | Locale for compact number formatting |
| `lang:auto` | Use the visitor's browser locale |

`skip` applies first, then `limit` caps the count. `skip:2|limit:4` displays 4 items starting from the 3rd. Both apply in static and dynamic rendering; in static mode `limit` caps how many of the placed templates are filled.

**Unknown option names are silently ignored.** A typo produces no error and no effect — check spelling before blaming the feed.

## Post-level options

Placed on the `feedspring="post"` element, not the wrapper.

| Option | Effect |
|---|---|
| `appear:display-none` | Default. Clears an inline `display:none` when rendering |
| `appear:display-block` | Forces `display: block` after render |
| `appear:display-flex` | Forces `display: flex` after render |

The default exists so you can hide the template with `style="display:none"` and avoid an empty card flashing before data arrives:

```html
<article feedspring="post" style="display:none">
  <img feed-field="img" alt="" />
</article>
```

Use `appear:display-flex` when the template relies on flexbox.

## Loading state

Anything inside `feedspring="loading"` is removed once rendering starts. Style it however you like — a spinner, a shimmer, a message.

```html
<div feedspring="inst_..." feed-options="render:dynamic">
  <div feedspring="loading"><p>Loading posts…</p></div>
  <article feedspring="post"><img feed-field="img" alt="" /></article>
</div>
```

## Modifier attributes

### `feed-timestamp`

On an element with `feed-field="timestamp"`:

- `from-now` — "2 days ago". Handles empty timestamps gracefully
- Any Day.js format string — `MMMM D, YYYY`, `DD/MM/YY`, `HH:mm`
- Omitted — defaults to `MMMM D, YYYY`

## Gotchas

- **Wrong element type means the field is skipped.** Links need `<a>`, images need `<img>`, TikTok video needs `<iframe>`.
- **Feed ID prefix must match the loaded script.** A `google_` ID with the Instagram script renders nothing.
- **Some fields render HTML, not text** — Instagram `caption`, TikTok `title`/`description`/`name`/`bio`, Dribbble `bio`. Relevant if sanitising user content.
- **Instagram `timestamp` can be an empty string.** Use `feed-timestamp="from-now"` or let the element collapse.
- **`verified` (TikTok) is a gate, not a value.** The element is kept if verified, removed if not.
- **Stars (Google) are repeaters, not values.** Provide `star` and `star-inactive` templates.
- **Follower count naming differs.** `follower-count` on Instagram and TikTok, `followers` on Dribbble.
- **`bg` is Instagram only** and sets `background-image` rather than `src`.

## Hybrid layouts

Two wrappers pointing at the same feed — one featured item, then the rest:

```html
<div feedspring="inst_YOUR-FEED-ID">
  <article feedspring="post" class="hero"><img feed-field="img" alt="" /></article>
</div>

<div feedspring="inst_YOUR-FEED-ID" feed-options="render:dynamic|skip:1|limit:6">
  <article feedspring="post" class="small"><img feed-field="img" alt="" /></article>
</div>
```

## Dashboard-side filtering

Some filtering happens on the feed itself, not in markup, and applies to every delivery method:

- **Google Reviews** — minimum star rating, and keyword filtering
- **Instagram, TikTok, Dribbble** — not currently available; use `limit` and `skip`

If a user wants only 5-star reviews, point them at the dashboard filter rather than trying to do it in markup — attributes can't filter by value.
