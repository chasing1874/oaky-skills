# `xiaohongshu.long_note@1`

Use for an article-like Xiaohongshu knowledge note whose main information is the text and whose images appear at deliberate positions as evidence, explanation, or pacing. Do not use it for a finished image-card carousel or video post.

## Data

```json
{
  "coverTitle": "封面上的短标题",
  "coverImage": { "assetId": "550e8400-e29b-41d4-a716-4466554400aa" },
  "title": "长笔记标题",
  "blocks": [
    {
      "type": "markdown",
      "body": "## 先说结论\n\n这里是第一段正文。"
    },
    {
      "type": "image",
      "image": { "assetId": "550e8400-e29b-41d4-a716-4466554400ab" },
      "caption": "这张截图只用于证明上面的判断"
    },
    {
      "type": "markdown",
      "body": "## 为什么\n\n继续展开论证。\n\n#AI创作 #产品设计"
    }
  ]
}
```

- `coverTitle`: up to 120 characters; used for a deliberate text cover when `coverImage` is absent.
- `coverImage`: optional current-space image asset ref. Set it explicitly; inline images never become the cover automatically.
- `title`: up to 240 characters and required for readiness.
- `blocks`: up to 80 ordered blocks. Include at least one non-empty `markdown` block. A `markdown` block has `body` up to 20,000 characters. An `image` block has one current-space image ref and an optional caption up to 240 characters.
- Inline image asset ids must be unique and cannot exceed 20. The cover may intentionally reuse an inline image.
- Put relevant `#hashtags` directly in a markdown block body as ordinary text. Do not send a separate `topics` or `hashtags` field.

## Creative blueprint gate

Before importing or generating final media or writing the document, present one recommended long-note blueprint and ask for approval once. Include:

1. the audience, core claim, and why this should be a long note instead of an image carousel;
2. the cover concept;
3. every planned section with its heading, main copy, and transition;
4. the exact position and role of every source image: evidence, explanation, comparison, or pacing.

Do not convert the article into arbitrary card pages. Do not add a screenshot merely because it is available. If a screenshot supports a specific claim, place it immediately after that section and explain its role in the caption. A material user correction replaces the prior blueprint and requires approval again. Skip the gate only under the explicit delegation and reuse conditions in the main workflow.

## Writing guidance

- Lead with a useful claim or concrete scene, then earn the longer read through clear section progression.
- Keep paragraphs readable on mobile, but preserve full reasoning instead of chopping every sentence into a card slogan.
- Use headings only when they help scanning. Avoid repetitive summary sections and generic engagement bait.
- Keep inline images semantically attached to the text they support. Prefer no image over a decorative or misleading one.
- Add only relevant topics. Do not fill the limit.

## Write

```json
{
  "action": "create",
  "formatId": "xiaohongshu.long_note",
  "schemaVersion": 1,
  "title": "小红书长笔记",
  "publicationTargets": [
    { "targetId": "xiaohongshu.long_note", "config": {} }
  ],
  "data": {
    "coverTitle": "封面上的短标题",
    "title": "长笔记标题",
    "blocks": [
      {
        "type": "markdown",
        "body": "## 先说结论\n\n这里是完整正文。\n\n#AI创作 #产品设计"
      }
    ]
  }
}
```

Submit `data` as the complete native JSON object shown above, never as quoted or stringified JSON. Omit `coverImage` when it is unknown; never invent an `assetId`. On replace, also send `itemId` and `expectedRevision`.
