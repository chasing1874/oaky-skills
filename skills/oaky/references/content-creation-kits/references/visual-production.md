# Reference-based visual production

The user describes an outcome in ordinary language. You own the reference selection, file identities, production order and quality checks; do not ask the user to write technical prompts or manage assets.

## Establish what must stay consistent

- Locate the user's actual references: source content, visual style, and recurring subject/IP. Read the relevant images, not just their titles. A note mentioning the brand or a generic picture of an acorn does not establish its character design.
- Keep a small working list of each selected file ref, its role, and what it constrains. Use stable refs already returned by source/file/generation tools. Preserve the revision on space files. Reuse existing assets instead of uploading or generating duplicates just to form a sequence.
- If the requested IP/subject image cannot be found, ask for it before producing pages that depend on it. Never invent ears, a costume, or a new character as a substitute for the user's identity.
- Select only the most useful references within the image tool's limit. Usually one or two style examples plus a subject image suffice. Do not send every saved image or unrelated recent note.

## Looking at a reference is not passing it to the generator

The image-reading tool shows images to the planning model. It does not attach them to the next image-generation request. Every generation that relies on an image must explicitly include that file in `references`:

- `style_reference`: visual language, palette, texture or layout rhythm.
- `content_reference`: subject identity, recurring character or relevant composition.
- `edit_target`: the particular existing page being changed.

Use the actual file ref, not an opaque imageRef, URL, filename or a description of the image. Prompt text explains how to use the supplied references; it cannot replace their bytes. Name the role of each supplied image in the prompt so multiple references do not contradict each other.

## Decide order from dependencies

| Situation | Action |
| --- | --- |
| One independent image with no required reference | Generate directly. |
| New series with no accepted visual baseline | Generate one representative sample using the actual style and subject references, then inspect it. |
| User asked to approve the sample or direction is still materially uncertain | Show the sample and wait for their choice. Reuse approvals already given; do not add a confirmation ladder. |
| User delegated execution and the sample satisfies the agreed direction | Continue after your own visual check; do not require extra approval. |
| Accepted sample/character baseline already exists | Reuse it directly. Do not rebuild the baseline. |
| Several pages share a ready baseline and do not depend on each other's output | Generate them in parallel, explicitly passing the same baseline and relevant original refs in every call. |
| A task needs another task's output | Wait for that output and capture its returned ref before starting the dependent task. |
| Correct one existing page | Pass that exact page as edit_target, state the small change and invariants, and preserve the other pages. |

For this comic workflow: style reference + original IP -> representative sample -> remaining pages sharing that sample. Do not chain page 2 from page 1, page 3 from page 2, etc. unless that changing state is intentional; cumulative drift can otherwise replace the original identity.

## Reuse and verify

- To recover the direct references for an older generated page, call `queryFiles` with `action: "references"`, its exact `space_file` ref (including revision), and the same project context. Reuse only entries marked `ready`; unavailable entries do not grant access. `recorded` with an empty list means no reference images were supplied; `not_recorded` means the historical recipe cannot be established. The query is one hop, not recursive, and does not inspect image bytes. External callers use the same object through the file query adapter.
- Each successful generation returns a saved file ref. Keep that ref with the page number and use it in downstream calls. Distinguish using an asset as input from creating a new version of it; follow the file tool's explicit write intent when a revision is required.
- Inspect the sample against the original references for character identity, palette, line treatment and layout density. A style name such as “handwritten” is not sufficient evidence of a match.
- Inspect final pages for identity/style consistency, exact intended copy, order and dimensions. Inspect any page you claim to have verified; if you only sampled pages, say which were checked.
- Reuse healthy outputs after a failure; retry only failed or rejected pages. Never substitute text-only generation when required reference loading fails.
- Deliver the ordered pages through the selected Creation Document format when the user requested a publishable carousel. Do not treat a pile of independently generated files as the completed structured post.

## Native imageGen argument shapes

The UUIDs below are illustrative. Replace each with a real returned file ref.

A sample with style and subject:

```json
{
  "prompt": "用第一张的画风画第二张中的同一个橡果角色，做一页漫画样张，保留它的帽子、眼睛和身体特征。",
  "references": [
    { "file": { "source": "clip_file", "id": "00000000-0000-4000-8000-000000000001" }, "role": "style_reference" },
    { "file": { "source": "space_file", "id": "00000000-0000-4000-8000-000000000002", "revision": 1 }, "role": "content_reference" }
  ],
  "size": "1024x1536"
}
```

A later page sharing an accepted sample and the original subject:

```json
{
  "prompt": "沿用第一张样张的人物，画风参考第二张，橡果细节以第三张原设为准。画下一页：她带着收藏和自己的想法开始聊天。不要复制样张的文字。",
  "references": [
    { "file": { "source": "space_file", "id": "00000000-0000-4000-8000-000000000003", "revision": 1 }, "role": "content_reference" },
    { "file": { "source": "clip_file", "id": "00000000-0000-4000-8000-000000000001" }, "role": "style_reference" },
    { "file": { "source": "space_file", "id": "00000000-0000-4000-8000-000000000002", "revision": 1 }, "role": "content_reference" }
  ],
  "size": "1024x1536"
}
```

These are image tool arguments, not document payloads. External callers use the external image tool's declared transport and project/idempotency requirements; do not send them to the Creation Document CLI commands.
