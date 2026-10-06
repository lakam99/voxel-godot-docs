# Owner-published catalogs: production cutover charter

Status: implementation entry; the parent rendering migration remains incomplete.
Game branch: `codex/chunk-owned-world-rendering-migration`, latest committed base
`ff39dd5f`, with pre-existing broad migration changes and generated/import churn.
Preserve all unrelated work. Canonical parent:
[section-owned rendering charter](section-owned-world-rendering-charter.md).

## Outcome and evidence

Normal startup and traversal must reuse complete immutable producer catalog
publications across frames. Section jobs consume those publications alongside
current local terrain, structure and durable-removal inputs, then install complete
native section candidates. This is the next complete ownership/API cutover; a
per-frame cache or trusting counters on publicly mutable data is not the design.

Baseline: `main-section-cohabitation-gate-catalog-v2-descriptor-20261006/report.json`
on dirty `90141c2c` sources (the independent descriptor subsystem was subsequently
committed as `ff39dd5f`). Same seed `ecology-main-retirement-stage5`, tutorial
skipped, startup observation bound 120 seconds. Startup remained pending at
120.485 seconds: 394 full catalog captures, 203 pending source jobs, 1,337 aggregate
section demands, zero native candidates/compiles. No terminal source errors;
functional exit 1, clean owned-process cleanup and authoritative zero. Exact
commands and earlier failures are in
[the integration report](native-compile-cutover-2026-10-06.md).

The support bridge is already one-hop. Do not change its closure based on the
aggregate demand counter: published terrain sections and direct support owners
share that counter. Minecraft 26.2 `RenderRegionCache` shares immutable
`SectionCopy` values across its fixed region; `SectionCompiler` reads those values,
and `SectionRenderDispatcher` retains the old mesh until replacement uploads and
installation finish. Apply those ownership boundaries, without copying Java or
replacing our smooth terrain mesher.

## Authority and complete data path

1. `BiomeEnvironmentCatalog` owns validated profile policy. Setup/reload and
   explicit replacement APIs stage and validate candidate data, then atomically
   publish frozen value data plus semantic content digest and runtime receipt.
   Existing Resource-based access must return defensive copies; hot production
   consumers move to immutable profile values. No borrowed mutable Resource is a
   publication authority.
2. `VisualAssetRegistry` owns manifest/family/disabled data, scene ownership, static
   descriptors and rock-support policy. `AnimatedAssetRegistry` owns its manifest,
   SceneState descriptors and scene ownership. Sealing happens at setup or explicit
   mutation/reload boundaries. PackedScenes/resources remain with the registry;
   worker-facing publications contain owned values only. Prevent public mutation
   aliases, or detect/reject invalidation before a publication can be reused.
3. Each owner exposes a published snapshot and receipt. Its receipt binds owner,
   generation and actual accepted resource ownership. Its semantic digest excludes
   owner/instance IDs. Equivalent content replacement preserves semantic identity
   while rejecting earlier ownership. Failed replacement must not make a partial
   publication authoritative.
4. Main composes the current owner publications with explicit detail/settings
   policy into one leased catalog artifact. Unchanged owner publications reuse it
   without rescanning profiles, SceneState or support envelopes. Composition is
   rebuilt when any actual authoritative input changes; no arbitrary frame timer.
5. Main/Adapter/Queue/Compiler/Index use that artifact through the existing lease
   boundaries. Source-local terrain revision, exact structure dependency closure,
   removal projections and source identities remain independently validated.
   Catalog reuse does not waive stale worker rejection or native installation and
   provider acknowledgement checks.
6. Retirement releases source, section, worker, idle-cache and world leases.
   Bounded stores must retain active demand yet retire expendable idle publications
   under pressure. No deadlock when more than eight catalog generations have
   existed. Reset drains owners and invalidates old receipts.

Preserve seeded draws and static generation results. Mobs/NPC simulation remains
independent. The existing wildlife/static pass still shares presentation-dependent
RNG consumption: do not swallow an actor descriptor failure or silently change the
RNG stream in this ownership unit. Separating that readiness is an explicit open
parent-migration requirement. Save format, terrain generation, NPC routing,
render radius and gameplay authorities are non-goals here.

## Accountable scopes and sequential gates

HEAD owns API agreement, stage advancement, final diff review, runs and scoped
commits. Catalog lead owns the three registries, snapshot capture helpers,
non-Main profile readers, explicit mutation APIs and their focused contracts;
it may delegate profile ownership/readers in a disjoint child scope. Integration
lead owns Main catalog composition, Domain/Context/Adapter/Queue/Compiler/Index
consumers, lease retirement, telemetry and their fixtures. Coordinate exact
snapshot schemas before edits; no overlapping mutable files.

1. **Owners and consumers together.** Map every external read/write and migrate
   it; inventory evidence has found production writes confined to setup/reload,
   with direct public mutation largely in fixtures. Migrate fixtures to admitted
   mutation APIs while preserving their substantive stale/mutation assertions.
   Raw alias mutation must either be isolated from authority or invalidate it.
2. **Focused verification.** Actual profile/asset parity; unchanged acquisitions
   perform zero additional seals/scans; explicit mutation invalidates; equivalent
   owner replacement retains content digest but rejects old receipts; invalid
   reload is atomic; leases/reset/pressure remain retryable and bounded. Compile
   the production project and preserve all existing section identity, support,
   tombstone and renderer-install assertions.
3. **One real Main gate.** Same controlled seed and tutorial setting, owned
   watchdog. Report owner seal/scan counts, composition/reuse counts, original
   view sections, terrain-backed demands, direct support-only sections, and unique
   source domains/jobs separately. Require actual candidate/native install progress,
   then current terrain/building/ecology cohabitation and settled acknowledgements.
   Stop on terminal failure; inspect the report before any repeat. No loading
   timeout bypass or empty-success substitute.
4. **Parent exits remain required.** Live visual/traversal, harvesting and edits,
   unload/replay, save/reload and representative frame performance. Focused owner
   checks or a first native receipt do not complete the full migration.

## Independent review criteria for the combined cutover

- Repeated acquisition must not deep-copy or hash catalog contents or full
  resource-binding receipts. Compare genuine frozen owner publications and small
  owner/world/settings identities; compose and seal only after a replacement.
  Copied or forged payloads with unchanged claimed digest/receipt remain rejected.
- A failed reload reports failure while retaining the prior valid publication.
  It must not clear a prior invalidation, resurrect stale ownership, or let a
  retired resource's signal invalidate the newly accepted publication.
- Resource ownership includes relevant nested scene and animation dependencies.
  A signal on the root PackedScene alone does not establish that changes inside
  an AnimationLibrary or other nested Resource will invalidate its descriptor.
  Prove isolation or invalidation at the actual dependency mutation boundary.
- Legacy profile access returns defensive Resources; production value reads use
  recursively immutable scalar/Array/Dictionary data. Preserve direct profile
  parity and seeded draw assertions while migrating old borrowed-alias tests to
  explicit owner mutation and replacement contracts.
- Bind the shared catalog artifact's runtime identity into ecology's existing
  provider `authorityRevision`, including empty sections. Keep semantic source
  revisions unchanged. Existing census validation then rejects an old candidate
  after equivalent-content owner replacement. Delayed provider acknowledgements
  must also revalidate the census before retiring visuals; pending capture retains
  demand, and changed authority requests replacement while keeping the old slot.
