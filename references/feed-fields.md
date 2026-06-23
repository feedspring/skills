# FeedSpring Feed Fields

Use this reference when generating feed-specific FeedSpring markup. These fields are based on the current attributes scripts project.

## Instagram

Post fields:

| Field | Element notes |
| --- | --- |
| `img` | Use on `<img>` |
| `caption` | Text/HTML content |
| `link` | Use on `<a>` |
| `avatar` | Use on `<img>` |
| `username` | Text |
| `like-count` | Compact number by default |
| `comment-count` | Compact number by default |
| `timestamp` | Text |

Profile fields:

| Field | Element notes |
| --- | --- |
| `avatar` | Use on `<img>` |
| `name` | Text |
| `username` | Text |
| `bio` | Text |
| `following-count` | Compact number by default |
| `follower-count` | Compact number by default |

## Google Reviews

Post fields:

| Field | Element notes |
| --- | --- |
| `rating` | Numeric rating text |
| `rating-string` | Rating label text |
| `name` | Reviewer name |
| `avatar` | Reviewer image |
| `star` | Clone visual active star elements |
| `star-inactive` | Clone visual inactive star elements |
| `review` | Review text |
| `link` | Use on `<a>`; links to the Google profile |
| `timestamp` | Text |

Profile fields:

| Field | Element notes |
| --- | --- |
| `average-rating` | Overall rating |
| `total` | Total review count |

Common mistake: use `review`, not `caption`, for Google review text.

## TikTok

Post fields:

| Field | Element notes |
| --- | --- |
| `link` | Use on `<a>` |
| `video` | Use on `<iframe>` |
| `img` | Use on `<img>`; TikTok thumbnail image |
| `title` | Text/HTML content |
| `description` | Text/HTML content |
| `share-count` | Plain number |
| `duration` | Text |
| `timestamp` | Text |
| `comment-count` | Compact number by default |
| `view-count` | Compact number by default |
| `like-count` | Compact number by default |

Profile fields:

| Field | Element notes |
| --- | --- |
| `avatar` | Use on `<img>` |
| `name` | Display name |
| `bio` | Text/HTML content |
| `profile-link` | Use on `<a>` |
| `verified` | Remove element if not verified |
| `following-count` | Compact number by default |
| `total-likes` | Compact number by default |
| `follower-count` | Compact number by default |

Common mistake: use `title` or `description`, not `caption`, for TikTok text.

## Dribbble

Post fields:

| Field | Element notes |
| --- | --- |
| `img` | Use on `<img>` |
| `link` | Use on `<a>` |
| `title` | Shot title |
| `tag` | Clones tags |
| `timestamp` | Text |

Profile fields:

| Field | Element notes |
| --- | --- |
| `avatar` | Use on `<img>` |
| `bio` | Text/HTML content |
| `name` | Text |
| `profile-link` | Use on `<a>` |
| `location` | Text |
| `followers` | Compact number by default |

Common mistake: use `title`, not `caption`, for Dribbble shot titles.

## Field Options

Timestamp fields support `feed-timestamp`.

Examples:

```html
<span feed-field="timestamp" feed-timestamp="MMMM D, YYYY"></span>
<span feed-field="timestamp" feed-timestamp="from-now"></span>
```

Number fields are compact by default. Use `feed-options="format:plain"` for plain numbers when supported by the number field.

```html
<span feed-field="like-count" feed-options="format:plain"></span>
```

## Element Safety

Some fields only support specific elements:

- `img` and `avatar` should be placed on `<img>`.
- `link` and profile links should be placed on `<a>`.
- TikTok `video` should be placed on `<iframe>`.

If the wrong element is used, FeedSpring may log a warning and skip the field.
