# FeedSpring Feed Fields

Every field, for every source. Read this before generating feed-specific markup — guessing field names produces markup that renders nothing, with no error.

The **Attribute** column is the value for `feed-field="..."`. The **JSON key** is the same data in the GraphQL API.

Supported sources: Instagram, Google Reviews, TikTok, Dribbble. There is no YouTube support.

---

## Instagram

Feed ID prefix `inst_` · script `instagram-attrs.js`

### Post fields

| Attribute | JSON key | Notes |
|---|---|---|
| `img` | `mediaUrl` | Use on `<img>` |
| `bg` | `mediaUrl` | Any element; sets `background-image` instead of `src`. Instagram only |
| `link` | `permalink` | Use on `<a>` |
| `caption` | `caption` | Written with `innerHTML` |
| `like-count` | `likeCount` | Compact number (`1.2K`) |
| `comment-count` | `commentCount` | Compact number |
| `timestamp` | `timestamp` | May be an empty string — handle gracefully |

### Profile fields

| Attribute | JSON key | Notes |
|---|---|---|
| `avatar` | `avatar` | Use on `<img>` |
| `name` | `fullName` | Display name |
| `username` | `username` | The @-handle |
| `bio` | `bio` | Text |
| `follower-count` | `followersCount` | Compact number |
| `following-count` | `followingCount` | Compact number |

Profile fields may also be placed inside a post template — they resolve to the same account-level value on every card.

---

## Google Reviews

Feed ID prefix `google_` · script `google-reviews-attrs.js`

### Post fields

| Attribute | JSON key | Notes |
|---|---|---|
| `review` | `comment` | The review text. Plain text, not HTML |
| `name` | `author.name` | Reviewer name |
| `avatar` | `author.photoUrl` | Use on `<img>` |
| `rating` | `rating.number` | Numeric, 1–5 |
| `rating-string` | `rating.string` | English word — "five", "four" |
| `star` | `rating.number` | Repeater: cloned once per rating point |
| `star-inactive` | `rating.number` | Repeater: cloned `5 - rating` times |
| `timestamp` | `createdAt` | — |

### Profile fields

| Attribute | JSON key | Notes |
|---|---|---|
| `average-rating` | `averageRating` | One decimal place |
| `total` | `total` | Total review count |

### Stars

Stars are a repeater, not a value target. Provide both states and FeedSpring clones each the right number of times:

```html
<div feedspring="post">
  <div class="stars">
    <svg feed-field="star" class="star-active">...</svg>
    <svg feed-field="star-inactive" class="star-inactive">...</svg>
  </div>
  <p feed-field="review"></p>
  <span feed-field="name"></span>
</div>
```

A 4-star review renders 4 active stars and 1 inactive.

### There is no per-review link

Google does not expose a URL per review, so there is no `link` field. Link to the business listing instead — visitors can read the other reviews and leave one of their own:

```html
<a href="https://search.google.com/local/reviews?placeid=YOUR_PLACE_ID" target="_blank" rel="noopener">
  Read all reviews on Google
</a>
```

This is a static link the user supplies, not feed data. Ask for their Place ID; don't invent one.

**Common mistake:** use `review`, not `caption`, for review text.

---

## TikTok

Feed ID prefix `tiktok_` · script `tiktok-attrs.js`

### Post fields

| Attribute | JSON key | Notes |
|---|---|---|
| `img` | `coverImageUrl` | Video thumbnail. Use on `<img>` |
| `video` | derived from `id` | Use on `<iframe>`; inline TikTok embed player |
| `link` | `shareUrl` | Use on `<a>` |
| `title` | `title` | Written with `innerHTML` |
| `description` | `description` | Written with `innerHTML` |
| `duration` | `duration` | Raw number of seconds — format it yourself |
| `view-count` | `viewCount` | Compact number |
| `like-count` | `likeCount` | Compact number |
| `comment-count` | `commentCount` | Compact number |
| `share-count` | `shareCount` | Plain number |
| `timestamp` | `createTime` | — |

### Profile fields

| Attribute | JSON key | Notes |
|---|---|---|
| `avatar` | `avatarUrl` | Use on `<img>` |
| `name` | `displayName` | Written with `innerHTML` |
| `bio` | `bio` | Written with `innerHTML` |
| `profile-link` | `url` | Use on `<a>` |
| `follower-count` | `followerCount` | Compact number |
| `following-count` | `followingCount` | Compact number |
| `total-likes` | `likesCount` | Total likes across the account, compact |
| `verified` | `isVerified` | Conditional gate — see below |

### `verified` is a gate, not a value

Place it on any element. If the account is verified the element stays; if not it is removed entirely.

```html
<svg feed-field="verified"><!-- blue checkmark --></svg>
```

### Inline video

`feed-field="video"` on an `<iframe>` sets the src to TikTok's embed player. It pulls in TikTok's embed script, which is heavy — for performance-sensitive pages use `img` plus a `link` instead.

**Common mistakes:** use `title` or `description`, not `caption`. `total-likes` is the account total; `like-count` is per video.

---

## Dribbble

Feed ID prefix `dribbble_` · script `dribbble-attrs.js`

### Post fields

| Attribute | JSON key | Notes |
|---|---|---|
| `img` | `image` | Use on `<img>`. Shots are 4:3 |
| `link` | `url` | Use on `<a>` |
| `title` | `title` | Shot title |
| `timestamp` | `publishedAt` | — |

### Profile fields

| Attribute | JSON key | Notes |
|---|---|---|
| `avatar` | `avatarUrl` | Use on `<img>` |
| `name` | `name` | — |
| `bio` | `bio` | Written with `innerHTML` |
| `location` | `location` | Profile location, not per-shot |
| `profile-link` | `url` | Use on `<a>` |
| `followers` | `followersCount` | Compact number |

**Common mistakes:** use `title`, not `caption`, for shot titles. Follower count is `followers` on Dribbble — Instagram and TikTok use `follower-count`.

Dribbble exposes no like or view counts.

---

## Cross-source

Attributes that mean the same thing everywhere:

| Attribute | Works on |
|---|---|
| `img` | Instagram, TikTok, Dribbble |
| `link` | Instagram, TikTok, Dribbble |
| `timestamp` | All |
| `avatar` | All |
| `name` | All |
| `bio` | Instagram, TikTok, Dribbble |
| `profile-link` | TikTok, Dribbble |
| `like-count` | Instagram, TikTok |
| `comment-count` | Instagram, TikTok |
| `follower-count` | Instagram, TikTok (Dribbble uses `followers`) |
| `following-count` | Instagram, TikTok |

Source-specific only:

| Source | Attributes |
|---|---|
| Instagram | `bg`, `caption`, `username` |
| Google Reviews | `review`, `rating`, `rating-string`, `star`, `star-inactive`, `average-rating`, `total` |
| TikTok | `video`, `description`, `duration`, `view-count`, `share-count`, `total-likes`, `verified` |
| Dribbble | `location`, `followers` |

---

## Timestamps

Add `feed-timestamp` alongside `feed-field="timestamp"`:

```html
<span feed-field="timestamp" feed-timestamp="from-now"></span>
<span feed-field="timestamp" feed-timestamp="MMMM D, YYYY"></span>
```

- `from-now` — relative time, "2 days ago". Handles empty values gracefully
- Any other value is a Day.js format string
- Omitted, the default is `MMMM D, YYYY`

## Element behaviour

The element a `feed-field` sits on decides what happens to the value:

| Element | Behaviour |
|---|---|
| `<img>` | Sets `src`. Removes `srcset` if present |
| `<a>` | Sets `href` |
| `<iframe>` | Sets `src` to the derived embed URL (TikTok `video` only) |
| Anything else | Inserts the value as text or HTML |

Put the field on the right element or it is skipped.

## Fields written with `innerHTML`

HTML in these values is rendered, not escaped:

- Instagram: `caption`
- TikTok: `title`, `description`, `name`, `bio`
- Dribbble: `bio`

Everything else is inserted as plain text.

## Numbers

Counts are formatted with `Intl.NumberFormat` in compact notation, so `1234` renders as `1.2K` in `en-US` and `1,2 k` in `fr-FR`. Control it with `feed-options="lang:en-GB"` or `lang:auto` on the wrapper.

Exceptions that render as plain numbers: TikTok `share-count`, TikTok `duration`, Google `rating` and `total`.
