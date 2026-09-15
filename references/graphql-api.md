# FeedSpring GraphQL API

Read this before writing a query. Three things are easy to get wrong and all fail loudly: the response is a **union type**, the **collection name differs per source**, and **every collection wraps its items in `nodes`**.

Endpoint: `https://api.feedspring.com/graphql` — POST only, `Content-Type: application/json`.

**No API key.** The Feed ID is the credential, it's public, and it's safe in browser code. Do not invent an `Authorization` header.

Schema SDL: `https://api.feedspring.com/graphql/schema.graphql` · introspection is enabled.

## When to use the API instead of attributes

Attributes are the default for most sites. Reach for the API when the user needs:

- Server-side rendering, or feed content present in the page HTML for SEO — matters most for **Google Reviews**
- Caching or a shared data layer
- To transform, filter or merge feed data before rendering
- A non-browser target: backend, mobile app, pipeline

## The union type

`feed(publicKey:)` returns a union. Always select `__typename` and use an inline fragment for the source:

| Source | GraphQL type | Items are at |
|---|---|---|
| Instagram | `InstagramFeedData` | `posts.nodes` |
| TikTok | `TikTokFeedData` | `videos.nodes` |
| Dribbble | `DribbbleFeedData` | `shots.nodes` |
| Google Reviews | `GoogleReviewsFeedData` | `reviews.nodes` |

## Collections wrap their items in `nodes`

Each collection is a connection object, not a plain list. Select the item fields inside `nodes`:

```graphql
posts { nodes { id caption } }   # correct
posts { id caption }             # fails: Cannot query field "id" on type "InstagramPostConnection"
```

In code, read the array from `nodes`: `feed.posts.nodes`, `feed.reviews.nodes`. The wrapper is there so pagination can be added later.

Only the four top-level collections are wrapped. Instagram's `children` (carousel media) and Dribbble's `tags` are plain lists.

## Images are objects, not strings

Request `image { url(input: { width: 800 }) }` — never `image` on its own. Without `input` you get the source dimensions in WebP.

```graphql
image {
  url(input: { width: 800, height: 600, format: WEBP })
  srcset(input: [
    { key: "small",  width: 480 }
    { key: "medium", width: 768 }
    { key: "large",  width: 1200 }
  ]) { key url }
}
```

Resizing is fit, never cropped — if you set only one dimension the other follows the source aspect ratio. Formats: `WEBP` (default), `JPEG`. Limits: 1–4096px per side, max 3 `srcset` variants, keys 1–32 chars of `A-Za-z0-9_-` and unique.

The image field is named `image` on Instagram and Dribbble, `cover` on TikTok videos, and `photo` on Google review authors.

## Instagram

```graphql
query InstagramFeed($publicKey: String!) {
  feed(publicKey: $publicKey) {
    __typename
    ... on InstagramFeedData {
      profile {
        username
        fullName
        bio
        followerCount
        avatar { url(input: { width: 160, height: 160 }) }
      }
      posts {
        nodes {
          id
          caption
          url
          mediaType
          likeCount
          commentCount
          publishedAt
          image { url(input: { width: 1200 }) }
          children { id mediaType image { url(input: { width: 1200 }) } }
        }
      }
    }
  }
}
```

`profile` is null for hashtag feeds, `hashtag` is null for account feeds. `mediaType` is `IMAGE`, `VIDEO` or `CAROUSEL`. `children` holds carousel media and is `[]` for single posts. `publishedAt` can be null.

## TikTok

```graphql
query TikTokFeed($publicKey: String!) {
  feed(publicKey: $publicKey) {
    __typename
    ... on TikTokFeedData {
      profile {
        displayName
        bio
        url
        isVerified
        followerCount
        likeCount
        avatar { url(input: { width: 160, height: 160 }) }
      }
      videos {
        nodes {
          id
          url
          embedUrl
          title
          description
          viewCount
          likeCount
          commentCount
          shareCount
          durationSeconds
          publishedAt
          cover { url(input: { width: 800 }) }
        }
      }
    }
  }
}
```

**`likeCount` means two different things.** On `profile` it's total likes across the account; on a video it's that video's likes. Label them clearly in the UI.

`embedHtml` is also available but returns raw third-party HTML from TikTok. Prefer `embedUrl` in an `<iframe>`. If you must use `embedHtml` in React it requires `dangerouslySetInnerHTML` — warn the user.

## Dribbble

```graphql
query DribbbleFeed($publicKey: String!) {
  feed(publicKey: $publicKey) {
    __typename
    ... on DribbbleFeedData {
      profile {
        name
        username
        bio
        location
        url
        websiteUrl
        followerCount
        avatar { url(input: { width: 160, height: 160 }) }
      }
      shots {
        nodes {
          id
          url
          title
          tags
          publishedAt
          image { url(input: { width: 1200 }) }
          team { name url }
        }
      }
    }
  }
}
```

`team` is null for shots not published by a team. `tags` is always an array, possibly empty.

## Google Reviews

```graphql
query GoogleReviewsFeed($publicKey: String!) {
  feed(publicKey: $publicKey) {
    __typename
    ... on GoogleReviewsFeedData {
      business { name }
      location { name address placeId }
      reviewCount
      averageRating
      reviews {
        nodes {
          id
          comment
          createdAt
          rating { value label }
          author {
            name
            isAnonymous
            photo { url(input: { width: 128, height: 128 }) }
          }
          reply { comment updatedAt }
        }
      }
    }
  }
}
```

`rating.value` is 1–5. `rating.label` is an English word, not localised. `reply` is null when the business hasn't replied. `location.placeId` is what you need to build a "leave a review" link.

**Star-only reviews with no text are common** — `comment` can be empty, so don't render an empty quote block.

## Limits and errors

The collections currently take **no pagination arguments**. The whole collection comes back; slice it client-side:

```js
const visible = feed.posts.nodes.slice(0, 6)
```

The `nodes` wrapper exists so pagination can be added. Check the schema before assuming arguments are still unavailable.

If the user needs a small number of items out of a large feed, attributes are currently the better tool.

One feed lookup per operation — aliasing a second `feed` call is rejected. Request body max 64 KiB. Overly complex queries are rejected.

Always check `errors`, even on HTTP 200:

```js
const { data, errors } = await res.json()
if (errors) throw new Error(errors[0].message)
```

| Code | Meaning |
|---|---|
| `feed_not_found` | No feed for that Feed ID |
| `feed_not_active` | Feed exists but isn't active |
| `origin_not_allowed` | Request origin isn't in the feed's allow-list |
| `views_limit_reached` | Owner's monthly view allowance is used up |
| `invalid_image_transform` | Bad image transform or srcset input |
| `image_unavailable` | No source for the requested image |
| `request_too_large` | Body over 64 KiB (HTTP 413) |
| `query_too_complex` | Operation too complex (HTTP 422) |
| `internal_error` | Server-side failure |

`views_limit_reached` is worth handling visibly — render a graceful empty state rather than a blank section, so the site owner notices.

## Domain allow-list

Feeds can be restricted to origins set in the dashboard. Browsers send `Origin` automatically; a backend or CLI client must set it explicitly:

```bash
curl --request POST 'https://api.feedspring.com/graphql' \
  --header 'Content-Type: application/json' \
  --header 'Origin: https://example.com' \
  --data '{"query":"...","variables":{"publicKey":"inst_YOUR_FEED_ID"}}'
```

If a feed returns `origin_not_allowed` from a server, this is usually why.

## Next.js example

```jsx
async function getFeed(publicKey) {
  const res = await fetch('https://api.feedspring.com/graphql', {
    method: 'POST',
    headers: { 'Content-Type': 'application/json' },
    body: JSON.stringify({
      query: `
        query Feed($publicKey: String!) {
          feed(publicKey: $publicKey) {
            __typename
            ... on InstagramFeedData {
              posts { nodes { id caption url image { url(input: { width: 800 }) } } }
            }
          }
        }
      `,
      variables: { publicKey },
    }),
  })

  const { data, errors } = await res.json()
  if (errors) throw new Error(errors[0].message)
  return data.feed
}

export default async function InstagramGrid() {
  const feed = await getFeed('inst_YOUR-FEED-ID')

  return (
    <div className="grid">
      {feed.posts.nodes.slice(0, 8).map((post) => (
        <a key={post.id} href={post.url} target="_blank" rel="noopener noreferrer">
          <img src={post.image?.url} alt="" />
          <p>{post.caption}</p>
        </a>
      ))}
    </div>
  )
}
```

Responses are sent with `no-store`. Feeds sync hourly at best, so re-fetching on every request is wasteful — but check current FeedSpring guidance before adding a persistent cache, as caching interacts with plan view allowances.
