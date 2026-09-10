# `generic.image_post@1`

Use for one reusable image-and-text post whose copy and ordered image set can be handed off to more than one compatible platform. The first supported publication targets are Xiaohongshu and WeChat Moments. Keep an explicitly platform-only draft in its platform-specific Format.

## Data

```json
{
  "title": "图文标题",
  "body": "完整图文正文\n\n#话题一 #话题二",
  "images": []
}
```

- `title`: up to 240 characters and required for readiness.
- `body`: up to 20,000 characters and required for readiness.
- Write relevant `#hashtags` directly in `body` as ordinary text. Do not send a separate `topics` or `hashtags` field.
- `images`: ordered, unique current-space image asset refs. The first image is the cover. This Format does not impose an image-count limit; publication targets may enforce their own limits.

## Publication targets

- Compatible targets: `xiaohongshu.image_post` and `wechat_moments.post`.
- When `xiaohongshu.image_post` is selected, `body` must be at most 1,000 characters. Draft with margin; if the Registry returns `publication_target_body_too_long` at `data.body`, shorten it and retry `manageCreationItem` in the same task.
- Use only destinations the user explicitly requested. Both may be selected for the same document.
- Omission or `[]` keeps the document neutral; it does not guess a platform.
- Platform handoff may apply target-specific copying or launch behavior, while the stored content remains one generic document.

## Write

```json
{
  "action": "create",
  "formatId": "generic.image_post",
  "schemaVersion": 1,
  "title": "跨平台图文",
  "publicationTargets": [
    { "targetId": "xiaohongshu.image_post", "config": {} },
    { "targetId": "wechat_moments.post", "config": {} }
  ],
  "data": {
    "title": "图文标题",
    "body": "完整图文正文\n\n#话题一 #话题二",
    "images": []
  }
}
```

Submit `data` as the complete native JSON object shown above, never as quoted or stringified JSON. On replace, also send `itemId` and `expectedRevision`. Never invent an `assetId`; obtain it from image generation or the target file query adapter.
