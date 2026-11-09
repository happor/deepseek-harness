# Execution

The process and filesystem world: the executors that run commands, the filesystem providers that own file identity, and the terminals, remote connections, language servers, and program runtimes they host — all confined by one sandbox. What makes them a single context is a shared execution world, so a provider swap moves every capability at once; the word `seam` is defined once in the [root glossary](../../glossary.md#capability-seam) and is not redefined here.

## Language

### Execution world

**execution world**: The path and process namespace that the mounted filesystem, subprocess, and sandbox providers share, so commands, terminals, and language servers all see the same directories and executables. Pointing those providers at a remote host moves every one of them together. _Lives in_: `docs/architecture.md` (§Capability seams); `docs/subsystems/ssh.md` (§Execution coordinates).

### Filesystem

**target**: A path that a filesystem backend has resolved into a stable identity, which every later operation takes in place of a raw path. Its display path is for people and the model; its identity is opaque and may name a remote object rather than a local path. _Lives in_: `docs/subsystems/filesystem.md` (§Target identity and metadata); `packages/fs`.

**file version**: The backend-owned freshness token for one file at one moment, which a guarded mutation checks before it publishes. A mismatch means the caller's observation is stale, so consumers may display the token but never interpret it. _Lives in_: `docs/subsystems/filesystem.md` (§Target identity and metadata; §Write and edit guards); `packages/fs`.

**read-before-edit**: The optional policy that permits a write, edit, delete, or move only when the same owner has already observed the file present at a version — or confirmed absent, for a create. It decides mutations through filesystem intents, so removing it leaves the bare provider's unconditional behavior instead of breaking the tools. _Lives in_: `docs/subsystems/filesystem.md` (§Observed-file state); `packages/fs/fs-observation-policy`.

### Shell and processes

**resolved spec**: The fully-specified form of an execution request once the executor has filled and capped every optional field; it is the only shape the executor acts on. The split keeps defaults with the executor rather than hidden inside the spawn. _Lives in_: `docs/subsystems/shell.md` (§Request vs. spec); `packages/shell`.

**foreground run**: A command run whose caller awaits its outcome instead of keeping a handle — foreground is a property of what the caller awaits, not of the spawn. Its result reports exit status, signal, timeout, and abort independently, so a cut-short run is never read as a clean success. _Lives in_: `docs/subsystems/shell.md` (§Foreground runs); `packages/shell`.

**managed range**: The provider-private set of processes and terminal sessions that one spawn owns, and that both termination and exit waiting observe. Providers disclose what they can actually observe, so a consumer's teardown ladder can hold each tier on real quiescence. _Lives in_: `docs/subsystems/subprocess.md` (§Handles: streams, readers, and managed-range termination); `packages/subprocess`.

**collected output**: A bounded capture of one child stream that keeps the tail in memory and, when spilling is enabled, keeps the complete stream recoverable from a file. Readers address it by stream offset and never consume it, so independent readers cannot steal one another's bytes. _Lives in_: `docs/subsystems/subprocess.md` (§Handles); `packages/subprocess`.

**managed environment namespace**: The `DSH_*` namespace of harness-owned child-process facts, rebuilt for every execution: ambient values are discarded and only the current trusted snapshot is injected. It is the one deliberate channel through which harness facts reach a child. _Lives in_: `docs/subsystems/subprocess.md` (§Managed environment namespace and captured output); `packages/shell/shell-env`.

### Terminals

**terminal session**: A persistent, owner-scoped interactive shell or REPL kept alive across tool calls, with its own opaque identity and at most one active send. Its state is process-local and does not survive a restart. _Lives in_: `docs/subsystems/terminal.md`; `packages/terminal`.

**wait reason**: Why one interactive send returned control: the shell is waiting for input, was inferred idle, hit a deadline, or the top-level shell exited. It is independent of that shell's process status, because silence can return while the shell is still alive. _Lives in_: `docs/subsystems/terminal.md` (§Identity and readiness); `packages/terminal`.

### Sandbox

**sandbox mode**: The file-effect policy a confined execution runs under: `read-only`, `workspace-write`, or `danger-full-access`. It governs file effects only — network and process visibility are outside its vocabulary — and only the two confining modes are ever handed to a provider. _Lives in_: `docs/subsystems/sandbox.md` (§Modes and enforcement); `packages/sandbox`.

**enforcement**: How completely a backend achieves the promise of the mode it was given: `full` when it governs every promised file effect, `partial` when a backend limit or older kernel interface governs only a subset. A caller that needs an absolute boundary must not read `partial` as `full`. _Lives in_: `docs/subsystems/sandbox.md` (§Modes and enforcement); `packages/sandbox`.

**sandbox policy**: The complete file-effect authority resolved for one capability call: mode, workspace root, any extra writable roots, and an opaque owner identity that backends key their per-call state by. It travels per call, so two consumers can confine under different boundaries at the same instant. _Lives in_: `docs/subsystems/sandbox.md` (§Per-call policy); `packages/sandbox/sandbox-policy`.

**confined argv**: The wrapped command line a consuming executor spawns in place of its own, together with the enforcement fact and the stderr evidence that distinguishes a denial from a runner failure. The consumer keeps its own argv; the provider owns only the fence around it. _Lives in_: `docs/subsystems/sandbox.md` (§Wrapped argv and classification dialects); `packages/sandbox`.

**runner failure**: Evidence that the sandbox runner refused or failed before the wrapped command ever ran — the opposite of a denial, where confinement worked and blocked the command. Consumers surface it as a sandbox infrastructure failure, never as an ordinary task failure. _Lives in_: `docs/subsystems/sandbox.md` (§Wrapped argv and classification dialects); `packages/sandbox/sandbox-local`.

### Remote execution

**remote helper**: The installed executable on an SSH host whose identity and digest the connection verifies before use, and which runs filesystem and process requests there through trusted local providers. It is what lets remote work reuse the local capability contracts instead of a parallel remote API. _Lives in_: `docs/subsystems/ssh.md` (§Transport and trust); `packages/ssh`.

### Code intelligence

**language-server provider**: A backend registered on the language-server seam that owns one stable identity and a lowercase, leading-dot extension map, and answers normalized navigation queries for files it claims. Selection is per query and order-independent, so changing servers never changes how the model asks for navigation. _Lives in_: `docs/subsystems/lsp.md` (§Provider and service); `packages/lsp`.

### Program execution

**binding**: A host function exposed to a running program as one async global, callable from the program and restricted to lossless JSON in both directions. Bindings are how a program reaches harness capabilities without the runtime knowing any particular consumer. _Lives in_: `docs/subsystems/ptc-runtime.md` (§Bindings); `packages/ptc-runtime`.

### Document conversion

**document engine**: The converter a target declares for Office-to-PDF: a precompiled native engine where the kit supplies one for that target, otherwise Node WASM. Invalid metadata, missing assets, and conversion errors reject rather than silently switching engines, and conversion always runs on the host. _Lives in_: `docs/subsystems/office-to-pdf.md` (§Engine selection and limits); `packages/document`.
