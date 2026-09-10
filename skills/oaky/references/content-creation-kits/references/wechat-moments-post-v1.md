# `wechat_moments.post@1`

Use for a WeChat Moments post made of text, images, or both.

## Data

```json
{
  "body": "朋友圈正文",
  "images": []
}
```

- `body`: up to 20,000 characters.
- `images`: 0-9 ordered, unique current-space image asset refs.
- At least one of `body` or `images` must contain content for readiness.
- Image order is publication order. Do not add layout fields: the client derives the grid from image count.

## Grid-aware selection

- 1 image: choose the strongest standalone visual; the preview preserves a natural single-image ratio.
- 2 images: make the pair coherent.
- 3 images: use a balanced three-column sequence.
- 4 images: design for a 2x2 story.
- 5-9 images: order for a three-column grid; put the strongest image first.

The layout is a client concern. Never crop, split, duplicate, or reorder assets merely to reach nine images unless the user explicitly asks for that treatment.

## Writing guidance

- Match the user's real social voice. Moments often reads more personal and conversational than a platform article.
- For ongoing personal-brand or sales use, make one clear point or call to action without pretending engagement, testimonials, or results.
- Avoid Xiaohongshu-only fields such as `title`, `coverTitle`, or `topics`.

## Write

```json
{
  "action": "create",
  "formatId": "wechat_moments.post",
  "schemaVersion": 1,
  "publicationTargets": [
    { "targetId": "wechat_moments.post", "config": {} }
  ],
  "data": {
    "body": "朋友圈正文",
    "images": []
  }
}
```

Submit the complete `data` object directly. On replace, also send `itemId` and `expectedRevision`. Never invent an `assetId`; obtain it from image generation or the target file query adapter.
