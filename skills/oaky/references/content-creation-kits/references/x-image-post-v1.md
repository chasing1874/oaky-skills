# X / Twitter 图文 `x.image_post@1`

Load this reference only for a single X/Twitter image post. Do not use it for threads, replies, quotes, polls, GIFs, or video.

## Contract

```json
{
  "action": "create",
  "formatId": "x.image_post",
  "schemaVersion": 1,
  "publicationTargets": [
    { "targetId": "x.image_post", "config": {} }
  ],
  "data": {
    "body": "Post text with #hashtags inline",
    "images": []
  }
}
```

- Send a complete `data` object. Do not add `title`, `topics`, `hashtags`, author, metrics, publish time, reply, quote, or layout fields.
- Keep `images` ordered and unique. Use 0-4 current-space image asset ids.
- The body or at least one image must be present.
- Put mentions, links, and hashtags directly in `body`.

## Standard length

Default to a standard Post of at most 280 weighted characters. Latin text and common punctuation usually count as 1; CJK and emoji usually count as 2; each URL counts as 23; attached images count as 0. The Registry returns `x.image_post.standard_length_exceeded` when the draft needs Premium long-post capability or further compression.

Unless the user explicitly wants a Premium long post, repair that warning by tightening the body without losing its main claim.

## Image order

- 1 image: make it the clear hero visual.
- 2 images: use a pair, comparison, or before/after order.
- 3 images: put the strongest image first; the other two support it.
- 4 images: order them as a coherent 2x2 scan from top-left to bottom-right.

Choose images that add information. Do not create layout metadata; the renderer derives the media grid from image count.

## Workflow

1. Write one standalone idea with the important point early.
2. Keep hashtags sparse and inline; do not add them only to fill space.
3. Reuse existing image assets or generate the required images before writing their asset ids.
4. Submit this complete envelope through the target Creation Item manage adapter; keep `data` as a native object rather than stringified JSON.
5. If validation fails, submit `{"action":"describe_format","formatId":"x.image_post"}` through the target Creation Item query adapter, repair the returned path, and retry.
