# `generic.script@1`

Use for complete platform-neutral scripts, spoken drafts, narration, copy variants, and other text-first deliverables. Keep title or hook candidates, direction comparisons, and short samples used only to choose a direction in chat. Once the user selects a direction or asks for complete variants, use one document per finished script.

## Data

```json
{
  "style": "口语、直接",
  "body": "完整稿件正文"
}
```

- `style`: optional semantic label, up to 100 characters; use an empty string when unnecessary.
- `body`: complete script text, up to 12,000 characters.
- Put the user-facing document name in the request-level `title`, not inside `data`.

## Write

```json
{
  "action": "create",
  "formatId": "generic.script",
  "schemaVersion": 1,
  "title": "产品发布口播稿",
  "data": {
    "style": "口语、直接",
    "body": "完整稿件正文"
  }
}
```

Keep `schemaVersion` numeric and `data` as the native object shown above. On replace, also send `itemId` and `expectedRevision`. Write one complete script per document. Create separate documents for complete alternatives; do not turn exploratory snippets into documents. Organize them in a folder only when the user asks for that structure.
