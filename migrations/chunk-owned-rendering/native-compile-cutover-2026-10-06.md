# Native section compilation cutover — 2026-10-06

Status: implementation and focused verification in progress; no migration stage
exit is claimed. Game branch `codex/chunk-owned-world-rendering-migration`,
base HEAD `446f58ea365ddac9cb6b6d493cbdb5d1c33509d9`, with pre-existing migration
edits and import churn preserved. Per-run launch records bind tested source and
DLL hashes. The controlling plan remains [the architecture charter](section-owned-world-rendering-charter.md).

## Complete implementation unit

Source-domain capture feeds a complete section census. The assembler prepares
immutable contributor groups without merging buffers on Main. A world-owned
`NativeSectionCompileDispatcher` copies numeric/string values into C++ jobs,
compiles compatible static-instance batches on two workers, and returns results
for Main-thread resource binding and native renderer installation. Smooth terrain
meshing and producer-specific geometry generation remain their existing paths;
this dispatcher compiles their admitted instance groups, not their SDF/mesh recipes.

The coordinator rechecks census, generation and capture identity after completion.
Cancellation is qualified by world, epoch, section and generation, including
already-ready results. Shutdown joins workers. Old installed representations stay
until the replacement's existing installation/presentation acknowledgement.
Install scheduling rotates pending sections. Source capture has fair dispatch,
exact subscriber wake/detach, immediate superseded-payload release, and a bounded
128-entry idle cache. Gameplay collision, interactions, saves and actors retain
their authorities.

Reference decisions came from local Minecraft 26.2 `RenderSectionRegion`,
`SectionCompiler`, `SectionRenderDispatcher`, and `SectionTaskDynamicQueue`:
stable capture, whole-section output, cancellable owned jobs, current camera
selection and explicit publication lifetime. No Minecraft implementation copied.

## Evidence

All output paths below are relative to the game repository. Each directory has
`report.json`, `launch.json`, and `watchdog.json`.

| Check | Command | Result and boundary |
| --- | --- | --- |
| Native build | `node tools/build-native-terrain-meshing.mjs` | Passed; Windows debug DLL rebuilt and installed. |
| Native worker | `node tools/visible-world/run-native-section-compile-dispatcher-contract.mjs --OutputDirectory artifacts/citadel-runtime-integration/native-section-compile-dispatcher-20261006-r3` | 16/16. Actual worker with synthetic inputs: candidate parity, batching, copy isolation, identity rejection, backpressure, ready cancellation, epoch isolation, drain. |
| Source lifecycle | `node tools/run-ecology-section-value-adapter-contract.mjs --OutputDirectory artifacts/citadel-runtime-integration/ecology-section-value-adapter-async-cutover-20261006e` | 75/75. Synthetic source/census, tombstone/reappearance, resources, queue fairness and retirement. |
| Coordinator | `node tools/visible-world/run-visible-section-demand-driver-contract.mjs --OutputDirectory artifacts/citadel-runtime-integration/visible-section-demand-driver-native-20261006e` | 57/57. Synthetic demand scheduling, actual native compile/install and translucent replay. The real frame-presentation subcase requires a headed runner. |
| Native installation | `node tools/visible-world/run-native-section-presentation-lifecycle.mjs --OutputDirectory artifacts/citadel-runtime-integration/native-section-presentation-lifecycle-worker-cutover-20261006b` | 17/17. Headed synthetic producers, actual worker and actual native renderer: installation, replacement retention, invalidation, explicit empty, acknowledgement and teardown. Does not prove candidate-specific visible pixels. |
| Project compilation | `node tools/run-project-compile-smoke.mjs` | Passed; watchdog `artifacts/node-tools/process-runs/godot-OKBrxT/watchdog.json`. Does not compile every fixture or prove gameplay. |

The four named contract runs ended with functional exit 0, `cleanupPassed=true`,
`authoritativeZeroProven=true`, and zero final owned processes.

Earlier native r2 exposed duplicate-segment validation scoped too broadly across
batches and an outdated fixture census digest; both corrected before r3. Ecology
c/d exposed fixture typing errors, corrected before e. The first headed lifecycle
run installed successfully but its admission-count assertion still read fields
from synchronous admission; the fixture now reads actual accepted candidate
coverage and manifest counts. Earlier failures remain in their separate artifacts.

## Open acceptance

The real Main gate ran with `node tools/visible-world/run-main-section-cohabitation-gate.mjs --OutputDirectory artifacts/citadel-runtime-integration/main-section-cohabitation-gate-native-cutover-20261006 --TimeoutSeconds 960 --StartupWaitSeconds 300`.
It failed startup after 304.5 seconds, still in `terrain_view_expansion` with
1,692 pending section demands, zero native compiles and zero installed candidates.
The last step was 9,893 ms; ecology's reported source capture remained queued.
Functional exit was 1, but cleanup passed with authoritative zero processes.
This is a current production failure, not a native-worker throughput result;
the earlier baseline used a longer observation window and cannot establish a
performance improvement/regression by itself.

### Next producer-context gate

Native dispatcher source was committed independently as game `b62a71d2`; the
production bridge and producer integration remain uncommitted and incomplete.

The measured diagnostic used `node tools/visible-world/run-main-section-cohabitation-gate.mjs --OutputDirectory artifacts/citadel-runtime-integration/main-section-cohabitation-gate-source-profile-20261006 --TimeoutSeconds 360 --StartupWaitSeconds 120`.
It recorded 634 catalog-input captures, maximum ecology census admission
11,112.271 ms, maximum catalog-input build 1,325.089 ms (rock envelope
1,282.78 ms), and maximum installation service 0.088 ms. This is preparation
cost before native dispatch, not native compiler throughput.

The follow-up `node tools/visible-world/run-main-section-cohabitation-gate.mjs --OutputDirectory artifacts/citadel-runtime-integration/main-section-cohabitation-gate-source-failures-20261006 --TimeoutSeconds 360 --StartupWaitSeconds 60`
identified all four sampled source-job failures as
`ecology_structure_admission_revision_stale`, incorrectly terminal for their
queued identities. Both diagnostic runs failed startup, exited functionally 1,
and proved cleanup with zero final owned processes. Their windows differ, so
these are causal diagnostics, not before/after performance comparisons.

The implementation unit now includes both shared catalog capture and source
revision supersession. One accountable lead owns catalog capture/policy; another
owns source scheduling, callback currentness and retry; HEAD owns integration
and acceptance, with a separate read-only reviewer. Capture sharing must have
an explicit synchronous lifetime or authoritative invalidation boundary; no
revision-only cache may conceal unrevisioned resource changes. Stale queued
inputs must be retired and current inputs recaptured, without dropping section
demand or endlessly retrying the obsolete immutable input. No gameplay,
generation, collision, navigation or save authority changes are in scope.

Chosen initial context lifetime: one synchronous provider census or one bounded
source service call, with no yielding inside that scope. Capture semantic
catalog values once on entry, reuse them for every inverse source chunk, and
release the context at exit through a wrapper that covers all early returns.
Later scopes capture fresh values, including same-revision resource changes.
Policy memoization keys the shared catalog values, while domain identities
retain exact local revisions. A persistent world-lifetime context is deferred
until catalog mutation boundaries can authoritatively invalidate it.

Minecraft lifetime cross-check: `LevelExtractor` creates `RenderRegionCache`
for one section-update extraction pass and shares it across dirty visible
sections. `SectionCopy` copies non-air state containers, and region objects keep
those copies alive for dispatch. The cache is not world-lifetime. Its Java graph
also retains live level/light/block-entity references; our C++ worker boundary
continues to require owned value data rather than copying that object model.

Review identified two remaining dependency boundaries. The existing structure
revision includes global publication queue/count changes, so even distant
structure activity can supersede a source pass. It remains conservative until a
separate token proves the exact exclusion/footprint, town-input and citadel
reservation closure, including post-draw tree margins. Also the canonical prop
pass emits wildlife actor intents and consumes presentation-dependent RNG;
animated catalog identity must remain bound while that is true. Actors are not
installed by the static renderer, but their producer inputs are not fully
independent of static capture yet. Separating those producers requires an
explicit deterministic RNG/parity decision; removing the identity alone would
be incorrect.

Producer-context implementation verification (dirty game base `b62a71d2`):

- `node tools/run-project-compile-smoke.mjs` passed; owned report
  `artifacts/node-tools/process-runs/godot-sLcMYt/watchdog.json`.
- `node tools/run-ecology-producer-catalog-context-contract.mjs --OutputDirectory artifacts/citadel-runtime-integration/ecology-producer-catalog-context-20261006`
  passed 22/22 synthetic value ownership, scope identity/lifetime and policy
  digest checks. It does not prove real catalog reuse or gameplay.
- `node tools/run-ecology-section-value-adapter-contract.mjs --OutputDirectory artifacts/citadel-runtime-integration/ecology-section-value-adapter-catalog-context-20261006c`
  passed 81/81 synthetic lifecycle and existing contribution checks. Added
  checks cover stale recapture fan-out, exact partial-state retirement on
  supersession/release/reset, retained completed cache and idle scope avoidance.
  The earlier first attempt had an explicitly typed boolean missing in its new
  fixture; the second exposed reordered missing-authority diagnostics. Both
  were corrected without relaxing assertions. The first was stopped by the
  owned error monitor (cleanup false, authoritative zero true); the second
  failed functionally with clean teardown.

Both successful contract watchdogs report functional 0, cleanup true,
authoritative zero true and empty final membership. The owned headless editor
import at `artifacts/catalog-context-import-20261006/watchdog.json` also passed
and generated the new script UID sidecars. It reported the existing nested
`artifacts/vt/b/98399ab4-25f/source/project/project.godot` project as ignored.
Production Main replay remains a separate gate.

Game commit `90141c2c` preserves the verified catalog-context value/lifetime
component, policy cache, static-member identity helpers, focused runner and UID
sidecars. Main/adapter production integration is still in the working tree.

The replay `node tools/visible-world/run-main-section-cohabitation-gate.mjs --OutputDirectory artifacts/citadel-runtime-integration/main-section-cohabitation-gate-catalog-context-20261006 --TimeoutSeconds 360 --StartupWaitSeconds 120`
failed startup at 122.98 seconds. Compared with the same-seed 120-second
diagnostic above, full catalog input captures fell from 634 to 124 and maximum
ecology admission from 11,112.271 to 5,268.936 ms. These are individual diagnostic
runs, not a statistical performance gate. Native compilation/installation
remained zero. There were no terminal source failures; stale jobs correctly
reported recapture. The final four serviced jobs were invalidated solely as the
global generated-structure count changed from four to five, with their local
town/citadel values unchanged. Functional exit 1, cleanup true, authoritative
zero true, empty final membership.

### Next complete producer-admission unit

Entry evidence is the replay above. Fix the producer dependency boundary as
one unit: (1) replace ecology's broad structure publication revision with a
semantic dependency closure for its actual source chunk and declared post-draw
tree margins; (2) carry one owned catalog artifact through the source batch,
using compact, validated identity at repeated source-job boundaries instead of
rehashing/deep-copying the same catalog for each inverse source. Preserve fresh
semantic capture between batches and reject forged or mutated provenance.

The structure closure must include local natural exclusions, terrain footprints,
admitted citadel reservations and generations, deterministic nearby town inputs,
and pending/final state needed to know those records are complete. Distant queue
indices and global generated counts are not ecology facts. Do not replace the
existing general publication/navigation revision; add an ecology-specific
contract and migrate every ecology consumer together. Preserve world generation,
RNG, local edits, collision/navigation and old-render-slot authority.

Acceptance: local changes invalidate and restart a whole source pass; unrelated
structure scheduling does not; missing/undecided local admission remains pending;
same-revision catalog mutation and forged digests are rejected; partial state is
retired; immutable input parity remains exact. Then repeat the real Main gate
and require native candidate progress before advancing to traversal/performance.
Do not repeat the unchanged costly run as evidence of progress.

Ownership for this unit: ecology lead owns the compact catalog artifact and
Main/adapter/domain consumers, and may delegate support-index/tree-compiler
consumers with separate mutable scopes. Structure lead owns only the additive
StructureSystem closure API, its composed helper and focused contracts. HEAD
owns integration, independent diff review and acceptance. The artifact resolver
must be per-world, hold explicit lifetime ownership through source jobs,
snapshots/index and asynchronous tree admission, deduplicate only freshly
validated equal content, retire unleased entries with a bound, and drain on
reset. Scope end cannot evict a retained proof. Unknown compact identities fail
closed; the direct full-input form remains strictly validated for existing
explicit synthetic contracts. No source/actor/navigation authority moves.

The context component and measurements above are entry evidence for this unit,
not a separate future milestone. Implement the producer, compact-input consumers,
local structure closure, and lease/reset lifecycle together before freezing the
integrated source for verification. Run the focused ownership/locality contracts
and compilation checks first, then the real Main gate with the recorded seed.
The next accepted milestone requires a current candidate to reach the native
renderer; a contract-only result cannot advance it. Keep the existing bounded
capture/service/census timings in the report to distinguish eliminated repeated
work from deferred work.

Minecraft 26.2 reference recheck: `RenderRegionCache.createRegion` shares section
copies across a capture batch; `SectionCompiler.compile` consumes that region
and emits one result containing its render layers. Apply those ownership and
completion boundaries here. Our producer catalogs and arbitrary tree supports
require explicit shared artifacts and inverse-domain completeness, while smooth
terrain retains its own mesher. No Java implementation is copied.

The first isolated v2 gate passed on the dirty `90141c2c` game base:
`node tools/run-ecology-producer-catalog-context-contract.mjs --OutputDirectory artifacts/citadel-runtime-integration/ecology-producer-catalog-context-v2-20261006`.
Its 41 synthetic assertions cover owned immutable catalog values, fresh-content
deduplication, mutation without a revision change, unknown/forged lease rejection,
retention under eviction pressure, retry after pinned-capacity backpressure,
scope-token ownership, nested-scope reset, world replacement, and stable semantic
source revision with distinct runtime cache identity after owner replacement.
The runner checked that its recorded source hashes remained unchanged. Watchdog:
functional 0, cleanup true, authoritative zero true, no final members; completed
2026-10-06 07:29:42 UTC. Report:
`artifacts/citadel-runtime-integration/ecology-producer-catalog-context-v2-20261006/report.json`.
This is value/lifetime evidence only; it does not establish Main startup,
renderer installation, visual correctness or performance.

The compiler source guard now records
`420b4703562065ea6b4432c6696913a122ef15226138e0095205552e263b9596` after review
of the compact-provenance and lease changes. Acquire/resolve, partial-admission
cancellation, currentness revalidation, and terminal release were reviewed;
three policy lookups now use admitted compact inputs with the resolved artifact
instead of deriving policy again from the full catalog. Geometry transforms,
recipe generation, wind/support mathematics, partitions and contributor
ownership were not changed by this unit. The child's starting dirty source was
not separately saved, so its scoped-change report and HEAD's final code review
are the provenance for this update; the complete working-tree diff and later
compiler/visual gates remain required. No guard was bypassed.

Integrated adapter verification passed 81/81 assertions:
`node tools/run-ecology-section-value-adapter-contract.mjs --OutputDirectory artifacts/citadel-runtime-integration/ecology-section-value-adapter-v2-typesfix-20261006`.
The source manifest stayed stable; functional exit 0, cleanup true, authoritative
zero true, no final members. This preserves source/part identity, source revision,
stale capture, ownership replacement, exact tombstone/retirement, support census,
material and partition assertions through compact v2 admission. It is a synthetic
adapter contract, not native installation or gameplay acceptance.

Integration checks exposed introduced compile errors in the new SupportIndex
continuation/type annotations, Main owner-identity inference, StructureSystem
footprint sorting/type annotations, and updated fixture signatures. These are
integration defects, distinct from the recorded startup baseline. Failed run
directories are retained under `artifacts/citadel-runtime-integration/` with
`v2` names. The general compile runner stops on first script error; its forced
stop reports cleanup false even though authoritative Job Object zero is proven.
An owned editor check also exited 0 with clean cleanup but logged Main's unresolved
parent script: that exit is explicitly not compilation success. Direct owned
`--check-only` and focused fixture loads isolate the dependency errors before a
long Main run is allowed.

Additional integrated checks passed on the same dirty game base:

| Gate | Evidence | Result |
| --- | --- | --- |
| `node tools/run-project-compile-smoke.mjs` | Main scene, menu, and PlaytestRunner load | Passed; watchdog `artifacts/node-tools/process-runs/godot-aXthu6/watchdog.json` |
| `node tools/visible-world/run-ordinary-section-geometry-adapter-contract.mjs --OutputDirectory artifacts/citadel-runtime-integration/ordinary-section-geometry-adapter-ecology-v2-mainfix-20261006` | Synthetic producer geometry and local structure dependency closure | 39 checks |
| `node tools/visible-world/run-tree-recipe-section-compiler-contract.mjs --OutputDirectory artifacts/citadel-runtime-integration/tree-recipe-section-compiler-catalog-v2-typesfix-20261006` | Headed synthetic compiler/queue/adapter value integration, exact support and budget slicing | 25 checks, 9 batches, 1806 work units |
| `node tools/visible-world/run-ecology-section-support-coverage-contract.mjs --OutputDirectory artifacts/citadel-runtime-integration/ecology-section-support-coverage-v2-admissionfix-20261006` | Synthetic distinct source/part support and footprint replacement proof | 7 checks |

All four exited functionally 0 with clean cleanup and authoritative zero. The
three focused source manifests were independently checked unchanged after exit.
These are compile/service contracts; the headed compiler fixture is not live
gameplay acceptance.

The v2 SupportIndex gate ran all 27 assertions but failed its existing requirement
that superseded revisions A and B remain demanded until the installed C receipt.
Report: `artifacts/citadel-runtime-integration/ecology-world-support-index-v2-20261006/report.json`.
The dirty index implementation filtered tombstones to old supports outside the
replacement footprint, losing same-section revision history before installation.
This is a production lifetime defect found during integration; the assertion
remains required. Restore receipt-bound old-revision retention before the real
Main gate. Minecraft `SectionRenderDispatcher.checkSectionMesh` likewise retains
the old installed mesh until every applicable vertex/index upload has completed
and the replacement is installed.

After restoring history retention, the SupportIndex gate passes 35 checks:
`node tools/visible-world/run-ecology-world-support-index-contract.mjs --OutputDirectory artifacts/citadel-runtime-integration/ecology-world-support-index-v2-retentionfix-20261006`.
Re-running the adapter then exposes four integration failures, including
`invalid_or_current_static_source_removal`. The adapter was projecting every
historical tombstone into the current candidate removal roster. A candidate
cannot both contain current C and remove the same source/part pair for historical
A/B. The coherent repair keeps all revision-specific old receipt obligations in
the index while generating current removals only for pairs actually absent from
the new candidate, coalescing multiple retired revisions deterministically.
Coordinator absence proof already checks exact source revision, so current C can
prove A/B absent without confusing their identities. This second adapter run is
failed evidence, not waived by the earlier 81-check pass; the repaired integration
must pass before Main acceptance. The final history/current-content split passed
all 81 adapter assertions in
`artifacts/citadel-runtime-integration/ecology-section-value-adapter-v2-historysplit-20261006/report.json`
(same runner, output directory named for this report; stable sources, functional 0,
clean cleanup, authoritative zero).

The next real Main check ran:
`node tools/visible-world/run-main-section-cohabitation-gate.mjs --OutputDirectory artifacts/citadel-runtime-integration/main-section-cohabitation-gate-catalog-v2-20261006 --TimeoutSeconds 360 --StartupWaitSeconds 120`.
The recorded comparison seed remains `ecology-main-retirement-stage5`, tutorial
skipped. It reached terminal source failures `animation_library_state_unavailable`
and no native compiled/installed candidates. HEAD requested an owned stop after
the failure was identified. The watchdog proves zero remaining Job members;
`cleanupPassed=false` records the forced diagnostic stop, not a successful run.
Progress and `diagnostic-summary.json` retain the source failures and timings;
there is no successful final gameplay report. Maximum ecology admission was
1500.707 ms, source service 398.817 ms, catalog capture 1425.194 ms cold and
193.697 ms latest. These failed-run observations are not performance acceptance.
The next falsifier inspected actual imported AnimationPlayer SceneState properties
without instantiation. Report:
`artifacts/citadel-runtime-integration/animated-scene-state-diagnostic-20261006/report-r2.json`;
owned watchdog `watchdog-r2.json` has functional 0, clean cleanup and authoritative
zero. All five imported assets serialize the default library as `libraries/`
with an `AnimationLibrary` resource, while the descriptor reader recognized only
a `libraries` dictionary. The first diagnostic had a temporary script indentation
error; the corrected diagnostic proves this schema mismatch.

The next cohesive producer descriptor change supports actual default/named
serialized library properties, rejects malformed or ambiguous entries, and
preserves bounded asset/node/property diagnostics through source capture. Its
content digest excludes instance IDs and registry counters; runtime owner proof
retains those identities separately so equal-content owner replacement changes
freshness without changing generated content revisions. Reuse the leased frozen
catalog in source pass state instead of deep-copying it. Acceptance requires the
actual five imported descriptors, default/named and malformed contract cases,
equal-content resource replacement with stale-owner rejection, relevant compact
capture gates, then the same bounded real Main gate. No missing animation data
will be treated as empty or accepted by a fallback.

Descriptor verification passed:
`node tools/run-animated-asset-scene-state-descriptor-contract.mjs --OutputDirectory artifacts/citadel-runtime-integration/animated-asset-scene-state-descriptor-r3-20261006`
has 13 passing checks, including all five actual imported clips and stable semantic
digest plus stale old-descriptor rejection after equivalent Resource replacement.
The runner binds source and GLB hashes and verifies source stability. Functional 0,
clean cleanup and authoritative zero. Earlier r1/r2 fixture/indentation failures
are retained; r3 is the passing evidence. The production capture integration also
passes the full project compile smoke (watchdog `godot-l94o6p`) and all 81 adapter
assertions in `ecology-section-value-adapter-v2-descriptor-20261006/report.json`.
These remain descriptor/compile/service checks. The next real Main run used
`main-section-cohabitation-gate-catalog-v2-descriptor-20261006` with the same seed
and bounds. It failed `main_startup_ready` at 120.485 seconds, functional exit 1,
clean cleanup and authoritative zero. The animation failure is gone: no terminal
source capture failures were recorded. However no section candidate or native
compile was completed. At exit, 203 source jobs and 1,337 section demands remained;
the run performed 394 fresh catalog captures. Maximum ecology admission was
1045.104 ms (latest 216.095), source service 347.390 ms (latest 236.245), and catalog
capture 987.874 ms (latest 105.493). The pending set grew from 89 source jobs and
770 demands around 41.5 seconds. Before another full run, trace whether support
owner requests recursively expand rendering demand and redesign catalog ownership
to avoid repeatedly proving unchanged immutable data. Fixed-neighborhood input
capture and the render demand set are separate concepts in Minecraft; preserve
that separation while retaining all genuinely intersecting tree/building support.

Read-only closure audit **did not find recursive expansion**: the bridge derives
owner sections only from original view snapshots, then queries those owners in a
separate map. It never derives another owner set from that map. Native viewer
publication and one-hop owners both enter the coordinator's aggregate counter,
so growth alone cannot identify either source. Current startup view distances are
80 cells initially and 96 at target. Keep this finite closure; add distinct view,
terrain-backed, support-only and source-chunk counters before attributing growth.
The confirmed repeated work is the 394 outer catalog scopes: each reconstructs
profile values, asset/rock/tree envelopes and animated descriptors, then copies
and hashes them before deduplication. The next architecture unit is owner-published
immutable catalogs with explicit mutation/reload boundaries, composed and leased
across frames. A generation-counter shortcut over still-mutable public Resources
would not satisfy the contract. Owner/consumer/write inventory and exact API
agreement are entry requirements before that production edit.

The independently usable descriptor subsystem was committed as game commit
`ff39dd5f` (`Add verified SceneState descriptors for animated assets`): registry,
dedicated fixture and its Godot-generated UID, and Node runner only. Final rerun
`artifacts/citadel-runtime-integration/animated-asset-scene-state-descriptor-final-20261006/report.json`
passes 13 checks after adding the UID to the source manifest. The owned editor UID
import completed with clean cleanup and zero members; its only diagnostic was
the existing nested scratch-project warning. The broader rendering integration
remains dirty and unaccepted. No game push was made.

The failure currently arises during the deterministic wildlife-intent pass,
which shares seeded iteration with static ecology. Mobs remain separate rendered
actors, but their descriptor admission can currently fail the static source pass.
Full migration still needs actor readiness separated from static publication while
preserving all seeded RNG draws; fixing the descriptor format alone does not
prove that independence.

Independent review also identified a non-startup capacity risk: 128 idle ready
source jobs can retain leases across the catalog store's 8 artifact generations.
Ordinary asset lookup does not change registry generations; increments occur
during setup or explicit test/reload actions. Add pressure-aware idle retirement
before declaring repeated registry replacement fully accepted.

Independent read-only consumer/lifecycle review found no additional production
reader expecting full catalogs in v2 inputs. Capture cancellation releases through
the original Main resolver; section demand retirement releases support leases;
world reset drains compilers before revoking old-epoch artifacts. The SupportIndex
synthetic fixture now uses the same admitted-input contract with all existing
revision/receipt assertions preserved and added lease-lifecycle coverage.

Real Main cohabitation, ordinary menu/startup, headed visual/traversal, edits and
harvest, unload/replay, save/reload and representative runtime performance remain
open. The earlier 900-second Main startup failure is the live comparison baseline.
The native byte limit is a logical input/output buffer quota; allocator overhead
and temporary Godot result arrays are additional. The latest-generation metadata
map still grows with visited section slots. Native identities are validated and
echoed; full census and payload validation belongs to coordinator/finalization,
so a native receipt alone is not authoritative world readiness.
