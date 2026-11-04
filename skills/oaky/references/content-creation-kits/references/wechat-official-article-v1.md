# 微信公众号文章 `wechat_official.article@1`

Use for a complete long-form article intended to be read in a WeChat Official Account layout.
Do not use it for a WeChat Moments post.

## Data

```json
{
  "title": "文章标题",
  "author": "作者名",
  "digest": "一段可独立阅读的摘要",
  "body": "正文开场。\n\n## 第一部分\n\n正文段落。",
  "coverImage": { "assetId": "550e8400-e29b-41d4-a716-446655440000" },
  "themeId": "raphael-notion",
  "themeVersion": 1,
  "images": []
}
```

- `title`: required for readiness, up to 64 characters.
- `author`: optional, up to 32 characters.
- `digest`: optional, up to 120 characters.
- `body`: complete Markdown article, required for readiness, up to 50,000 characters.
- `coverImage`: one current-space image asset used as the explicit WeChat draft cover. A document remains previewable without it, but cannot be synced to WeChat drafts.
- `themeId`: one theme registered for this Format. If the current theme list is not already available, submit an `action: "describe_format"` request for `wechat_official.article` through the target Creation Item query adapter instead of guessing.
- `themeVersion`: always `1`.
- `images`: up to 20 ordered, unique current-space image asset references.

## Theme selection

Choose an active registered theme that matches the requested reading experience. Do not hardcode a theme list in this Skill; the Format Registry is authoritative.

Treat changing a theme as replacing the same document. Do not create a new document only to
preview another theme unless the user explicitly wants multiple independent versions.

## Writing guidance

- Write for reading, not for spoken delivery. Do not turn the article into a video script.
- Keep the article title only in `title`; do not repeat it as the first heading in `body`.
- Keep one clear argument or narrative arc. Use headings only when they improve scanning.
- Prefer readable paragraphs, concrete examples, and the user's natural voice.
- In Chinese prose, close bold markers before sentence punctuation when text continues immediately:
  write `**重点**。后续` instead of `**重点。**后续`.
- Use Markdown tables and code blocks sparingly because narrow mobile layouts are the primary preview.
- Do not emit raw HTML, CSS, JavaScript, iframe, or style attributes.

## Images

Use a stable Oaky image asset in Markdown as:

```markdown
![图片说明](clipbase-asset://550e8400-e29b-41d4-a716-446655440000)
```

List the same asset in `images`:

```json
{
  "assetId": "550e8400-e29b-41d4-a716-446655440000"
}
```

Never invent an `assetId`; obtain it from image generation or the target file query adapter. Put each referenced
Oaky asset in `images` once, ordered by its first appearance in `body`.
Set `coverImage` explicitly. It may reference the same asset as a body image, but it is not inferred from `images[0]`.

## Write

```json
{
  "action": "create",
  "formatId": "wechat_official.article",
  "schemaVersion": 1,
  "title": "文章标题",
  "publicationTargets": [
    { "targetId": "wechat_official.article", "config": {} }
  ],
  "data": {
    "title": "文章标题",
    "author": "作者名",
    "digest": "一段可独立阅读的摘要",
    "body": "正文开场。\n\n## 第一部分\n\n正文段落。",
    "themeId": "raphael-notion",
    "themeVersion": 1,
    "images": []
  }
}
```

The example omits `coverImage` only because no real asset exists yet. Once image generation or the file query adapter returns a real asset id, add `"coverImage": { "assetId": "<real asset id>" }` to `data`; a document without it stays previewable but cannot be synced to WeChat drafts.

Submit the complete `data` object directly. On replace, also send `itemId` and `expectedRevision`. Creating or replacing the document only creates a previewable article; do not claim it was saved to a WeChat draft unless the separate authenticated draft-sync action succeeds.
