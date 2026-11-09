# Web GUI

The product surface people see in a browser: the browser half (`apps/web`, `packages/client`), the host half (`packages/host`), and the typed Remote layer (`packages/api`) that joins them. What makes this one context is a single wire contract — the same change moves the Host service, the generated Remote face, and the browser model together; terms shared with the rest of the repo are defined once in the root [glossary](../../glossary.md), and this file owns only what is specific to the browser–host product surface.

`experimental/` packages are filed here by subject: `inspector` (a cross-realm CDP hub for Host debugging and Client Runtime inspection), `inspector-profile` (an optional Web bundle that inspects Session data in the Sidebar and opens the NodeJS inspector), `webworker-packer` (the build-time packer for the browser runtime's VFS image) and `webworker-runtime` (the browser-only runtime it packs).

## Language

### Halves and the wire

**Web Client**: The browser-side Cordis application of the web GUI, assembled at page load from independently built plugins rather than compiled as one bundle. It owns presentation and browser-local viewing state; every durable fact it shows originates on the Host. _Lives in_: `packages/client` · `apps/web` · [web-client.md](../../subsystems/web-client.md). _Avoid_: frontend, SPA, browser app.

**Remote layer**: The typed call surface between Client and Host: a Host service declares the methods and streams a browser may call, and the browser sees them as concrete typed methods grouped by namespace. It is the layer that makes Host capabilities callable without importing Host runtime code. _Lives in_: `packages/api` · [api-gateway.md](../../api-gateway.md). _Avoid_: RPC endpoints, backend API, BFF.

**Connection**: The shared browser–Host wire that carries Remote calls, forwarded Host events, and exact Fetch responses for one page, and owns trust, request correlation, and cancellation. Every Remote call reuses it; no feature opens its own transport. _Lives in_: `packages/client/connection` · [web-client.md](../../subsystems/web-client.md) (§Remote communication). _Avoid_: socket, transport, bridge.

**Connection generation**: One contiguous epoch of the browser–Host wire, opened by a ready signal only after every incremental listener is attached. Each generation begins with a baseline that atomically replaces the browser state built on the previous epoch, so a reconnect never merges two histories. _Lives in_: `packages/client/connection` · [web-client.md](../../subsystems/web-client.md) (§Reconnection). _Avoid_: reconnect, session, socket lifetime.

**Client model**: The React-free browser-side projection paired with one Host controller, exposing identity-stable snapshots and narrow commands. It merges delayed responses and replacement baselines, but never becomes a second source of business truth. _Lives in_: `packages/api/session-controller` and `packages/api/workspace-controller` (client faces) · [web-client.md](../../subsystems/web-client.md) (§Client models). _Avoid_: store, cache, view model.

### Serving and boot

**Client module**: A plugin package's browser half, loaded into the page by the client module system instead of the shell's static bundle. Modules are the unit of boot, live replacement, and plugin unload. _Lives in_: `packages/client/modules` · [client-modules.md](../../subsystems/client-modules.md). _Avoid_: chunk, script, micro-frontend.

**Boot graph**: The ordered set of client modules the Host composes for one page boot, together with the release identity that decides whether that page may accept a later composition. It is the wire between the Host's composition and the browser's module table. _Lives in_: `packages/client/modules` · [client-modules.md](../../subsystems/client-modules.md). _Avoid_: manifest, entry list, module graph.

**Web server**: The browser HTTP carrier of the web GUI Host: a named-route registry with exactly one fallback seat for everything unclaimed, plus the index rendering every page passes through. It serves browsers only and owns no authentication or capability policy of its own. _Lives in_: `packages/host/webserver` · [web-server.md](../../subsystems/web-server.md). _Avoid_: static server, HTTP listener.

### UI composition

**Slot**: A typed extension position in the Web Client's React tree, declared by the component that owns the render location and filled by plugins at runtime. Its declaration fixes cardinality — single, list, keyed, chain, or hoc — and scope — root, session-maybe, or session. _Lives in_: `packages/client/ui-slots` · [slots.md](../../subsystems/slots.md). _Avoid_: extension point, outlet, render prop.

**Component Factory**: A reusable slot definition that carries its own child Slots and is rendered by reference from another component's position, so one assembly serves several call sites without redeclaring its interior. It is chosen for reuse, not required for a one-off component. _Lives in_: `packages/client/ui-slots` · [slots.md](../../subsystems/slots.md). _Avoid_: component library, template, wrapper.

### Conversation

**Conversation**: The target-neutral assembly layer between a Session's loaded event window and its views, correlating events into business Contexts and publishing one snapshot per registered target. It owns identity routing, ordered replay, and Location data, never a target's final display model. _Lives in_: `packages/client/ui-conversation` · [conversation.md](../../subsystems/conversation.md). _Avoid_: chat, transcript, thread.

**Event Definition**: A business package's contribution for one event family: it matches each event by a stable business id, folds deterministic State, and optionally materializes one renderable node for its target. Durable and Client-only transient events arrive through the same interface. _Lives in_: `packages/client/ui-conversation` and target packages · [conversation.md](../../subsystems/conversation.md) (§Data model and ownership). _Avoid_: parser, reducer, event handler.

**View target**: A rendering destination a Session can activate — Chat, Trajectory, or a third-party View — which owns its incremental builder and its own final snapshot type. Targets may interpret the same event family but never share each other's display model. _Lives in_: `packages/client/ui-conversation` and target packages (`ui-chat`, `ui-trajectory`) · [conversation.md](../../subsystems/conversation.md). _Avoid_: tab, view mode, renderer.

**Location**: The engine-owned Session, Turn, or Step coordinates a Conversation node is placed on, derived from durable boundary events rather than declared by a View. An Event Definition may publish typed data onto one Turn or Step for another node in that Location to read. _Lives in_: [conversation.md](../../subsystems/conversation.md) · `packages/client/ui-conversation`. _Avoid_: position, anchor, scope.

### Right Sidebar

**Right Sidebar**: The per-Session docking surface beside the conversation, where addressed content opens as tabs in panes that split, float, and close. It holds a layout history entry per change and keeps layouts browser-local. _Lives in_: `packages/client/ui-sidebar-right` · [sidebar-right.md](../../subsystems/sidebar-right.md). _Avoid_: drawer, inspector, side panel.

**Tab type**: One kind of Right Sidebar tab, made of a static definition that claims addresses and a keyed Slot body that renders any tab of that kind. A tab's identity is the pair of its type and its address. _Lives in_: `packages/client/ui-sidebar-right` · [sidebar-right.md](../../subsystems/sidebar-right.md) (§Tab-type registration). _Avoid_: tab plugin, viewer, panel type.

### Resources

**Resource address**: A `dsh-resource://` URL naming one piece of live content: the host names the protocol and the path belongs to that protocol, so adding content means adding a protocol rather than a scheme. Addresses compare as strings, and only the canonical spelling opens a resource. _Lives in_: `packages/client/resources` · [client-resources.md](../../subsystems/client-resources.md). _Avoid_: URL, path, content id.

**Resource provider**: The single registered source behind one resource protocol, yielding an address's current value and then every change while the resource has a holder. Disposing it ends those streams and makes its addresses read as absent. _Lives in_: `packages/client/resources` · [client-resources.md](../../subsystems/client-resources.md) (§Registering a provider). _Avoid_: fetcher, loader, data source.

### Styling

**Semantic alias**: A theme token a feature component consumes by role — surface, text, border, elevation — instead of a palette value, so light/dark and material behavior stay owned by the theme package. Shared tokens belong to the theme; feature CSS never defines a second global system. _Lives in_: `packages/client/ui-theme` · [web-styling.md](../../web-styling.md). _Avoid_: CSS variable, color constant, theme class.

**Radius scale**: The role-based corner-radius standard every control and surface selects from — small detail through enclosing panel — so a local adjustment never invents an off-scale value. Role comes first, size variant second. _Lives in_: `packages/client/ui-theme` · [ui-radius.md](../../ui-radius.md). _Avoid_: border-radius, rounding, size.

## Owned elsewhere

- [turn](../../glossary.md#turn) and [step](../../glossary.md#step) — here they appear only as Conversation Locations: derived coordinates a node sits on, never a unit the Web Client counts or drives.
- [exchange](../../glossary.md#exchange) — here it names the chat-folding unit: Chat keeps the latest exchange expanded while earlier ones fold.
