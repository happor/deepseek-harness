# Session Data
The durable session plane: the append-only event log that is the single source of truth for an interaction, the formats it is persisted in, the projections and retrieval built over it, and the records kept around it.

Members: `packages/session` (22), `packages/session-query` (3), `packages/storage` (4), `packages/workspace` (1), `packages/attachment` (2), `packages/spill` (3), `packages/feedback` (2), `packages/deliverables` (3) — 40 packages.

Terms shared with other contexts are defined once in the root [glossary](../../glossary.md) — a session's [turn and step](../../glossary.md#turn) belong to the loop, and [seam](../../glossary.md#capability-seam) belongs to the capability vocabulary.

`experimental/` packages are filed here by subject: `forge*` (the change-request registry and its providers, bound to the Session that tracked or pushed them), `worktree*` (named Git worktrees that change a Session working directory), `liveshare*` (opt-in Session sharing through a Relay), and `session-inspector` (a reader over Session logs).

## Language

### The log

**Session**: The durable record of one agent interaction: an append-only, event-sourced log of typed events from which every model message and every client read is derived, never stored alongside it. _Lives in_: [`packages/session/`](../../../packages/session/README.md) (the live Session type itself is declared in [`packages/core/session/`](../../../packages/core/session/README.md)), [`docs/subsystems/session.md`](../../subsystems/session.md). _Avoid_: conversation, chat, thread.

**session event**: One committed, immutable entry in a session log, carrying a type, a contiguous sequence position, a timestamp, and lossless-JSON data; a reader that meets an unrecognized type refuses the log unless the event is explicitly marked as safe to skip. _Lives in_: [`docs/subsystems/session.md`](../../subsystems/session.md), [`docs/persistence-catalog.md`](../../persistence-catalog.md) (the envelope and every declared event). _Avoid_: log line, row, record.

**surface**: The ordered sequence of message-producing events, and the only thing the model's history is derived from: an event either appends a node or replaces an inclusive range of existing nodes, which is how compaction shadows what it summarized. _Lives in_: [`docs/subsystems/session.md`](../../subsystems/session.md) (Surface types), [`packages/session/`](../../../packages/session/README.md). _Avoid_: transcript, message list, history table.

**log-only event**: An event that contributes nothing to the surface and therefore never reaches the model; the plugin that declares it also decides whether it belongs inside an open turn or may stand between turns. _Lives in_: [`docs/subsystems/session.md`](../../subsystems/session.md), [`docs/persistence-catalog.md`](../../persistence-catalog.md). _Avoid_: internal event, metadata event.

### Durability and format

**session format version**: The logical generation of a persisted log, stamped in the session's metadata beside the log rather than inside it; the writer constant names the current generation, while the finalization and release records name what has actually shipped. _Lives in_: [`docs/session-format-status.md`](../../session-format-status.md), [`packages/session/session-format/`](../../../packages/session/session-format/README.md). _Avoid_: schema version, storage layout.

**adjacent migration**: The frozen one-step conversion from one format version to the next; published generations are immutable, so a change may only add a successor and never move, overwrite, or delete committed data. _Lives in_: [`docs/persistence-changes/README.md`](../../persistence-changes/README.md), [`packages/session/session-format-v0-to-v1/`](../../../packages/session/session-format-v0-to-v1/README.md) … [`session-format-v3-to-v4/`](../../../packages/session/session-format-v3-to-v4/README.md). _Avoid_: upgrade, converter, backfill.

**persistence backend**: The provider that makes a session log durable, storing the very same events with no parallel persisted event type and never handing a reader a torn physical tail — an incomplete fragment of an append that never resolved is discarded before the next write. _Lives in_: [`packages/session/session-persistence/`](../../../packages/session/session-persistence/README.md), [`session-persistence-jsonl/`](../../../packages/session/session-persistence-jsonl/README.md), [`docs/subsystems/persistence.md`](../../subsystems/persistence.md). _Avoid_: store, database, log file.

**session handle**: The one open channel onto a stored session, and the only door every read and write passes through; a write handle holds exclusive ownership, so a second writer is refused instead of racing. _Lives in_: [`docs/subsystems/persistence.md`](../../subsystems/persistence.md), [`packages/session/session-persistence/`](../../../packages/session/session-persistence/README.md). _Avoid_: connection, writer, stream.

**durability barrier**: The one operation that promises storage — an append is only best-effort until the barrier resolves, at which point every acknowledged event survives a crash and the session is materialized for other processes. The repo performs it as a *flush*. _Lives in_: [`docs/subsystems/persistence.md`](../../subsystems/persistence.md), [`packages/session/session-checkpoint-policy/`](../../../packages/session/session-checkpoint-policy/README.md). _Avoid_: save, commit, sync.

**interrupted-turn repair**: The recovery rule that a log crashed mid-turn keeps its open tail rather than being truncated, and is closed after the fact with a synthetic turn end; a read-only observer balances the same log in memory and writes nothing back. _Lives in_: [`docs/subsystems/persistence.md`](../../subsystems/persistence.md#crash-recovery-preserves-an-interrupted-turn). _Avoid_: rollback, truncation, cleanup.

### Derived state

**projection unit**: One domain's pure, synchronous fold from committed events to a whole current value — a computation the framework drives on every event, not a getter a consumer calls. _Lives in_: [`packages/session/session-projection/`](../../../packages/session/session-projection/README.md), [`docs/subsystems/session-projection.md`](../../subsystems/session-projection.md). _Avoid_: selector, derived view, reducer.

**projection snapshot**: One consistent read cut over every registered client-visible unit for a session, stamped with the shared watermark of the last event all of them reflect, so a carrier never sees values from different log positions. _Lives in_: [`docs/subsystems/session-projection.md`](../../subsystems/session-projection.md), [`packages/session/session-projection/`](../../../packages/session/session-projection/README.md). _Avoid_: state dump, cache read.

**projection checkpoint**: The persisted per-key state rows that let a later read resume from a watermark instead of replaying the whole log, on the rule that a stale row must yield an explicit outcome rather than silently falling back to a full read. _Lives in_: [`packages/session/session-projection-cache/`](../../../packages/session/session-projection-cache/README.md), [`docs/subsystems/session-projection.md`](../../subsystems/session-projection.md), [`docs/defensive-patterns/conversation-and-session.md`](../../defensive-patterns/conversation-and-session.md). _Avoid_: snapshot, savepoint, durability checkpoint.

### Retrieval and lineage

**session corpus**: The set of sessions a reader can see, resolved live-preferred: a live session supersedes its stored copy, and every read returns which source answered along with the record. _Lives in_: [`packages/session-query/`](../../../packages/session-query/README.md), [`docs/subsystems/session-query.md`](../../subsystems/session-query.md). _Avoid_: session list, index, database.

**fork seed**: The exact leading prefix of a source session's log that a child session is created with, cut by an end-seed marker so inherited history stays distinguishable from the child's own work after a fork, a crash, or a restore. _Lives in_: [`docs/subsystems/session.md`](../../subsystems/session.md), [`packages/session/`](../../../packages/session/README.md). _Avoid_: copy, clone, branch snapshot.

### Records around the log

**working directory**: A session's effective directory, which may move away from the original project recorded in its header without changing existing processes, sandbox roots, or project membership. _Lives in_: [`packages/session/working-directory/`](../../../packages/session/working-directory/README.md), [`docs/subsystems/working-directory.md`](../../subsystems/working-directory.md). _Avoid_: project root, workspace, cwd.

**Workspace**: A named, ordered project record over one canonical directory path that owns the candidate account of sessions belonging to it; a session outside every account is ungrouped, archiving hides it without touching its log, and membership is validated against the session's recorded directory. _Lives in_: [`packages/workspace/`](../../../packages/workspace/README.md), [`docs/subsystems/workspace.md`](../../subsystems/workspace.md). _Avoid_: project, folder, working directory.

**storage domain**: A schema-declared, versioned set of typed records that a host package opens over a configured storage backend, with every durable change announced only after the backend accepted it. _Lives in_: [`packages/storage/`](../../../packages/storage/README.md), [`docs/subsystems/storage.md`](../../subsystems/storage.md). _Avoid_: table, collection, KV store.

**attachment reference**: An immutable, content-addressed handle to committed binary bytes; session events and model-visible blocks carry the reference and its verified metadata, never a path, URL, or encoded payload. _Lives in_: [`packages/attachment/`](../../../packages/attachment/README.md), [`docs/subsystems/attachment.md`](../../subsystems/attachment.md). _Avoid_: attachment id, file path, blob.

**deliverable**: What a turn hands to the user, recorded in log-only events that only clients read: the files the model declared and the files the turn changed. The split matters — only [`tool-present`](../../../packages/deliverables/tool-present/README.md) is model-facing; the changed-files summary is kept and served by the host and never enters the log body. _Lives in_: [`packages/deliverables/`](../../../packages/deliverables/README.md) (only `tool-present` is model-facing), [`docs/subsystems/deliverables.md`](../../subsystems/deliverables.md). _Avoid_: artifact, output, result file.
