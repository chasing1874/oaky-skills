# `xiaohongshu.image_post@1`

Use for a Xiaohongshu carousel whose ordered `images` are finished publishable pages or photos. Do not use it for video posts or article-like knowledge content with source images embedded between text sections; use `xiaohongshu.long_note@1` for that mode.

## Data

```json
{
  "coverTitle": "封面上的短标题",
  "title": "笔记标题",
  "body": "完整笔记正文\n\n#话题一 #话题二",
  "images": []
}
```

- `coverTitle`: up to 120 characters; used as the text-cover fallback when there is no image.
- `title`: up to 240 characters and required for readiness.
- `body`: at most 1,000 characters and required for readiness. Draft with some margin instead of relying on exact mental counting. The Registry hard-rejects a longer body with an AI-repairable `invalid_format_data` result at `data.body`; shorten it and retry `manageCreationItem` in the same task.
- Write relevant `#hashtags` directly in `body` as ordinary text; they count toward its length. Do not send a separate `topics` or `hashtags` field.
- `images`: 0-9 ordered, unique current-space image asset refs. Each item is a final carousel page or publishable photo, and the first image is the deliberate cover.

## Creative blueprint gate

Before generating images or writing the final document, present one recommended page blueprint and ask for approval once. For every page, state its job, proposed on-page copy, visual or source material, and information density. Identify the cover explicitly.

- Treat a raw screenshot, chart, source image, or illustration as supporting material, not as a carousel page by default. Composite it into a designed final page when it is evidence inside a knowledge card.
- Never place an unapproved raw screenshot first; the first `images` item becomes the published cover.
- If the user changes the content mode, page sequence, copy, or role of a source image, update the blueprint and ask again before final generation.
- Skip the gate only when the user explicitly delegates the choices, provides the exact pages, approves the current blueprint, or asks to reuse an approved template.

## Writing guidance

- Make the title concrete and useful; keep the opening able to stand on its own before expansion.
- Use readable short paragraphs. Preserve the user's voice instead of forcing a fixed viral template.
- Add only relevant topics. Do not fill the limit.
- Prefer real images already present in the current space when they are publishable photos. For knowledge cards, design the page around the information; do not let an available image dictate the page structure.

## Write

```json
{
  "action": "create",
  "formatId": "xiaohongshu.image_post",
  "schemaVersion": 1,
  "title": "小红书笔记",
  "publicationTargets": [
    { "targetId": "xiaohongshu.image_post", "config": {} }
  ],
  "data": {
    "coverTitle": "封面上的短标题",
    "title": "笔记标题",
    "body": "完整笔记正文\n\n#话题一 #话题二",
    "images": []
  }
}
```

Submit `data` as the complete native JSON object shown above, never as quoted or stringified JSON. On replace, also send `itemId` and `expectedRevision`. Never invent an `assetId`; obtain it from image generation or the target file query adapter.
