# Platform
Shared primitives, the typed-remote type graph, the config and secret surfaces, and the telemetry channels that every other context consumes but none owns. It exists so a feature context never invents its own id form, path rule, JSON shape, credential resolution, or reporting transport.

Members: `packages/util` (21), `packages/typert` (4), `packages/settings` (1), `packages/credentials` (5), `packages/identity` (1), `packages/telemetry` (1), `packages/test-support` (8) — 41 packages.

## Language

### The typed-remote type graph

**Typert**: The harness's typed-remote type graph: a build-time analysis of source type declarations, a runtime registry of the artifacts it produces, and the shared declarations that let a Client call a Host capability as a typed method without hand-written wire code. _Lives in_: [`packages/typert/`](../../../packages/typert/README.md), [`docs/subsystems/typert.md`](../../subsystems/typert.md). _Avoid_: RPC framework, type system, wire layer.

**invocation descriptor**: The generated, carrier-independent description of one exported method — the identity it is called by, its ordered business parameters, and the shape of what it returns — shared by the Host build and the Client build. A descriptor is *strict* when its types were fully recovered and *SRC* when they were not, and an SRC descriptor enforces JSON-safe inputs instead of structural types. _Lives in_: [`docs/subsystems/typert.md`](../../subsystems/typert.md) (Invocation descriptors), [`packages/typert/protocol/`](../../../packages/typert/protocol/README.md). _Avoid_: schema, endpoint spec, API signature.

**remote contribution**: The generated per-package artifact that a consumer explicitly selects and mounts to receive one Host-for-Client call surface, which then lives as long as the calling fiber. Nothing is mounted implicitly, and no Host business type reaches the consumer. _Lives in_: [`docs/subsystems/typert.md`](../../subsystems/typert.md) (Consumer Remote), [`packages/typert/registry/`](../../../packages/typert/registry/README.md). _Avoid_: client SDK, bundle, plugin.

**lookup**: A declared wire identity that stands in for one Host business object that cannot cross the wire itself, so a call carries an identity and the Host resolves the object. The declaration outlives its resolver: a consumer still treats the parameter as a lookup, and the call fails as unavailable rather than accepting the value as an ordinary object. _Lives in_: [`docs/subsystems/typert.md`](../../subsystems/typert.md) (Lookup and Context declarations). _Avoid_: resolver, reference id, foreign key.

### Configuration and secrets

**volatile configuration**: A plugin configuration field whose change is applied to the already-running plugin in place, without remounting it. The settings surface projects exactly these fields, which is why editing a form does not restart the plugin it configures. _Lives in_: [`docs/subsystems/settings.md`](../../subsystems/settings.md), the loader's volatile-configuration rule in [`vendor/loader/README.md`](../../../vendor/loader/README.md), and each package's `Config` in [`docs/config-catalog.md`](../../config-catalog.md). _Avoid_: hot reload, live config, runtime config.

**form namespace**: The local id of one uniquely addressed profile entry, and the identity a configuration form and all of its edits are addressed by. Two plugin instances have separate forms exactly when their entry ids differ. _Lives in_: [`docs/subsystems/settings.md`](../../subsystems/settings.md), [`packages/settings/`](../../../packages/settings/README.md). _Avoid_: plugin id, scope, settings key.

**settings revision**: The optimistic-concurrency mark a form read returns and every write must match, so an edit built on a stale read is refused instead of overwriting a newer one. _Lives in_: [`docs/subsystems/settings.md`](../../subsystems/settings.md). _Avoid_: version, etag, timestamp.

**secret field**: A configuration field declared as a secret: no settings read may return it, and a path-addressed edit may leave it unstated instead of clearing it. A secret itself reaches configuration only as a credential reference. _Lives in_: [`docs/subsystems/settings.md`](../../subsystems/settings.md), [`docs/subsystems/credentials.md`](../../subsystems/credentials.md). _Avoid_: password field, redacted field, hidden field.

**credential reference**: A POSIX-style environment-variable name that stands for one secret, so settings sections and `cordis.yml` name a secret without carrying its value. A reference is resolved once per operation rather than once per process, which is how a rotated value reaches the very next request without a restart. _Lives in_: [`packages/credentials/`](../../../packages/credentials/README.md), [`docs/subsystems/credentials.md`](../../subsystems/credentials.md). _Avoid_: secret name, API key, secret.

**credential key**: The identity of one stored credential record a plugin holds for one id — the credential seam's second key space, answering "what credential does this plugin hold". Unlike a reference it cannot layer over environment fallbacks, so presence of the record is the whole fact. _Lives in_: [`docs/subsystems/credentials.md`](../../subsystems/credentials.md) (Resolution). _Avoid_: credential id, secret id.

**credential record**: The stored payload behind a credential key, written only through a serialized read-modify-write so a refresh cannot lose a concurrent rotation; an authorization grant held this way is a grant record. _Lives in_: [`docs/subsystems/credentials.md`](../../subsystems/credentials.md), [`packages/credentials/credentials/`](../../../packages/credentials/credentials/README.md). _Avoid_: token file, grant blob.

**authorization flow**: A plugin-owned way to obtain one credential by asking a human, registered under the credential key whose record it writes, with one attempt at a time per key. The seam owns the conversation and the attempt lifecycle; the flow owns the protocol. _Lives in_: [`docs/subsystems/credentials.md`](../../subsystems/credentials.md) (Authorization), [`packages/credentials/authorization/`](../../../packages/credentials/authorization/README.md). _Avoid_: login method, OAuth provider, auth strategy.

### Installation identity

**anonymous user id**: One anonymous identifier per harness home that telemetry exports, feedback acknowledgements, and DeepSeek requests all carry, so records from one installation can be recognized as coming from the same place without identifying the user. It is home-scoped and holds no machine or account data. _Lives in_: [`packages/identity/`](../../../packages/identity/README.md). _Avoid_: user id, installation id, tracking id.

### Telemetry channels

**OTel channel**: An independent reporting channel a consumer creates from the shared OpenTelemetry service — an ordinary-event channel batched by count, or a byte-bounded channel carrying one complete Session event per record. Channels never share a queue, a request, or a transport. _Lives in_: [`packages/telemetry/otel/`](../../../packages/telemetry/otel/README.md), [`docs/subsystems/otel.md`](../../subsystems/otel.md). _Avoid_: exporter, sink, logger.

**channel ownership**: The rule that whoever creates a channel owns its authorization, redaction, field selection, and shutdown, while the shared service supplies no policy and sends nothing on its own. Mounting the service alone allocates no provider and no transport. _Lives in_: [`docs/subsystems/otel.md`](../../subsystems/otel.md), [`packages/telemetry/otel/`](../../../packages/telemetry/otel/README.md). _Avoid_: telemetry policy, consent layer.

### Shared primitives

**harness home**: The single user-data root an installation shares, chosen by an explicit path first, then `$DSH_HOME`, then a default under the operating-system home, with every derived path joined onto it. The shared agent-configuration root sits deliberately outside it, because other agent tools read that one too. _Lives in_: [`packages/util/home-paths/`](../../../packages/util/home-paths/README.md). _Avoid_: data dir, config dir, install root.

**launch environment**: The frozen snapshot of the environment this run was launched with, which remembers which layer supplied each value instead of flattening them all into one process environment. Layers rank inherited process environment, then the invocation directory's `.env`, then the home `.env`. _Lives in_: [`packages/util/launch-environment/`](../../../packages/util/launch-environment/README.md). _Avoid_: env vars, process environment, dotenv.

**lossless JSON value**: A JSON-compatible value that survives a round trip exactly — no cycles, holes, symbol keys, class instances, or non-finite numbers — which is the form every durable and wire-carried payload must be in. _Lives in_: [`packages/util/values/`](../../../packages/util/values/README.md), [`docs/subsystems/typert.md`](../../subsystems/typert.md) (SRC inputs). _Avoid_: plain data, JSON object, serializable.

### Support tier

**support tier**: The maturity status that makes a package test infrastructure for other packages rather than a product contract; a package leaves the group once it gains a product contract and product consumers. _Lives in_: [`packages/test-support/`](../../../packages/test-support/README.md), [`docs/testing.md`](../../testing.md). _Avoid_: test utils, dev dependency.

## Defined elsewhere

- **seam** — defined once in the [root glossary](../../glossary.md#capability-seam); this context uses it for the credential, authorization, account, and Session-telemetry capabilities and adds no second sense.
- **gateway** — the Host-side dispatch that resolves descriptors against live services, and the access-control vocabulary around it, belong to the Remote assembly in `web-gui`; platform owns only the generated graph it consumes. See [`docs/api-gateway.md`](../../api-gateway.md).
- **session telemetry record** — the capture contract, its ledger and ops channels, and its sharing status live with the Session data plane; platform owns only the transport service a backend is built on. See [`docs/subsystems/session-telemetry.md`](../../subsystems/session-telemetry.md).
