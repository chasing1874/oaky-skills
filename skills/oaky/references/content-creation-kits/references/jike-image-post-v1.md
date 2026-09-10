# 即刻图文 `jike.image_post@1`

Load this reference only for a single 即刻 image post. Do not use it for video, reposts, comments, external article cards, or platform publishing.

## Contract

```json
{
  "action": "create",
  "formatId": "jike.image_post",
  "schemaVersion": 1,
  "publicationTargets": [
    { "targetId": "jike.image_post", "config": {} }
  ],
  "data": {
    "body": "动态正文",
    "topic": "产品经理的日常",
    "images": []
  }
}
```

- Send a complete `data` object. Do not add title, author, metrics, publish time, platform ids, crop coordinates, image URLs, or layout fields.
- Keep `images` ordered and unique. Use 0-9 current-space image asset ids.
- The body or at least one image must be present.
- `topic` is the human-readable 即刻圈子 name. Use `""` when no circle can be chosen confidently; never invent a platform topic id.

## Writing and topic choice

Write like a real person sharing an observation, experience, small discovery, question, or work-in-progress. Keep the useful detail and personal judgment; avoid promotional slogans, artificial sectioning, and a pile of generic hashtags.

Choose one topic only when it clearly matches the post. Prefer a specific community context over a broad traffic label. A missing topic is a repairable warning, not permission to fabricate one.

## Image order

- 1 image: use the main scene or result and preserve its natural ratio.
- 2-3 images: order them as a short horizontal sequence.
- 4 images: plan a 2x2 reading order.
- 5-9 images: use a three-column sequence; put the strongest context-setting image first.

Do not encode grid decisions in JSON. The renderer derives layout from image count.

## Workflow

1. Identify the concrete moment, opinion, or useful discovery worth sharing.
2. Select a relevant topic name when the context supports it.
3. Reuse existing image assets or generate the required images before writing their asset ids.
4. Submit this complete envelope through the target Creation Item manage adapter; keep `data` as a native object rather than stringified JSON.
5. If validation fails, submit `{"action":"describe_format","formatId":"jike.image_post"}` through the target Creation Item query adapter, repair the returned path, and retry.
