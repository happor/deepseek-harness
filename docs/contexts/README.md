# Context glossaries

One `GLOSSARY.md` per context: a domain seam over `apps/` and `packages/` that carries vocabulary of its own. [The glossary map](../../GLOSSARY-MAP.md) owns the context list, each context's one-line purpose, and how the contexts relate; this page owns only what the directory is and what an entry must look like.

## What an entry is

One physical line per entry — `**term**: definition. _Lives in_: … _Avoid_: ….` The definition is one or two sentences of domain language, with no API signatures and no code; _Lives in_ names the group, package, or owning doc where the term lives; _Avoid_ names the synonyms the term displaces.

A term has one home. Where [the published glossary](../glossary.md) already defines it, the context glossary links there instead of redefining it; a term that legitimately spans two contexts lives in its home context, and the other context links to it. A context with fewer than three real terms stays absent rather than written thin.

## Adding a context

Write `docs/contexts/<slug>/GLOSSARY.md`, then add its link to [the glossary map](../../GLOSSARY-MAP.md). The directory exclusion in [the pairing manifest](../../scripts/translation-pairing.manifest.json) covers the new page, so no manifest edit is needed; a future translation pass removes that entry and adds the pairs with their `.i18n.yaml` records, per the rule stated in the map.
