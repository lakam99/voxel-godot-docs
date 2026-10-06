# Owned source publication cutover

## Outcome, baseline, and boundaries

Deliver the complete production section candidate through native compilation and
installation without repeatedly rebuilding immutable source proofs on Main.
This is one coordinated producer-to-consumer cutover, as requested by the user.
Keep smooth terrain, deterministic generation/RNG, gameplay collision,
interactions, save deltas, independent actors, and complete family coverage.
Do not change support bounds, readiness rules, deadlines, or frame budgets.
Do not add another world generator or renderer. The native C++ section compiler
and installation backend remain the production rendering destination.

Game: `codex/chunk-owned-world-rendering-migration`, HEAD `5ffac014`, current
worktree `outputs/voxel-biome-world-godot`. Substantial uncommitted migration work
and unrelated import churn predate this unit and must be preserved. Canonical
documentation is this separate repository. Prior unit evidence is recorded in
[the family cutover charter](family-scoped-source-generation-charter-2026-10-06.md).

Entry evidence: seven focused gates pass, including real Main generation parity.
The same-seed headed Main gate still fails readiness at 121.531 s: 60 ready source
jobs, 12 pending, 0 failed, and 0 native installs. Source sealing consumes 21.434 s,
retention 20.757 s, useful generation 7.468 s. Main seals a bundle twice solely to
rotate a transport lease; family/member consumers repeatedly reconstruct it.
All test processes are drained with authoritative zero-member evidence.

## Architectural decision and reference

Minecraft 26.2 `SectionCopy`, `RenderRegionCache`, `RenderSectionRegion`,
`SectionCompiler`, and `SectionRenderDispatcher` establish the reference:
capture owned immutable input, compile against that input, upload required layers,
then swap the accepted result. Do not copy its block mesher or fixed-neighborhood
assumption into our smooth terrain and procedural tree support closure.

The production source publication owns one canonical, deeply immutable value
payload and precomputed family/member lookup tables. It is derived from existing
source authorities. It is an ownership boundary, not a second source of truth.
Acquire its durable catalog lease before the final seal. Separate semantic
payload identity from transport leases; acquiring another consumer lease does
not rebuild source rows or alter family revisions.

The coordinated Main API is fixed before consumer edits:

```text
admit_ecology_source_publication(snapshot, catalog_lease_token, owner_kind, owner_key)
acquire_ecology_source_publication(publication_id, owner_kind, owner_key)
resolve_ecology_source_publication(lease_token, expected_world_id, expected_world_epoch)
ecology_source_publication_is_current(lease_token, expected_world_id,
  expected_world_epoch, expected_owner_receipt, expected_source_revision,
  expected_removed_projection_digest)
ecology_source_publication_local_is_current(view, lease_token)
ecology_source_publication_record_is_current(view, lease_token, record)
release_ecology_source_publication(lease_token)
```

Successful acquire/admit responses include `publicationId`, `leaseToken`, and
the exact retained `view`. The `ecology-source-publication-view/v1` view contains
the exact payload alias, owner receipt, content/source identity, frozen normalized
`familyResultsById`, `memberIndex`, precomputed member digests, support policy,
and family policies. Family entries preserve the established ready/pending/
failed result shape and direct row aliases. Resolve/acquire never reseal content.
Owner-only currentness does not replace local authority currentness: the local
facade checks fresh terrain, structure, removal, and catalog inputs at task and
installation boundaries. Per-record checks use retained exact aliases and the
member index. Independently supplied values may undergo fresh full admission,
but a copied wrapper/row cannot impersonate an existing publication proof.

Admission must prove owned value data, complete source identity, and exact
payload alias. An artifact ID, matching digest, shallow read-only container, or
an equal-looking copied Dictionary is insufficient. Follow the existing catalog
publication pattern: exact owner-issued payload identity plus current owner
receipt. Keep the canonical validating path for independently supplied values.
No unbounded global memo table and no metadata-only admission shortcut.

One validated publication view provides direct family slices and indexed
`(family, sourceId, sourcePartId)` membership. Deep semantic validation happens at
admission; consumers check the exact retained publication, its own live lease,
world/epoch, source revision and current authority receipt. Source changes,
replacement Main/catalog owners, terrain/structure/removal changes, and stale
worker completion reject the prior proof. Recheck these boundaries before
candidate publication and installation. Unchanged content alone never proves
that an earlier installation is current.

Narrow and wide requests retain separate immutable publications and stable shared
family revisions. Private generation cursors remain shared by the existing
source session. No actor/RNG, recipe, material, geometry, support, or save identity
changes are intended. Deferred content remains pending, never empty success.

## Complete path and lifecycle

1. Main captures current generation, catalog, terrain, structure, and removal
   inputs; its retained deterministic source session produces demanded families.
2. The producer constructs and seals one immutable family publication, obtains
   a durable owner receipt, and retains its exact aliases and membership index.
3. Adapter capture jobs acquire publication leases and reuse the view across
   families and member preparation. Index admission and tree compilation use the
   same accepted view; no per-row full-bundle reseal or transport-token deep copy.
4. SupportIndex retains required family coverage and unacknowledged tombstones.
   Tree queue retains publication ownership through async compilation and result
   consumers. Only owned value buffers cross native worker boundaries; Main
   references, Nodes, Resources, WeakRefs, Callables, and RIDs do not.
5. The existing complete section manifest/layers flow to native compile/install.
   The previous valid representation remains installed until current candidate
   identity, layers, frame/native receipt, and provider acknowledgments succeed.
6. Cancellation, reset, unload, owner replacement, and final receipt retirement
   release the exact consumer leases. Bounded cache eviction cannot invalidate
   actively retained demand. Do not move final large-payload destruction onto an
   unbudgeted gameplay frame; reuse existing retirement ownership.

## Stages, ownership, and evidence

HEAD owns architecture, integration, serialized Godot runs, final diff review,
documentation, explicit staging, and all acceptance decisions.

- Producer lead owns Main source capture, `EcologyProducerDomain`, and a composed
  publication owner if needed. Define exact shared API signatures with consumer
  lead before implementation; preserve existing snapshot schemas where possible.
- Consumer lead owns Adapter and SupportIndex integration and lease handoffs.
- Tree/review lead owns TreePublicationQueue/compiler integration, coordinated
  fixture updates, and independent review of producer/consumer changes. HEAD
  independently reviews this lead's production diff.
- No overlapping mutable scopes. No concurrent Godot tests or edits to a running
  gate's dependencies. Preserve all substantive fixture assertions.

**Stage A — coordinated implementation:** one seal and owner publication,
consumer view/lease adoption, currentness and retirement integrated together.
Entry: shared API fixed and dependency owners known. Proof: engine compilation
and exact final diff review; this alone does not advance gameplay acceptance.

**Stage B — focused falsifiers:** prove single sealing/admission despite multiple
family/member consumers; immutable alias integrity; forged/copied values rejected;
catalog/Main replacement, terrain/structure/removal revision changes, empty and
deferred families, cancellation, narrow/wide reuse, and complete lease drainage.
Retain real source-pass row/RNG/actor parity and all prior adapter/index/recipe
checks. Bounded telemetry reports seal/full-validation counts and time, alias
checks, publication/lease counts. Do not infer success from lower work counts.

**Stage C — actual renderer and live acceptance:** repeat the exact same-seed
Main gate and compare source completion and queue service timings. Require a
current complete production candidate compiled and installed through the native
renderer with settled provider acknowledgments. Then run live visuals/traversal,
interaction/edit/harvest, unload/replay/save, and representative performance gates
from the controlling migration matrix. Inspect screenshots and traces. Report
failed and untested rows separately; a focused proof cannot waive this exit.

### Next production unit: section-scoped immutable preparation

Minecraft 26.2 confirms the useful unit is a target section compiled from a
retained, immutable neighborhood capture, followed by staged render-layer upload
and an atomic visible swap. It does not prescribe our mesher. The project already
has a complete-section candidate assembler and native install session; this unit
removes the serial, source-by-source preparation bottleneck before those existing
owners. Do not replace deterministic producer authorities or publish individual
tree visuals beside the section candidate.

The captured section request must freeze a node-free value manifest containing:

- world identity/epoch, target section key, section request generation, coverage
  and roster revisions;
- census provider identities and revisions, source-domain/publication lease
  identity, and exact contributor/source revisions admitted for the section;
- tree rows from the existing canonical recipe authority, including recipe and
  LOD digests, stable source/member IDs, certified long-tree support closure,
  transform, biome/material/wind policy identities, and removal revision;
- terrain effective-volume and static contributor values only where their
  current authoritative capture APIs prove edited cells, durable deltas, scene
  overlays, and native pin/source identity are included;
- complete render-layer/batch compatibility policy, explicit empty results, and
  the producer completeness manifest required by the existing CandidateAssembler.

Capture and validate this closure on the main authority once per request. Worker
inputs own copied value arrays only; they contain no Node, Resource, RID, WeakRef,
or Callable. The first bounded production change batches all admitted tree recipe
rows for one section closure into a single cancellable preparation job. It keeps
`TreeSpawnService` authoritative for grammar, stable identity and LOD, and emits
section-local opaque/foliage layer data plus per-source ownership/support receipts.
It must cover the impostor tier as well as bole, branch and foliage roles. Mesh or
GPU resource creation remains on the renderer owner. Do not route through the
current `NativeTreeArtifactBuilder`, which regenerates partial recipe grammars.

Bind the job/result to the complete captured manifest digest and generation.
Reject cancellation, world/roster/provider/source/removal changes before worker
admission and again before candidate composition/install. Keep pending demand
retryable. A stale or incomplete result contributes nothing; the installed section
remains visible. Successful preparation must join terrain, buildings, trees and
props into the same complete render-layer candidate, including explicit empty
layers. Commit the replacement and retire the prior visual only after every
required native layer and provider receipt acknowledges the same current candidate.
Mobs/NPC simulation, tree gameplay bodies/collision/harvest authority, generation
RNG and save deltas stay independently owned.

**Preparation stage entry:** preserve the failed Main baseline and prove the
existing complete-candidate/install boundary, then validate the capture manifest
and queue backpressure/cancellation contract. **Preparation stage exit:** exact
source/actor/RNG parity; focused recipe-to-layer parity covering all three tree
grammars and near/mid/far/impostor tiers; stale/cancel/empty/retry checks; then the
same-seed Main gate shows batched jobs admitted, native compiles and real installs
completed, and settled provider acknowledgments. This is still not live acceptance.
**Renderer stage exit:** headed initial view and traversal, harvest/edit/unload/
replay/save checks, screenshots, and representative performance evidence. The
underground source stays pending until its immutable capture includes edited cells,
saved deltas, scene overlays and current native identity with script/native parity.

Commit coherent verified scope with explicit file staging and retain known failed
reports. Publish canonical documentation to main. No game push is implied.

## Integration evidence in progress

The coordinated producer, Adapter/Index, and tree consumer implementation is now
present. It retains publication aliases and consumer leases, removes the v1
intermediate seal, and keeps the existing native compile/install destination.
This is not stage-C acceptance.

- `node tools/run-project-compile-smoke.mjs` passed: MainMenu, Main, and the
  playtest runner load. Owned-process evidence:
  `artifacts/node-tools/process-runs/godot-ggVDIp/watchdog.json`.
- `node tools/run-ecology-producer-catalog-context-contract.mjs --OutputDirectory
  artifacts/citadel-runtime-integration/ecology-producer-catalog-context-owned-r4-20261006`
  passed 54 synthetic checks. This proves the named publication ownership/value
  contracts, not gameplay. The earlier r1/r2 compiler failures are preserved;
  their forced stops have authoritative zero-member evidence. R3 passed 53
  checks before the legitimate immutable `Rect2i` structure-bounds case was added.
- Real Main generation parity is not yet accepted. First run stopped at the
  spatial-source review guard because its already-reviewed compiler hash had
  not been updated. R2 then reached source admission and rejected the production
  structure dependency's `Rect2i` bounds. Both runs exited with cleanup and
  authoritative zero-member evidence. The owner whitelist now accepts that
  immutable integer rectangle; no resource/object admission was relaxed.
- Independent review identified a final local-revision freshness gap in support
  queries. Owner/lease checks alone are insufficient after terrain, structure,
  or removal changes. The query/acknowledgement checks are integrated and pass
  the focused Index gate; old installed visuals remain while replacement is pending.

The frozen-baseline parity command additionally compares exact 64-bit RNG state,
generated rows, actor digests, source/family revisions, and counts with the
passing family-unit r5 report. JavaScript comparison retains raw unsafe-integer
JSON literals rather than rounding them. Full serialized v2 bundle equality is
not claimed: transport lease data and the duplicate legacy `actorIntents` field
are omitted; canonical actor snapshot/digest and generated content are retained.

Remaining risk: large payload retirement still follows the existing synchronous
alias-release behavior. This unit has not established hitch-free teardown or
retirement. Live traversal, screenshots, interactions, save/reload, and runtime
performance remain untested against this coordinated cutover.

### Combined gate results

All paths below are relative to game `artifacts/citadel-runtime-integration/`.
Passing focused runs have exit 0, cleanup passed, and authoritative zero members.

| Gate | Report directory | Result |
|---|---|---|
| Producer catalog/publication contract | `ecology-producer-catalog-context-owned-r4-20261006` | 54 checks passed |
| Adapter | `ecology-section-value-adapter-owned-r2-20261006` | 88 checks passed |
| Support index | `ecology-world-support-index-owned-r2-20261006` | 68 checks passed, including local freshness and stale acknowledgement rejection |
| Tree family | `tree-source-family-coverage-owned-20261006` | 22 checks passed |
| Source owner discovery/replay | `ecology-source-owner-discovery-contract-owned-20261006` | 14 checks passed |
| Real Main generation service parity | `ecology-source-pass-slicing-parity-owned-r4-20261006` | 20 in-run checks passed; older-build digest comparison failed and remains unresolved |
| Headed tree compiler/adapter | `tree-recipe-section-compiler-owned-20261006` | 22/25 checks passed; fixture authority lacked newly required publication APIs; fixture migrated, rerun pending |
| Headed actual Main cohabitation | `main-section-cohabitation-gate-owned-20261006` | Failed initial readiness at 120.162 s; zero native compiles/installations |

The real Main command remains unchanged apart from the fresh output directory:
`node tools/visible-world/run-main-section-cohabitation-gate.mjs --OutputDirectory
artifacts/citadel-runtime-integration/main-section-cohabitation-gate-owned-20261006
--TimeoutSeconds 360 --StartupWaitSeconds 120`. Seed is
`ecology-main-retirement-stage5`, with `-SkipTutorial`. It completed with exit 1,
cleanup passed, and authoritative zero members. The loading gate stayed closed.
There were 61 ready source jobs, three pending, zero failed, 64 captured source
chunks, and 2,355 terrain-backed section demands. The three unfinished source
sessions were in `underground_props`; 60 narrow completed sessions retained their
detail cursor for possible wider demand. No renderer installation is proven.

Diagnostic totals: sealing 17.848 s, accepted-result retention 24.668 s, useful
generation 9.662 s, total capture 59.241 s; maximum queue service 3.235 s. These are
not controlled performance comparisons: approximately ten seconds overlapped an
isolated diagnostic-project materialization. The functional pending phases and
zero-install outcome remain valid evidence.

The baseline mismatch has not been waived. Actor digests, actor/source counts,
family counts, and exact RNG states match r5. The reviewed tree compiler source
hash contributes to the certified support policy and catalog identity, which
also appears in source rows. Reports retain only row hashes, so they cannot prove
unchanged transforms/recipes independently. A separate ignored source-service
replay using the retained prior compiler/queue and their original certificate
pin is being prepared; it must use the unchanged baseline comparison, and must
not be cited as production renderer or gameplay acceptance.

## Next coordinated cutover: preparation before complete-section admission

The user requested larger implementation steps. This unit therefore spans the
producer store, asynchronous recipe preparation, demand retirement, and final
candidate admission together. HEAD owns acceptance; subsystem leads have separate
mutable file scopes. The game branch remains at `5ffac014` plus the preserved
migration work; no reset, generated-import staging, or game push is authorized.

### Resolved prerequisite evidence

The headed tree compiler/adapter rerun
`tree-recipe-section-compiler-owned-r2-20261006` passed all 25 checks, nine batches,
and its real queue-to-adapter mapping. This is compiler integration evidence,
not ordinary-game visual acceptance.

The isolated prior-certificate replay passed all 20 source-service checks and the
unchanged exact baseline comparison. Command: `node
tools/run-ecology-source-pass-slicing-parity.mjs --ProjectPath artifacts/parity
--OutputDirectory artifacts/citadel-runtime-integration/ecology-source-pass-slicing-parity-r1
--TimeoutSeconds 180 --BaselineReport <absolute game path>/artifacts/citadel-runtime-integration/ecology-source-pass-slicing-parity-family-r5-20261006/report.json`.
The diagnostic copy uses current producer code and retained prior compiler/queue
plus their prior certificate pin. This isolates the earlier digest differences
to certified source identity: under the same certificate, row digests, family
revisions, actor digests, and exact RNG states match. It does not establish new
renderer installation or geometry correctness on its own. Both passing runs have
exit 0, cleanup passed, and authoritative zero members. Initial diagnostic launch
failed before process creation because the nested Windows log path was too long;
the copied project was moved to `artifacts/parity`, preserving the setup failure.

### Authority and complete data path

Main's deterministic source pass remains the generation authority. A Context
producer factory will own raw value capture, canonical sealing, the immutable
payload, its catalog hold, and indexed family/member views. The factory invokes
the Domain sealer once. External presealed admission retains full validation;
there is no caller-controlled trusted boolean or certificate shortcut. Payload
digests retain their current meaning. Owner, epoch, terrain, structure, removal,
family and source revisions remain binding before and after asynchronous work.

The adapter can submit an explicitly complete tree-family publication to the
existing recipe queue while other source captures remain pending. Each job is
owned by retained demand; shared demand survives one subscriber leaving, while
last-demand unload, supersession, or world reset detaches/cancels owned work and
releases aliases through the existing retirement owner. An obsolete completion
cannot replace a newer source job. Pending/backpressure retains retryable demand.

The final section candidate still includes the complete current terrain,
building, tree, and prop manifest and required render layers. Early preparation
does not permit partial installation. The existing native compiler/renderer must
acknowledge installed layers and required collision/interaction consumers before
the candidate is current. Old valid publication stays visible until replacement
succeeds. Mobs, source RNG order, durable save deltas and gameplay authority stay
independent of compilation scheduling.

Minecraft reference inspected again: `SectionCopy` copies section state once;
`RenderRegionCache` shares that immutable capture by section identity;
`SectionRenderDispatcher.checkSectionMesh` waits for every required vertex/index
upload before swapping and releasing the old mesh. Its cancellation boundaries
inform demand-owned retirement here. Our smooth terrain and procedural-tree
support bounds remain game-specific.

### Entry, proof, and exit

1. **Owned factory:** exact factory-versus-canonical output parity; one canonical
   seal on the producer path; strict external rejection of forged/stale/object
   inputs; independent catalog holds; alias, reset and local revision checks.
2. **Independent preparation and retirement:** with another family blocked,
   admitted trees prepare while final section readiness remains pending; shared
   demand, last-unload cancellation, backpressure, empty-family proof and stale
   completion rejection have focused evidence. No readiness bounds or timeouts
   are relaxed.
3. **Production admission:** repeat the same actual Main gate, require current
   complete native installation and owner acknowledgements, then inspect live
   visuals/traversal and performance. A failed Main gate keeps this unit open.

Underground applicability requires additional authority evidence: a player above
ground does not by itself prove that cave openings or nearby underground props
are invisible. No blanket family omission or fabricated empty result is allowed.
Section-local generation must preserve deterministic candidate ordering/RNG;
otherwise accelerate preparation at the terrain-volume authority. This is an
explicit remaining architectural dependency, not an excuse to bypass admission.

Read-only dependency discovery found an existing C++ implementation in
`native/world_backend/core/native_underground_prop_stream.cpp`:
`NativeUndergroundFloorScan::create` preserves the ordered first-floor scan,
candidate hash threshold and first-36 cap, and accepts cancellation. Its only
GDExtension entry point currently used by scripts is the synchronous
`compose_underground_prop_ordered_shadow`, which also builds recipes and explicitly
does not claim production cutover. Reuse that scanner if native preparation is
needed; do not invent another algorithm. Promotion requires an asynchronous
floor-artifact job, current `NativeEffectiveTerrainSource` pin, proven agreement
with script edits/scene-block/structure/fluid state, and fresh differential
evidence before substituting it for the live incremental scanner. Existing N4
probe evidence is source/recipe parity, not live installation or worker ownership.

### Coordinated preparation unit: focused verification

The producer factory and adapter demand lifecycle are implemented together.
The Domain sealer, source RNG/generation rules, tree geometry compiler, Index
production code, and final six-family completeness requirement are unchanged.
Independent review approved producer ownership and corrected adapter ownership
paths: consumer identities include Adapter instance identity; accepted consumers
are retained before later admission can fail; cancellation addresses the original
queue through its weak owner; queue replacement readmits retained demand. Ready
source work is primed before returning other pending or terminal source results.

All passing commands below use `--OutputDirectory
artifacts/citadel-runtime-integration/<directory>`. They ran serially, exited 0,
and report cleanup passed with authoritative zero members.

| Command | Directory | Named evidence |
|---|---|---|
| `node tools/run-ecology-producer-catalog-context-contract.mjs` | `ecology-producer-catalog-context-preparation-r2-20261006` | 57 synthetic ownership/value checks; single seal, unchanged semantic/content digests, copied producer data, strict external admission |
| `node tools/run-ecology-section-value-adapter-contract.mjs` | `ecology-section-value-adapter-preparation-r3-20261006` | 90 synthetic adapter checks; priming, sibling demand, queue replacement, stale rejection and last-demand retirement |
| `node tools/visible-world/run-ecology-world-support-index-contract.mjs` | `ecology-world-support-index-preparation-20261006` | 68 synthetic freshness/receipt checks; runner now hashes its actual Adapter fixture dependencies and checks final source stability |
| `node tools/run-ecology-source-pass-slicing-parity.mjs` | `ecology-source-pass-slicing-parity-preparation-20261006` | 20 real Main source-service checks; exact comparison against same-certificate `ecology-source-pass-slicing-parity-owned-r4-20261006/report.json` passed, including rows, revisions, actors, and lossless RNG; 13.470 s |

The parity command also used `--TimeoutSeconds 180 --BaselineReport
artifacts/citadel-runtime-integration/ecology-source-pass-slicing-parity-owned-r4-20261006/report.json`.
These results do not establish live startup, renderer installation, gameplay or
performance acceptance. The actual Main gate is the next required exit.

Preserved introduced fixture failures: Context first run had three untyped
Variant-derived boolean assertions; Adapter first run had one untyped Dictionary
assertion; Adapter r2 explicitly freed a RefCounted fixture and aborted its result.
Each was stopped by the owned watchdog (exit 126, cleanup false, authoritative
zero true). Fixture setup was also corrected to supply required publication
handles and the section/job reverse subscription. Assertions and production
admission requirements were retained.

### Actual Main result and next falsifiable check

`node tools/visible-world/run-main-section-cohabitation-gate.mjs --OutputDirectory
artifacts/citadel-runtime-integration/main-section-cohabitation-gate-preparation-20261006
--TimeoutSeconds 360 --StartupWaitSeconds 120` failed initial readiness at
120.182 s with the same seed and SkipTutorial. It exited 1 with cleanup passed
and authoritative zero. There were 61 ready sources, three underground scans
pending, zero native compiles/installations, and one failed section cohort.
The existing report omits that cohort's failure reason; no source job was failed.

The live factory boundary is confirmed: 61 producer seals, 61 trusted insertions,
zero external revalidations and no lingering transient alias. Retention/store work
totalled 2.139 s; canonical sealing 18.698 s; capture calls 41.741 s; maximum source
queue service 1.517 s. These are diagnostics, not accepted performance: loading
still failed and the earlier comparison run had overlapping file-copy activity.

Before another expensive Main run, extend the existing source-service parity
fixture to admit its naturally generated tree publication through the actual
TreePublicationQueue and release that exact consumer. This distinguishes a real
publication/queue admission mismatch from unfinished underground generation,
without invoking gameplay or claiming native installation. Keep all existing
parity assertions and the unchanged baseline comparator. The next Main report
will also include bounded failed-cohort reasons and source tree job summaries;
do not change timeouts or final readiness criteria.

Focused reproduction `ecology-source-pass-slicing-parity-queue-admission-r2-20261006`
failed only the strengthened actual compiler-admission check: real tree records
use `ecology.static_source_value.v1`, while the compiler required the synthetic
fixture's `ecology-tree-runtime-source/v1`. The queue returned queued even though
compiler progress was idle. Exit 1, cleanup passed, authoritative zero. The first
diagnostic checked only queue registration and passed; that is explicitly
insufficient evidence, so the assertion now requires an active compiler with
the exact generated tree record count.

Repair the consumer to accept the existing shared producer value schema while
retaining tree family/kind/recipe-version and all payload/provenance checks.
Migrate the synthetic tree fixture to that same contract. Queue admission must
verify an active compiler before retaining a nonempty job and preserve the actual
rejection reason when admission fails. This changes no generation or geometry;
the reviewed compiler source certificate must change, so derived policy/source
digests change intentionally. Keep the old reports and exact prior comparison;
do not mislabel different certificate digests as a generation regression or waive
the new active-compiler assertion.

### Integrated tree compilation and source coverage correction

The repaired admission gate passed 23 checks in
`ecology-source-pass-slicing-parity-queue-admission-r4-20261006` with exact
same-certificate source/actor/RNG comparison against r3. The actual compiler
contract and Adapter contract passed again (`tree-recipe-section-compiler-admission-20261006`,
`ecology-section-value-adapter-admission-20261006`). These are service/contract
results, not renderer acceptance. All exited zero with cleanup and authoritative
zero proven.

Main gate `main-section-cohabitation-gate-admission-20261006` (same command,
seed and 120-second readiness criterion) still failed. Cleanup and authoritative
zero passed. Unlike the prior failed cohort, tree compilation now starts:
38 jobs, 37 active and one complete, 159 recipe jobs, no recipe workers remaining.
Active jobs report `tree_source_family_envelope_unproven`; four underground scans
remain pending, with zero native section installations. Do not rerun unchanged.

The next coherent unit repairs source support ownership through actual compiled
output. Main's nominal canopy/height AABB describes a silhouette; it does not
cover trunk hull, roots or wind. The existing admitted catalog family envelope
already certifies those effects and drives inverse source closure. Follow
Minecraft's immutable source-region / derived compiled-geometry separation:
derive conservative source coverage from that same certified family policy,
retain nominal generation inputs, and require actual compiled geometry to fit
the admitted coverage. Do not widen tolerances, bypass bounds checks, invent
another policy, or change seeded recipes/RNG. Identity revisions and source-row
digests may change when their support proof changes; gameplay values must not.

Before production edits, extend the existing real-generated source fixture
through worker recipe completion and section compilation, recording the exact
declared-versus-actual bounds predicate. Preserve the failing baseline. Then
verify real-generated completion, existing adverse bounds/stale-owner coverage,
and the actual Main gate. Native installation, traversal, collision/interactions,
save/replay and performance remain outstanding stage exits.

HEAD owns fixtures, evidence and final acceptance. The producer lead owns only
Main's source-proof production path and any required Domain helper; independent
review owns no mutable files. Godot runs remain serialized with dependency edits
frozen. The proposed native underground scan is a later integrated unit: it must
include script overlay capture, native source parity and cancellable receipt
validation together before replacing the existing scan.

The real-generated completion baseline reproduced the bounds contradiction;
`ecology-source-pass-slicing-parity-real-compile-baseline-20261006` contains
both AABBs. The Main-only correction was independently reviewed and passed all
24 checks in `ecology-source-pass-slicing-parity-real-compile-headed-20261006`
(15.641 s, cleanup and zero proven). The intermediate headless run failed because
the factory intentionally produces no bole geometry with its dummy renderer;
the extended fixture now uses a real GL renderer, without claiming gameplay.
Across the preserved baseline and corrected run, actor digests and losslessly
read 64-bit surface/detail/underground RNG states match. Nominal geometry is
unchanged by the reviewed production diff; support-proof digests change intentionally.

Main `main-section-cohabitation-gate-tree-coverage-20261006` progressed into
tree geometry compilation but still failed startup. Its bounded cohort report
identified `ecology_tree_source_compile_artifact_unsealed`. Both empty and
nonempty queue completion paths discard the return value of the copy-producing
`_freeze_section_value` helper. The next correction must retain that returned
owned artifact in both paths, keeping consumer seal checks unchanged. Extend
the real-generated completion assertion to check the artifact's recursive
container immutability, and cover authoritative empty completion explicitly.
Review the complete queue-to-adapter result boundary before another Main run.

The complete boundary review also found two Adapter integration errors to repair
as part of this unit: manifest assembly replaces a compiler world transform with
the producer's chunk-local transform, and contribution polling registers an old
chunk-only consumer token outside the section-demand retirement owner. Preserve
and validate the compiled world transform against chunk origin plus producer
placement; poll only under retained demand ownership. Prove a nonzero-chunk
source remains correctly located through support-index admission, and prove
contribution followed by last-demand release leaves no orphan queue consumer.
The producer lead now owns Adapter and its existing contract fixture only; HEAD
owns queue/real-generated fixtures and documentation; review remains independent.
All other publication identities, currentness checks and source closure remain
required. The sealed-output real-generated fixture passed 25 checks and exact
same-proof baseline comparison; this still does not accept the Adapter boundary.

The corrected manifest binder now preserves and validates the compiler's world
transform, and contribution looks up the existing section demand by capture
identity before polling its original queue/consumer. Independent review approved
both changes. `ecology-source-pass-slicing-parity-support-boundary-r2-20261006`
passed 26 checks in 16.427 s, including two naturally generated trees and 1,586
compiled members through the actual manifest binder, support projection and
Index row validation at chunk `(-4,-4)`. This is direct service evidence, not a
complete census/contribution/native-install or gameplay claim. Its unchanged
baseline comparator passed against `ecology-source-pass-slicing-parity-sealed-output-20261006`.

Preserve the first support-boundary report: all 26 in-run checks passed, but its
cross-run comparison failed. Adding an early Adapter preload changed process-local
owner IDs and therefore the owner-bound catalog artifact ID, despite identical
catalog content digest, source revision, actors and RNG. Loading that downstream
probe after Main's catalog capture restored exact comparison without changing
production identities or excluding comparison fields.

The wider read-only retirement review found a remaining separate production
defect: canonical `_latest_by_section` is keyed by source/member identities,
while legacy prepared-tree retirement looks those keys up as raw source IDs.
Moreover canonical compiled and legacy prepared revisions have different
contracts. The next retirement unit must explicitly bind the real prepared owner
to canonical source/member identities and require the complete owned-section
receipt set. Decoding keys alone is insufficient. Do not report legacy tree
retirement or duplicate-free mixed representation as accepted until live replay
proves that association and retirement.

Adapter gate `ecology-section-value-adapter-boundary-r4-20261006` passed all
93 checks with cleanup and authoritative zero. The new synthetic contribution
case reaches the actual Adapter contribution path, polls only the retained
section token, removes that token on unload and refuses to poll after release.
It uses an explicitly synthetic support index/queue; it does not prove live
renderer retirement. Earlier fixture-only failures are preserved: r1 had typed
stub/boolean parse errors, r2 had a parent return-signature mismatch (both exit
126, cleanup false, authoritative zero); r3 supplied inconsistent census and
query coverage digests (exit 1, cleanup and zero passed). The fixes align fixture
inputs/types with production contracts; assertions and production gates remain.

The next integrated run is `node tools/visible-world/run-main-section-cohabitation-gate.mjs
--OutputDirectory artifacts/citadel-runtime-integration/main-section-cohabitation-gate-integrated-boundary-20261006
--TimeoutSeconds 360 --StartupWaitSeconds 120`, seed
`ecology-main-retirement-stage5`, SkipTutorial, actual Main/Forward+.

That integrated run failed the 120-second readiness criterion, exited 1, and
passed cleanup with authoritative zero. The previous source-cohort contract
failures no longer appeared: zero failed cohorts, 61 complete source captures,
three pending underground scans. Tree preparation had 56 jobs (55 active, one
complete), 246 recipe jobs and one recipe worker. The concrete pending tree
reason was `tree_section_compile_in_progress`. Native completed compiles and
installed candidates both remained zero. This is not live renderer acceptance.
Maximum observed source queue service was 1,133.455 ms; section admission was
1,576.602 ms. These are named section timings, not a measured worst-frame claim.

Do not repeat this unchanged integrated run. The next larger preparation unit
must address both expensive source discovery and tree geometry preparation,
following Minecraft's retained region inputs, independent compile workers and
complete upload/swap boundary. The native underground scanner needs the explicit
script-overlay/native-source parity bridge described above; an existing shadow
native page alone is insufficient. Existing `native_tree_artifact` also exposes
pending broadleaf/savanna recipes, so it cannot replace the game's full grammar
unchanged by assertion. Prefer native geometry preparation from the already
admitted recipe values, preserving the game's generated recipe authority,
with exact geometry/material/wind/ownership parity before promoting it.
Retirement requires the separate exact member/owner receipt association already
recorded. Full production migration, live traversal, gameplay/save parity and
performance gates remain open. No stage is advanced by these focused passes.
