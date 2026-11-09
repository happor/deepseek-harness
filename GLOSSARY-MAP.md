# Glossary Map

## Contexts

- [Platform](./docs/contexts/platform/GLOSSARY.md): shared primitives, the typed-remote type graph, config and secret surfaces, and telemetry channels every other context consumes.
- [Agent Loop](./docs/contexts/agent-loop/GLOSSARY.md): the turn/step engine and the model I/O vocabulary — what the model is asked, with which tools and context, and when the loop compacts or stops.
- [Session Data](./docs/contexts/session-data/GLOSSARY.md): the durable session plane — event log, persisted formats, projections, retrieval, and the attachments, storage and workspace records around it.
- [Execution](./docs/contexts/execution/GLOSSARY.md): the process and filesystem world — executors, PTYs, remote connections, language servers, code execution, and the sandbox that confines them.
- [Capabilities](./docs/contexts/capabilities/GLOSSARY.md): model-facing capability plugins — one seam plus providers plus tool per capability, with delegation and orchestration.
- [Interaction](./docs/contexts/interaction/GLOSSARY.md): human decisions and session-scoped policy state that steer the loop — approvals, questions, commands, permissions, plan, goal, schedule, todo.
- [Web GUI](./docs/contexts/web-gui/GLOSSARY.md): the browser half, the host half, and the typed Remote layer that joins them into one product surface.
- [Distribution](./docs/contexts/distribution/GLOSSARY.md): how a profile becomes a running application — bundle layers, boot glue, the CLI/Desktop carriers, and the SDK/ACP entrypoints.

## Relationships

- **Platform → every context**: supplies the shared primitives, typed-remote graph, config/secret surfaces and telemetry transport; no feature context owns them.
- **Agent Loop → Session Data**: the loop decides each turn; the session plane persists its events, formats and projections.
- **Capabilities → Execution**: capability providers act through the execution world (shell, fs, subprocess, sandbox).
- **Interaction → Agent Loop**: approvals, questions and plan mode steer whether and how the loop continues.
- **Web GUI ↔ Agent Loop / Session Data**: the Remote layer presents turns and session history in the browser.
- **Distribution → all contexts**: assembles profiles and bundles into the CLI, desktop and SDK/ACP entrypoints.

## Translation

These glossaries are English-only by policy: `docs/contexts/` is one explicit exclusion in `scripts/translation-pairing.manifest.json`, so a new context glossary needs no manifest change. A future translation pass removes that entry and adds the pairs together with their `.i18n.yaml` records.
