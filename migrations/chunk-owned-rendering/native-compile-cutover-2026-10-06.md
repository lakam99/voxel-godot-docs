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

Before changing capture ownership, measure catalog-input building, source queue
callbacks, and census admission through the existing bounded performance monitor
and retain those timings in the Main report. A short diagnostic replay of the
same seed is sufficient; do not repeat the unchanged five-minute gate.

The next cohesive change, if timings confirm the repeated catalog work, is a
Main-thread-owned immutable producer catalog context with an explicit provenance
epoch. Reusable profile/model/grammar/asset envelopes belong to that context;
terrain revisions, structure admission, chunk identity and removal projections
remain per-source inputs. Audit the actual mutation/reload boundaries first.
An arbitrary frame cache or revision-only hit must not hide changed resources.
Preserve source/census stale checks and deterministic producer RNG. Acceptance:
exact before/after capture parity, invalidation on each catalog mutation boundary,
bounded context retirement, then real Main compilation/installation followed by
the still-open visual/traversal and performance gates.

Real Main cohabitation, ordinary menu/startup, headed visual/traversal, edits and
harvest, unload/replay, save/reload and representative runtime performance remain
open. The earlier 900-second Main startup failure is the live comparison baseline.
The native byte limit is a logical input/output buffer quota; allocator overhead
and temporary Godot result arrays are additional. The latest-generation metadata
map still grows with visited section slots. Native identities are validated and
echoed; full census and payload validation belongs to coordinator/finalization,
so a native receipt alone is not authoritative world readiness.
