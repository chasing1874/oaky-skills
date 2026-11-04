# Creation Document Workflow

## Document workflow

1. Decide whether the current turn is exploration or delivery. Direction comparisons, title or hook candidates, critique, and short samples used to choose a direction stay in chat. Give a few concrete options and one recommendation, then offer to turn the selected direction into a complete document. Stop without writing a document in that turn.
2. Treat a concise confirmation such as “the second one”, “okay”, or “write it” as approval to continue from the selected direction. If the user already asks for complete, independently usable, ready-to-use, or ready-to-publish output, skip the exploratory gate and write it without asking again. A request for multiple complete variants means one document per variant. If the user explicitly asks for chat-only output or says not to save it, do not write a document.
3. Infer the audience, goal, tone, source material, platform, content type, and constraints.
4. Load the selected format reference.
5. Before generating final media or writing a multi-page, multi-image, mixed-media, or otherwise structure-sensitive platform document, present one opinionated creative blueprint. Include the selected content mode, audience and core claim, every page or section's job and proposed copy, the role of every source image or illustration, and the deliberate cover. Ask for approval once. Do not stack a separate direction-selection confirmation on top of this gate; include the recommended direction in the blueprint.
6. Treat the request as permission to enter the workflow, not approval of an unstated structure. Skip the blueprint approval only when the user explicitly delegates the choices, supplies an exact final structure, says to use the presented plan, or asks to reuse an already approved template. Do not use an execution plan as a substitute for the creative blueprint.
7. If the user materially changes the content mode, page or section order, copy, or media role, replace the old blueprint and ask for approval again before producing final artifacts. Do not interpret the correction as approval of the previous plan.
8. Query an existing document before replacing it, and use its latest `expectedRevision`.
9. Treat `mediaManifest` from a document read as an inventory, not a visual summary. It gives each image's fixed file reference, document order, structural role, optional caption, and actual file name. The program does not attach all document images or decide which ones matter.
10. Decide whether visual inspection is necessary for the current task. Text-only rewriting, title, digest, topic, or structure changes normally use the document text without loading images. For cover review, a named image change, annotation, or overall visual critique, select the smallest relevant set from `mediaManifest` and use the target adapter's image-reading tool. File names can narrow candidates but never prove image contents; when names are generic, use role and order. Before producing a visual series, follow [visual-production.md](visual-production.md): establish shared references first; then generate independent pages with those same refs.
11. Create a new document for a new output or cross-platform adaptation.
12. Treat a structured validation error as repair instructions: inspect `path`, compare `expected`, correct the complete data object, and retry.
13. On revision conflict, fetch the latest document, reapply the requested change, and retry without overwriting unseen work.
14. After writing, summarize briefly and rely on the returned Artifact for display.
