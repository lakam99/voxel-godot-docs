# Section-Owned World Rendering: Architecture Charter and Stage Plan

**Status:** active migration; a live Main-scene production section candidate now reaches a current native render receipt. Full visual cutover and gameplay acceptance remain open.
**Recorded:** 2026-10-04
**Canonical source:** `voxel-godot` branch `codex/chunk-owned-world-rendering-migration`

## Outcome

Publish terrain and static non-mob world visuals as complete, revision-bound
render-section candidates owned by streamed chunks. A section replacement
includes all of that section's current content across the game's render
policies. Keep the previous installed section visible until every required
part of its replacement is installed and acknowledged. Empty is an explicit
replacement; missing or uncompiled content is never empty success.

The section renderer is a derived presentation system. `WorldGenerationSystem`,
`TerrainVolumeService`, prepared structure recipes, deterministic ecology,
durable edits, and stable gameplay object IDs remain authoritative. Collision,
navigation, interaction, harvesting, save/load, doors, and actor simulation
retain their existing owners and readiness contracts. Mobs and NPCs remain
independent live actors; this migration does not bake them into static section
geometry.

## Current source paths and cutover implications

| Domain | Current producer and renderer | Revisions, ownership, gameplay and persistence | Migration constraint |
|---|---|---|---|
| Terrain | `WorldGenerationSystem` composes `TerrainVolumeService`; `VoxelTerrainRuntime` creates Voxel Tools `VoxelTerrain`, `VoxelTerrainGenerator` fills SDF/material channels, and Voxel Tools' Transvoxel mesher publishes smooth visuals and collision. Exact fluid meshes use `TerrainMeshingService` and `MainRuntimeTools.apply_chunk_fluid_mesh`. | Durable cell deltas and a global volume revision are sectioned for save. Runtime invalidates edited section/chunk receipts and reapplies changes when native blocks are editable. Collision and terrain rendering share the Voxel Tools authority; navigation consumes separate terrain projections. | No application-level complete visual-section candidate or layer-install receipt currently wraps Voxel Tools remeshing. Verify its mesh replacement and stale-job behavior before selecting an interception/cutover point. Preserve smooth SDF meshing, material identity, seams and edit behavior; Minecraft's block mesher is not a replacement. Fluid/light publication is a distinct current path and must be represented honestly in the manifest. |
| Ordinary generated structures | `StructureSystem` creates deterministic per-cell records and revisions in `ordinary_visual_sources`; `MainChunkTerrain.create_block` materializes each cell as a `StaticBody3D` with a visual child plus collision/interaction metadata. `OrdinaryStructureVisualSourceCapture` and `GeneratedStructureVisualManifest` observe readiness through weak references to live nodes. | Stable source IDs are town-qualified or standalone-cell-qualified. Per-source and global `ordinary_visual_revision` change on completion, deletion and restoration. `removedGeneratedStructureBlocks` persists tombstones; recipes rebuild from seed. Each block body currently jointly hosts visual and gameplay data, so only its render child may transfer. | The observer is not copied mesh data and cannot census an unloaded source. Capture expected cells, omissions, stable revisions, mesh/material/layer and bounds before node creation/unload. Replace only the visual child after every affected section candidate is acknowledged; keep collision, interaction and save identity. |
| Blueprint buildings and landmark structures | `CitadelSiteBuildQueue` → `BuildingPublicationPreparation` → `BuildingScenePublicationJob` → `BuildingPartPublisher`. Current static visuals group by material, tier, logical owner cell and source part, then `BuildingStaticBatchFlush` commits individual `ChunkRenderPacketBackend` packets. | Source-part revision derives from its structural snapshot. 32-cell logical owner, 28-cell stream owner and render-section identity differ. Collision batches, door state/interactions, furnishings and navigation manifests remain structure-owned. Generated structures are reconstructed; durable player block edits/removals are saved separately. Per-source packet replay recipes are retained across chunk recreation. | Current packet commits are atomic only per source/material/tier group; a later group can fail after earlier visuals are visible. Replace this production grouping with complete section candidates and per-section contributor receipts. Do not merge collision, doors, furnishing interaction or nav authority into render buffers. |
| Trees and foliage | Deterministic chunk RNG and stable prop IDs feed `TreeRuntimeRequestBuilder`, `TreeEcologySampler`, `TreeSpawnService` and `TreePublicationQueue`. Near/mid visuals install per tree; horizon ecology has separate batches/impostors. `TreeChunkBatchRenderer` is a prototype, not production authority. | Tree body/trunk collision, harvest/drop metadata, prop ID and navigation blocking are gameplay-owned. Branch/leaf collision is not required. Removed prop IDs are saved; recipes and geometry regenerate from seed/authority. Preserve legacy RNG draw order, including compatibility draws. Horizon sources are retired/promoted separately from physical chunks. | Capture admitted canonical recipe/LOD outputs without re-running ecology or reading visuals back from nodes. Include branch/canopy bounds and cross-section coverage. Replace old tree visuals only after the complete affected section set is accepted; keep body, harvest identity and collider alive. |
| Props and decorative detail | `MainPlaytestTools` produces deterministic props and detail transforms per chunk. Rocks/ore/forage become per-prop bodies with visual children; decorative detail becomes typed `MultiMesh` batches under `DecorBatches`. `ChunkPropVisualManifest` observes candidate/represented state for readiness. | Prop source revisions include seed, chunk identity and completed candidate content. Harvested IDs are stored in `removedProps`; visual geometry regenerates. Physical props keep their collision and interaction bodies. Horizon-only roots are visual-only. | Capture sealed producer values before scene-node materialization, preserving transforms, colors, custom data, wind/material policy and stable IDs. A manifest/readiness observer is not the renderer. |
| Wildlife / NPCs | Wildlife and NPCs use animated, collision-aware actor bodies and independent behavior/movement schedules; distant visual-only wildlife may be separately represented. | Actor state, simulation distance, collision, interaction and save data are actor-owned. | Keep live mobs/NPCs outside static section buffers. Do not mistake their visual-only far proxies for simulation or physical readiness. |

The game branch now configures terrain, blueprint-building, ordinary-structure,
and ecology/static-prop providers on the world-owned coordinator. A seeded
headed Main-scene diagnostic passed at game revision under active development
with a complete 22-contributor census, 29 captured inputs, and a native receipt
for one selected section. Provider coverage may legitimately be explicitly
empty for domains with no members in that section, so this result does not yet
prove a populated blueprint or ordinary-structure visual migrated into the
candidate. The section candidate installs alongside existing category visuals;
old producer retirement is not wired. Stage 3 terrain is not complete because
the candidate bridge still rejects translucent/fluid sort publication and does
not yet subsume all terrain light/collision/readiness contracts. See the exact
production command and artifact report in the 2026-10-04 findings below.

The per-source packet baseline passes 24/24 checks at
`artifacts/citadel-runtime-integration/native-chunk-packet-baseline-pre-section-cutover-20261004/report.json`;
the prior static replay fixture passes 5/5 at
`artifacts/citadel-runtime-integration/static-flush-baseline-pre-section-cutover-20261004/report.json`.
The latter uses a fake backend. Snapshot/grid/partition/ledger contracts are
data-level evidence. The current game tree contains unrelated generated
`.import` churn that must remain unstaged. The documentation repository also
has pre-existing edits to the visible-world readiness plan and implementation;
those files are outside this charter and must be preserved.

## Shared section candidate contract

Use a stable 3D render-section key, distinct from source-part ID, logical
32-cell owner, and 28-cell streamed-chunk owner. Select the final section size
from terrain mesher alignment, culling, upload/memory costs, dependency shape,
and headed measurements; Minecraft's 16-block dimensions are a reference, not
a mandated size.

Each immutable candidate carries:

- world/session epoch, section key, monotonically increasing slot generation,
  source/boundary revision, candidate digest and explicit candidate state;
- a complete contributor manifest: stable source and part IDs, content
  revision, kind, canonical gameplay owner, per-section ranges, actual mesh and
  material/pipeline identities, transforms, exact bounds, render policy, and
  expected instance/byte counts;
- all render-layer outputs required by this game (opaque, cutout/foliage,
  translucent/fluid and any measured additional policy), plus sorting or
  material state needed by each layer; an empty layer/section is explicit;
- terrain volume/material/fluid/light revisions and the sample halo revisions
  required by the smooth mesher; static contributors are admitted from their
  existing deterministic, prepared artifacts;
- canonical section-render owner identity and every intersecting source
  coverage dependency, each bound to its authoritative source revision and
  immutable captured values. Source gameplay chunks may unload after capture;
  they are not long-lived render-owner leases. If capture is still reading live
  chunk data, that source remains admitted until the immutable snapshot is
  sealed. Cross-section geometry has one canonical visual owner and explicit
  coverage ranges/dependencies.

Preparation consumes owned value snapshots, never live Nodes, mutable producer
containers, or scene scans. Work is bound to world, section, source, and owner
revisions. Cancellation or section-slot reassignment invalidates older work.
Immediately before install and before promotion, revalidate the exact source
set and revisions, world epoch, candidate generation, and live section-owner
epoch/backend identity. A source revision change invalidates the candidate even
after source-chunk unload. A stale or missing source is pending or failed, never
empty success.

Build every required layer in staging while retaining the old installed
section. Atomically replace one section slot only when all present layer
resources and that section's exact contributor manifest have install
acknowledgements. Publish an explicit empty replacement when the current
manifest has no content. On failure/cancellation, retire only staged data and
keep that section's last valid content. A source crossing section boundaries
is partitioned into section-local contributions and receives one receipt per
affected section; source-level readiness is promoted only after every affected
section receipt is live. Individual section slots may promote independently,
so during an update adjacent sections may briefly show different generations.
Do not retain a global all-sections commit lock to hide that bounded skew.
Where old per-source visuals still exist during migration, retire each old
section-local representation only when its replacement slot is accepted. A
Godot node installation receipt is not a GPU fence and must be reported as
such.

The section system changes presentation ownership only. Terrain volume remains
the authority for solidity, material, digging, fluids and saved deltas. Existing
gameplay bodies remain the authorities for collision, interactions, harvest,
doors, nav blocking and durable IDs. Rendering may reference those IDs but may
not invent parallel gameplay state. Saves continue to store durable edits and
removed/generated-object deltas, not cached generated meshes; deterministic
generation reconstructs the candidate after load.

## Minecraft 26.2 design evidence

Use the local reference at
`C:\Users\arkam\Documents\Minecraft Java Source Reference\26.2\decompiled\net\minecraft\client\renderer\chunk`.
`SectionCompiler.compile` gathers one section's block and fluid geometry into
layer-keyed output, while also collecting visibility, block-entity references
and translucent sort state. `RenderSectionRegion` gives the compiler a captured
3×3×3 section neighborhood for reads. `SectionRenderDispatcher` cancels work
when a slot is reassigned, distinguishes uncompiled from empty, retains the
old mesh until every present layer's vertex/index upload is acknowledged, and
commits an empty result explicitly. These lifecycle rules inform our candidate
contract. The block-state/cube mesher, fixed section size, layer names, actor
rules and upload implementation do not transfer directly to our smooth SDF
terrain, generated recipes or Godot renderer.

## Staged migration and gates

| Stage | Work and gate | Exit evidence |
|---|---|---|
| 0. Source map and baseline — **complete** | Trace all producer paths, authorities, revisions, ownership, unload/replay and saves; record current packet behavior and known missing terrain receipt. | This charter, focused current packet/replay reports and independent audits. Baselines prove only their named contracts. |
| 1. Close the candidate contract | Bind candidates to exact prepared ledger outputs, world/session and owner/dependency generations. Define layer completeness, empty, cancellation, retries and contributor-to-section receipts. Verify available section mesh/upload APIs, especially Voxel Tools' real remesh retention and terrain payload capture. | Contracts reject forged/truncated/stale candidates, wrong owner/dependencies, duplicate/missing layers, partial installs and old generations; API/source audit resolves Voxel Tools interception. No production path cut over yet. |
| 2. Implement section-slot installation | Replace the source-keyed production packet primitive where necessary with a section-keyed staged owner independent of gameplay-chunk lifetime. Support compatible multi-batch/multi-layer payloads, exact manifests, source-revision-bound capture dependencies, cancellation, owner recreation, explicit empty, and old-slot retention until complete per-section install. Cross-section sources get independently promoted section-local contributions, with source readiness waiting for all affected section receipts. | Native/engine contract exercises real section owner/resource installation and receipt checks; cross-chunk captured candidates install without retaining source chunks; stage aborts leave the affected old section installed; native build and shutdown/retirement pass. |
| 3. Integrate smooth terrain | Feed mesh artifacts from the existing authoritative SDF/material source into the section candidate using the game's mesher and correct halo/seam contract. Keep collision, edit, fluid/light and nav authorities separate but revision-linked. Do not disable Voxel Tools visuals until parity and section-local replacement behavior are proven. | Real-scene section receipt for terrain; edit/remesh/empty/stale/cancel/unload/reload fixtures; seams, collision, lighting and fluid captures show source parity. Then retire the old terrain visual slot with no duplicate visual authority. |
| 4. Cut over construction | Migrate both ordinary per-cell generated structures and blueprint/landmark building batches. Aggregate every affected source's opaque/material groups into complete section candidates. Remove per-cell/per-source publication as the final visual authority. Reconcile stable source revisions, moved owners, all affected sections, empty removals, replay and scene-boundary readiness; preserve collision/interaction/doors/furnishings/nav. | Real generated structure visual playtest and captures; replacement/removal/cancel/replay and chunk recreation tests for both paths; boundary readiness cannot succeed on a partial section. |
| 5. Admit ecology and static props | Feed accepted tree recipes/LODs, decorative detail buffers and prop visual recipes through the same section manifest. Preserve color/custom/wind attributes and material layers. Replace per-tree/per-prop visual nodes only after all affected section receipts are accepted; retain their bodies/interactions. Keep wildlife/NPC actors independent. | Deterministic source parity and removal/save/reload tests; live forest/prop visual captures and traversal show complete coverage without pop or hitch. |
| 6. Readiness, performance and legacy retirement | Wire section receipts into visible-world readiness and chunk unload/replay. Remove obsolete production per-source visual publication only after all consumers have migrated. Run normal startup, movement/turn, edit, harvest, save/reload and unload/recreate journeys. | Headed live visual/traversal evidence across representative seeds; visual readiness has no candidate/receipt gaps; performance report includes startup, p95/max cadence, queue/backlog and streaming spikes; no legacy production renderer remains for migrated categories. |

### Startup readiness constraint — event driven, no production wall-clock timeout

The game's initial spawn remains behind the existing startup overlay until the
authoritative 360-degree visible terrain/render readiness contract reports
loaded for the selected spawn position. Progress reports completed/required
work while streaming and section compilation advance. A slow or retryable
producer keeps readiness pending; production boot must not convert elapsed wall
time into a readiness failure. Explicit cancellation/shutdown and an
authoritative non-retryable subsystem failure remain terminal. Keep finite
watchdogs in automated runners as diagnostics; they do not release or fail the
production player spawn.

The current `MainCore.gd` startup path still has hard deadlines
(`INITIAL_READINESS_TIMEOUT_SECONDS` and
`FINAL_TERRAIN_EXPANSION_TIMEOUT_SECONDS`), and some lower-level startup stages
return `*_timeout` on pending work. A headed diagnostic on seed
`source-additions-diagnostic-20261004` remained at `Scanning 360° view · 5/32
prop sources` until its runtime readiness call returned without `ready`; the
diagnostic then threw while converting the structured failure dictionary to
`String`, masking the cause. The reporting cast has been corrected to JSON, and
the same-seed replay is collecting the actual structured blocker. Before
calling startup complete, map each deadline to its readiness dependency and
replace elapsed-time failure with an event-driven retry/acknowledgement path.
Do not weaken completeness to make boot finish. Gate acceptance on a headed
normal-runtime load that remains responsive, displays advancing progress, and
spawns only after current terrain plus required visible static-section receipts
are acknowledged. The selected location, 360-degree coverage, and screenshot
must be included in the report.

## Next implementation charter — production source census admission

**Outcome:** begin wiring the normal world lifetime to the section coordinator
through one authoritative, revision-bound producer census. No section may be
promoted from a terrain-only, building-only, or other partial manifest. The
existing VoxelTerrain and per-source static visuals remain visible until every
contributor in each affected section has a prepared replacement and current
renderer receipts.

**Authority and boundary:** keep `WorldGenerationSystem` and
`TerrainVolumeService` authoritative for terrain; `StructureSystem`, prepared
building plans, deterministic ecology/prop producers, and durable removal
records authoritative for their source members. The census must be derived from
those producers, not from a scan of visible nodes or readiness manifests. Each
entry identifies stable producer/source identity, source and dependency
revisions, exact intersected render sections, bounds, render layers, and an
explicit complete/empty/pending/failed state. Mobs and NPCs remain independent
actors. Collision, interactions, navigation, harvest, and save authorities do
not move into render sections.

**Scope for this slice:** add a composed production roster/census under
`scripts/world/`, owned by `MainCore` beside the world-lifetime section
coordinator. Providers are registered by authority owner and method, referenced
weakly, and queried for exact requested sections; provider output carries
revisioned source IDs and explicit complete/empty/pending coverage. A provider
that cannot census independently of a render owner remains pending. The census
unions source revisions and sorted contributors only after every required
provider reports complete coverage, and its digest is rechecked during staged
installation. No old visual is retired in this slice. Keep the terrain shadow
queue diagnostic until it can submit through the complete coordinator census.
Preserve deterministic generation/RNG order, source tombstones, owner
recreation, and retryable requests.

**Baseline and known risks:** game branch
`codex/chunk-owned-world-rendering-migration` at `2b9e0f69`, with pre-existing
generated `.import` churn only. The coordinator currently has no production
caller. Terrain capture can seal current 19³ SDF/material bytes and 27 section
revisions but still requires a resident block to capture; ordinary structures
have stable tombstones but their live-node observer is not an immutable census;
Citadel plans provide exact prepared member sets but current receipts require a
live job; ecology ledger omits rocks, ore, forage and underground props. These
gaps risk falsely treating absence as empty and must be represented as pending.

**Stages and exit evidence:** (1) add the roster/census API and adversarial
contract coverage for missing, empty, stale and replaced producers; (2) attach
it to `MainCore`'s normal world lifetime and connect at least one production
authority census, while proving incomplete sections are refused; (3) route the
first fully enumerable real section through coordinator build/install/ack while
legacy visuals remain visible, proving native renderer receipts and stale-work
rejection; (4) only after all contributing domains are complete, run headed
visual/traversal and performance gates before retiring any old publication.
Report each gate independently; this charter is a Stage 1–2 implementation
slice, not a completed migration or a production visual cutover.

## Next implementation charter — prepared blueprint membership provider

**Outcome:** register the first normal-world source provider from immutable
`CitadelPublicationPlan` values. Its coverage is a site-local membership
census for exact 3D render sections. It must not infer global emptiness from a
single site or from absent renderer jobs. Terrain, ordinary structures and
ecology/props remain required roster domains and keep section admission
pending until their own authorities are registered.

**Authority and boundaries:** use admitted site bindings, prepared publication
bases and their immutable plan output signature/member records. Add a true 3D
AABB-to-section intersection query while preserving the existing XZ query
contracts. Stable contributor IDs are section-local projections of a site
member; revisions bind site/source identity, plan signature, member ID/group,
bounds and visual eligibility. Do not include job/Node instance IDs or scene
readiness receipts. The census changes no building visual, collision, door,
furnishing, navigation, save or publication path.

**Acceptance evidence:** test negative coordinates, vertical section boundaries,
members spanning section planes, explicit site-local empty coverage, unresolved
admission as pending, and plan/source replacement invalidation. In a real
MainCore-owned coordinator, verify the provider registers and its census is
complete only for blueprint membership; normal section admission must remain
pending because the other required domains are absent. No visual/traversal or
performance acceptance claim follows from this provider-only stage.

## Progress snapshot — 2026-10-04

- Stage 0 is complete. Producer and gameplay-authority maps plus current
  per-source packet/replay baselines are summarized above.
- Stage 1 has partial implementation in game commits `eb973da0` and
  `f35f5413`. `PreparedStaticContributorLedger` now builds and retains the
  exact replacement snapshots itself, binds a world ID, and rejects stale
  section generations. Its current contract report is
  `artifacts/citadel-runtime-integration/prepared-static-contributor-ledger-slot-generation-20261004/report.json`.
  Receipt dictionaries can still be fabricated by an untrusted caller, so the
  ledger contract is not an install authority.
- Stage 2 has an initial native bridge in commits `1f5a37b7` and `d546311b`.
  `NativeStaticSectionInstallSession` installs one immutable opaque candidate
  under a section-keyed slot using the native chunk packet backend. The 29-check
  runner
  `artifacts/citadel-runtime-integration/native-chunk-packet-owner-epoch-gate-20261004/report.json`
  builds the candidate through the ledger/partitioner/snapshot builder, installs
  its resources through the GDExtension, verifies the backend receipt, and
  promotes it. It also checks old-root retention on cancel, stale owner/generation
  rejection, and fail-closed cross-chunk dependencies.
- This is not a production producer cutover. `BuildingStaticBatchFlush` still
  installs source/material/tier packets, terrain remains on Voxel Tools, and
  tree/foliage/details/props retain their existing publication paths. The
  bridge supports opaque content only and has no normal-world producer callsite.
  Cross-chunk keys are capture-coverage metadata; exact per-source capture
  revisions/epochs and full producer census are not yet bound to a production
  candidate. No headed generated-world/traversal or performance report
  demonstrates live section-owned rendering yet.
- The project checkout contains the Voxel Tools extension descriptor and
  compiled extension but not its implementation source. The extension's exact
  mesh replacement, stale-work cancellation and payload interception hooks
  remain unavailable to the application-level audit; terrain integration cannot
  treat the existing renderer as a section-slot receipt authority.

**Installed Voxel Tools API reflection (2026-10-04):**
`node tools/run-voxel-tools-api-reflection.mjs` passed with no validation errors.
The exact debug DLL loaded by the project is identified in
`artifacts/native-world-backend/n1-api-reflection/2026-10-04T043737-257Z-92423ab4/report.probe.json`
by SHA-256 `b24cc4eb8d22c27ce5babf1cf23190571d4acca9b2d215cf9c9adf00614bfc97`.
ClassDB exposes `VoxelMesher.build_mesh(VoxelBuffer, materials,
additional_data)`, `VoxelTerrain.mesh_block_entered/exited`,
`VoxelTerrain.is_area_meshed`, `has_data_block`, and coordinate conversion.
It does not expose a ClassDB method to retrieve the installed visual mesh or
acknowledge an application-owned replacement. `mesh_block_entered` only
reports that a mesh block entered the terrain system; `is_area_meshed` reports
processing, not non-empty visibility or replacement ownership. Thus these
signals cannot stand in for a section receipt. The supported investigation path
is an immutable padded `VoxelBuffer` snapshot from the authoritative terrain
source, passed to the existing Transvoxel mesher to form the candidate; its
capture must include durable edits, neighboring sample revisions, collision
parity, and fluid/material state before the Voxel Tools visual is retired. The
reflection establishes only the public ClassDB surface of the installed
binary, not that no private/native hook exists. The DLL has no product version
metadata or pinned upstream commit in the repository, so upstream master docs
are not accepted as exact-binary evidence.

**Native renderer seam recheck (2026-10-04):**
`node tools/run-native-chunk-render-packet-contract.mjs -OutputDirectory artifacts/citadel-runtime-integration/native-chunk-packet-section-coordinator-preintegration-20261004`
passed all 29 checks. It reconfirms actual chunk-owned backend creation,
section candidate installation through the native renderer, retention of the
previous section root on cancellation, owner/generation rejection, and current
building packet replay after chunk recreation. The production building flush
still installs per source/material/tier; this report verifies the install seam,
not contributor aggregation, ordinary generated-world behavior, or visual and
traversal parity. The first production cutover therefore needs a world-owned
coordinator above `CitadelPublicationService`'s concurrent scene jobs; a
per-job ledger cannot prove section completeness when jobs overlap.

**World coordinator contract recheck (2026-10-04):** a draft
`WorldStaticSectionCoordinator` now serializes source-part replacement
boundaries above a ledger, requires a freshly supplied exact contributor census
on every install advance, and checks the actual native section-slot receipt
before ledger promotion. The native runner passed 31/31 checks at
`artifacts/citadel-runtime-integration/native-chunk-packet-world-coordinator-contract-final-20261004/report.json`,
including rejection of an incomplete census while the prior native generation
remained installed. The first run found that candidate contributor records live
at `replacement.snapshot.manifest`; the coordinator had assumed a nonexistent
`snapshot.contributors` field. This schema mismatch is fixed and covered by the
passing rerun. The coordinator is still unconnected to world producers, so this
does not advance the production-cutover gate or complete Stage 1. The renderer
still supports only opaque same-owner candidates, and terrain, building, tree,
flora and prop candidates are not yet composed into one production manifest.

Do not advance a stage because a pure contract or source scan passes. For each
stage report passed, failed, blocked and untested gates separately, preserve
prior installed content on failed replacement, and keep the overall migration
active until all required domains pass live acceptance.

**Independent section-owner cutover check (2026-10-04):** the main runtime now
creates bounded static section owners beside gameplay chunks, evicts them by
render demand, and resolves native section installation/receipt checks through
that owner registry. `NativeStaticSectionInstallSession` treats the existing
stream-chunk list as immutable source-capture coverage, not a gameplay-owner
pin. The focused runner passed 33/33 checks at
`artifacts/citadel-runtime-integration/native-chunk-packet-section-owner-contract-verified-20261004/report.json`.
This proves native resource installation and old-root retention for the section
slot, independent owner creation/retirement, cross-chunk manifest acceptance,
and census rejection. It remains fixture-driven: no normal terrain/building/
tree/prop producer calls the section coordinator, layer coverage is opaque-only,
and headed/live/performance gates remain open. Source capture manifests still
need exact source revisions and capture epochs for each covered chunk.

**Next production slice — resident smooth-terrain shadow candidate (2026-10-04):**
Before any visual switch, feed one real resident terrain section from the
authoritative Voxel Tools volume into the section candidate and install it in
the native section renderer while the Voxel Tools visual remains authoritative.
Capture the exact resident VoxelBuffer, including edited material/SDF channels,
with the configured Transvoxel mesher and its required halo. Bind the candidate
to world identity, section key, capture epoch, every intersecting source-block
revision, mesher/material schema, layer, bounds, and a digest of the resulting
mesh payload; reject it if any revision changes before installation receipt.
An explicitly empty result is valid only for a complete current capture.

The current packet backend represents batches with MultiMesh nodes, and the
application session currently admits only opaque instance-transform payloads.
Do not assume that an arbitrary ArrayMesh can safely be wrapped as a single
MultiMesh instance: first verify it in the real GDExtension renderer, preserve
Transvoxel surface attributes/materials, and account for mesh arrays as well as
instance buffers in staged and resident byte limits. Add the necessary
mesh/layer support at the shared candidate boundary rather than a terrain-only
visual authority. Keep the original Voxel Tools surface visible until candidate
content, material/lighting, seams, edit behavior, collision, and replacement
acknowledgement are proven in a controlled headed fixture; collision and digging
remain owned by Voxel Tools/TerrainVolumeService throughout this shadow step.

The first candidate is deliberately one real resident terrain section, not a
claim of complete world publication. Its manifest must state that terrain alone
is covered. It must not be promoted as a complete section while building,
ecology/tree, foliage, and prop contributors are absent. Before whole-section
promotion, connect authoritative producer registries above concurrent site
jobs and runtime ecology publishers, enumerate exact current contributors and
tombstones for all intersecting sections, and create a complete section-local
replacement for every section touched by cross-boundary sources. Promote
sections independently after each slot is complete; promote aggregate source
readiness only after all affected section receipts are current. Preserve
gameplay chunk unload/replay and save/delta authorities. Minecraft 26.2 validates the
section lifecycle pattern (3x3x3 region inputs, per-layer compilation,
cancellation of stale tasks, and retention/release of the prior section mesh),
but its block mesher and draw-buffer implementation are not suitable substitutes
for this game's smooth Transvoxel terrain or Godot native backend.

**Renderer resource-shape probe (2026-10-04):** the owned native contract
`node tools/run-native-chunk-render-packet-contract.mjs -OutputDirectory artifacts/citadel-runtime-integration/native-chunk-packet-arraymesh-section-20261004`
passed 34/34 checks. A real triangle `ArrayMesh` was installed as the mesh of a
`MultiMeshInstance3D` under the native section owner, and the test inspected the
installed node/resource identity. This verifies that the current Godot/native
wrapper can host arbitrary triangle surfaces; it is not a Transvoxel output,
mesh-byte-budget, live VoxelTerrain capture, headed image, or production terrain
publication proof. The next terrain stage still needs a bounded native mesh
payload budget and a capture matching the live VoxelTerrain authority.

**Section mesh identity and native payload accounting (2026-10-04):** the
candidate compatibility key now includes a deterministic digest over actual
mesh surface bounds, primitive types and packed arrays. The install session
rejects a resource whose content does not match the sealed manifest. The native
backend snapshots admitted meshes, validates them again before upload and
receipt, and tracks estimated CPU mesh-array plus instance-buffer payload across
staged, installed and retiring roots; these estimates do not include renderer
or GPU allocations. Retirement accounting is idempotent. The native runner
passed 36/36 checks at
`artifacts/citadel-runtime-integration/native-chunk-packet-manifest-mesh-digest-final-rerun-20261004/report.json`;
focused snapshot, builder and ledger contracts also pass. These tests still use
synthetic content and do not satisfy the production-producer, terrain-capture,
headed visual/traversal, or performance gates. Stages 1–2 remain partial.

**Candidate-to-native mesh binding recheck (2026-10-04):** the section install
session checked the resource digest when it began, but the native backend copied
the mesh only later during append. A mutable resource could therefore change
between those two points and be installed under a manifest describing its old
geometry. `append_batch` now requires the candidate's expected mesh-content
digest and compares it with the deep-copied mesh payload before staging. The
building packet path captures that digest before begin and binds it into its
packet digest as well. `StaticRenderMeshFingerprint` now handles both
`ArrayMesh` and `PrimitiveMesh` resources with one versioned payload contract.
The focused native section/building lifecycle runner passed all 37 checks at
`artifacts/citadel-runtime-integration/native-chunk-packet-append-mesh-binding-clean-20261004/report.json`,
including a real GDExtension check that mutates a `BoxMesh` after begin and
verifies append rejects it. This strengthens Stage 2 only: the coordinator still
has no normal producer caller, production buildings still commit source-keyed
packets, terrain/foliage/props do not share section candidates, and no headed
visual/traversal or performance gate has passed. The broader native-world
validation runner was attempted but stopped on failures in cave-field,
natural-terrain golden, underground-prop and world-source tests. Their baseline
classification is unknown; they remain unresolved and are preserved in
`artifacts/native-world-backend/section-mesh-identity-20261004-rerun/report.json`.

**Producer-boundary audit and next-stage plan (2026-10-04):** the current
branch is `codex/chunk-owned-world-rendering-migration` at
`787f43405a16b3594a840af61c2cf774c86059bf`. The game worktree has broad
Godot `.import` churn outside committed code; it is being preserved. The docs
worktree is on `main` at `8e48f40cfda9f9c71a019f16e62cccceb07fcfb4`; its two
dirty visible-world-readiness files are unrelated and remain untouched.

Three independent read-only audits completed before the next production edit:

- **Terrain producer:** production terrain is `VoxelTerrainRuntime.terrain`,
  with 16-cell Transvoxel mesh blocks and SDF16/INDICES8/DATA5_8 channels.
  A candidate can only be captured from resident authoritative VoxelData, not
  regenerated from the generator or from the native shadow-volume test source.
  The proposed padded input is 19³ samples, bound to terrain instance, seed /
  generator identity, global and all intersecting 3D section revisions,
  resident mesh-block event, and copied-byte digest. Voxel Tools remains the
  collision and visible authority during this first shadow installation; the
  installed extension exposes no application-level installed-mesh receipt.
  Terrain saves remain durable edited-cell deltas, not mesh cache.
- **Buildings:** `BuildingPartPublisher` already has prepared immutable visual
  segment buffers and exact source-part revisions, while `BuildingStaticBatchFlush`
  still publishes source/material/tier packets. Collision, doors, furnishings
  and navigation remain separately owned by the building/gameplay system.
  The coordinator has no production producer caller, its census must be supplied
  externally, it rejects replacements spanning more than one section, and the
  current candidate accepts opaque instances only. A building-only connection
  would omit co-located terrain and ecology and is not a valid cutover.
- **Ecology / props:** the deterministic spawn state machine has stable object
  IDs and durable harvest tombstones, but completed child manifests observe
  materialized scenes and cannot enumerate unloaded chunks. Detail IDs/revisions
  currently include runtime batch identity. Trees keep gameplay bodies,
  harvest state and trunk collision while visual recipes publish asynchronously.
  A stable producer-side, value-only generated-source ledger is needed to give
  loaded and unloaded sections the same exact census without changing RNG draw
  order.

### Immediate production slice and stage exits

The next code slice is a **resident live-terrain shadow candidate**, not a
whole-section promotion or a per-domain renderer cutover. Before capture, the
runtime must prove the requested mesh block is current, visible/resident and
editable/meshed. Capture the live 19³ halo and its exact SDF/material channels
into owned values; bind world/source/block identity, every affected 3D section
revision, edit-pending state, mesher/material configuration, and payload digest.
Revalidate after capture and immediately before installation. A stale,
nonresident, pending-edit, or unavailable source returns an explicit retryable
pending/failure reason, never empty success. Run those values through the
configured smooth Transvoxel mesher and install the resulting mesh/material
candidate through the real section owner. Keep Voxel Tools rendering and
collision authoritative until a controlled headed comparison proves the
candidate's appearance, material/light behavior, seams, and edit replacement.

The capture contract must be exercised on a real VoxelTerrain scene and include
resident-copy digest parity, edit invalidation, unload/re-entry invalidation,
and a real native-section receipt for the produced Transvoxel resource. The
first candidate manifest explicitly says terrain-only and is never promoted as
a complete section. This closes a terrain source boundary and proves a
production-shaped renderer installation, but does not pass the whole-section
producer gate.

Before any section is promoted as complete, the architecture must additionally
have: (1) a world-lifetime exact source census, including explicit empty and
tombstone revisions for terrain, generated structures, trees, foliage, detail
and props, independent of scene-child lifetime; (2) immutable per-section
multi-layer candidate payloads for smooth terrain, instanced recipes and the
game's translucent/fluid policies; (3) independent atomic section-slot
promotion after each affected slot has an exact live receipt, with aggregate
source readiness waiting for all affected sections; (4) stale source/owner
cancellation and replay after source/render-owner unload; and (5) production
producer routing that retains each old representation until
the complete replacement is acknowledged. Mobs/NPCs, collision, interactions,
navigation and saves stay with their established authorities.

Stage exits remain the seven rows in the table above: Stage 0 source map;
Stage 1 complete immutable manifest/census contract; Stage 2 section slot,
multi-layer/per-section install lifecycle; Stage 3 real terrain source and
visual cutover; Stage 4 buildings; Stage 5 ecology and props; Stage 6 gameplay
readiness, traversal/performance and legacy publication retirement. Stage 3
shadow installation is not terrain cutover. Each exit needs its named contract
evidence and all later gates remain open. The Minecraft source check was
repeated against `SectionCompiler.java`, `RenderSectionRegion.java`, and
`SectionRenderDispatcher.java`: compile visits section interior while reading a
3×3×3 copied neighborhood; dispatcher cancellation discards stale results,
stages each rendered layer, retains old content until upload acknowledgements,
and explicitly installs empty output. We adopt these lifecycle principles, not
its block-state mesher or fixed dimensions.

**Resident terrain capture and real section-renderer shadow install (2026-10-04):**
`VoxelTerrainRuntime.capture_resident_terrain_mesh_block` now copies live resident
VoxelData instead of regenerating the candidate from the generator. It requires
the visible resident mesh block and complete 19³ Transvoxel input area, rejects
pending edits, captures SDF16/INDICES8/DATA5_8 into owned bytes, records all 27
intersecting 3D section revisions plus mesh-block/world/generator identity, and
digests the channel payload. `resident_terrain_capture_is_current` rechecks the
source and section revisions and recomputes the byte digest before install. The
feature remains a shadow path; it does not disable or replace Voxel Tools
rendering, collision, edits, or saves.

The focused headed command
`node tools/run-playtest.mjs --only resident_terrain_section_capture --seed section-capture-audit-20261004 --visible true`
passed. Its report is
`artifacts/chunk-owned-rendering/terrain-candidate-install-headed-rerun-20261004/report.json`.
The gate observed a production resident block `(12, 0, -1)`, validated the
19³ byte counts and 27 source-section revisions, rejected a tampered payload,
built a one-surface Transvoxel mesh, and installed a terrain-only candidate
under Main's real independent static-section owner. The native receipt bound
generation 1, the manifest digest, owner `(6, -1)`, and source-capture
dependencies `(6, -1)` and `(7, -1)`. Voxel Tools visual/collision stayed
active. The screenshot still shows the loading overlay, so this proves source
capture and native renderer installation, not player-visible mesh/material
parity, edit replacement, traversal, or performance. The first install attempt
was rejected because the fixture used an unsupported native render tier; the
runner was corrected to use the backend's accepted structural importance tier
and the focused headed rerun passed.

The same runner was initially made to wait for full startup before invoking the
terrain probe. That run stopped at its 240-second watchdog with Main loading at
2075/2083 visuals, eight pending (all reported in structure publication) and
no runtime failure payload. It did not reach the capture assertion. This is a
separate unresolved startup gate with no baseline comparison; do not attribute
it to this renderer work or call the 360° startup readiness gate passed. The
capture-only gate was moved to observe Main's actual terrain while startup was
still active.

Stage 3 has passed its live resident-source and native-install shadow subgate.
Stage 1 remains partial (no all-domain, unload-independent census); Stage 2
remains partial (native layered slot API is contract-tested but production
installer/callers remain opaque-only); Stage 3 has a runtime-owned,
terrain-only shadow queue and live native receipt, but visible parity,
edit/collision/fluid/light parity and Voxel Tools visual retirement remain
untested. Stages 4–6 remain open. Keep the migration active. The verified game
slice is committed at
`2b9e0f69e308738fa832a8942e58c1fbbdafa5db` on
`codex/chunk-owned-world-rendering-migration`; generated `.import` churn remains
unstaged in the game worktree.

**Section lifecycle and ecology producer increments (2026-10-04):** after
reviewing Minecraft 26.2's per-section mesh swap in
`SectionRenderDispatcher`, the contract no longer requires one atomic commit
across multiple section slots. Each section independently stages and promotes
only after all its layers and exact manifest are acknowledged; a contributor
crossing sections is considered ready only after its full set of section
receipts is current. This allows bounded adjacent-section generation skew,
matching the reference lifecycle and avoiding an unsafe sequential cross-owner
transaction. See documentation commit `994ab51` for the decision.

Game commit `694ab21a` adds separate native `begin_packet_with_layers` and
`append_batch_in_layer` APIs while keeping the legacy packet signatures
unchanged. The focused owned-process runner
`node tools/run-native-chunk-render-packet-contract.mjs -OutputDirectory artifacts/citadel-runtime-integration/native-chunk-packet-multilayer-section-slot-clean-20261004`
passed 41/41 checks. The report proves per-layer batch/instance counts, explicit
empty layer receipts, cancellation/incomplete replacement preserving the old
root, and an installed empty replacement through the real GDExtension node.
Godot material still selects the actual pipeline; this is not a GPU fence or
sorting proof, and production install callers remain opaque-only.

Game commit `6870245c` adds a chunk-scoped value ledger at the deterministic
tree/detail producer boundary. The focused command
`node tools/run-detail-ordered-parity.mjs` passed for seed `atlas-1492` with all
53 attempts, unchanged final RNG state `-4425083070199577339`, matching tree
and detail recreation digests, one durable-removal tombstone, and native detail
parity. The ledger captures tree recipe inputs and detail instance attributes
before scene publication, contains no duplicated collision/interaction
authority, and unloads with its streamed chunk for deterministic reconstruction.
It omits rocks, ore, forage, wildlife and underground props; tree meshes are
not compiled into section payloads; no producer submits these values to section
slots. This closes only a bounded source-value subgate, not Stage 1 census or
Stage 5 integration. Game HEAD at this checkpoint is
`694ab21a80520ac8abc93eb404cd15bf17455c84` on
`codex/chunk-owned-world-rendering-migration`; unrelated generated `.import`
changes remain untouched.

**Layered snapshot-to-slot integration (2026-10-04):** game commit `173dacfa`
extends the immutable section snapshot with exact `opaque`, `cutout`, and
`translucent` batch/instance counts, including explicit empty rows. The
contributor ledger and snapshot validator now admit opaque and alpha-scissor
content and retain valid translucent sort-policy identity. The install session
passes the exact layer manifest through the native API and appends batches to
their declared layers. A real alpha-scissor material/candidate was installed
through the section owner; the other two layers received empty receipts. A
translucent candidate is explicitly rejected as
`native_section_translucent_sort_not_implemented` until its sorting behavior is
implemented and verified.

Focused evidence: the owned native contract
`node tools/run-native-chunk-render-packet-contract.mjs -OutputDirectory artifacts/citadel-runtime-integration/native-chunk-packet-section-layer-sort-gate-final-20261004`
passed all 42 checks. The ledger contract passed all checks at
`artifacts/citadel-runtime-integration/prepared-static-contributor-ledger-section-layers-commit-20261004/report.json`;
the snapshot-builder layer-manifest check passed at
`artifacts/citadel-runtime-integration/prepared-static-section-snapshot-builder-layers-final-20261004/report.json`;
and the pure assembled-snapshot contract passed at
`artifacts/citadel-runtime-integration/chunk-static-render-section-snapshot-layers-20261004/report.json`.
These establish immutable per-layer candidate counts and real native slot
installation/retention. They do not establish transparent sorting, production
producer cutover, complete cross-domain census, visuals in normal gameplay,
traversal, or performance. Stage 2 remains partial; Stages 1 and 3–6 remain
open.

**Producer census recheck (2026-10-04):** current worktree HEAD is
`7f3ddbed` on `codex/chunk-owned-world-rendering-migration`. The structure audit
identified two distinct production domains: ordinary generated structures
publish per-cell bodies/visual children through `StructureSystem` and
`MainChunkTerrain.create_block`; blueprint/landmark buildings publish through
`BuildingPartPublisher` and source/material/tier packets. Both are now explicit
in the source map and Stage 4 gate. The ecology audit confirms the seed-order
and stable-ID boundary in `MainPlaytestTools`, a partial tree/detail value
ledger, and missing value captures for rock, ore and forage. Wildlife actors
stay independent. The terrain audit's resident volume→Transvoxel candidate is
now submitted and installed by a runtime-owned, retryable terrain-only shadow
queue rather than test-owned meshing. Full source census/coordinator admission
and visible terrain replacement remain unimplemented.

The present seven-stage status is: Stage 0 source map complete; Stage 1
candidate/census contract partial; Stage 2 native section slots and cross-owner
receipt aggregation partial (translucent sort and stale-mid-boundary recovery
still open); Stage 3 terrain shadow capture/install and runtime-owned producer
subgate passed, full-census admission and visual/edit/collision/fluid/light
parity open; Stage 4 both structure
paths not cut over; Stage 5 only tree/detail source values captured (no complete
section admission or natural-prop values); Stage 6 gameplay readiness,
traversal/performance and old publisher retirement untested.

The runtime-owned terrain shadow producer is in place and its renderer-install
subgate passed (entry below). The next production boundary is to admit its
candidate through the world-lifetime section coordinator with an authoritative
full contributor census, while preserving VoxelTerrain until all contributors
in each affected section are represented. The padded Transvoxel origin has a
single-block bounds check, and source residency-at-copy is now separate from
the sealed candidate's world/local-authority revision check. Real unload/reload
and replay evidence is still required. Terrain visual replacement still needs
headed parity, edits, collision, fluid, light, unload/replay and performance
evidence.

**Multi-section coordinator proof (2026-10-04):** game worktree commit
`b3b53785` removes the coordinator's one-impacted-section rejection, and
`cfffcb76` strengthens its integration fixture across distinct render owners.
A changed source can now compile, install, and promote the exact full receipt
set for all sections it touches. The native contract moved one contributor from
section `(0,0,0)` in owner `(0,0)` into `(2,0,0)` in owner `(1,0)`, installed both
complete section slots through separate native backends, and accepted the
ledger only after both live receipts matched.
The same run rejected an incomplete contributor census without replacing the
current slot. Evidence:
`artifacts/citadel-runtime-integration/native-chunk-packet-multichunk-coordinator-verified-20261004/report.json`
(43/43 checks).

This remains a coordinator/native integration fixture, not a normal gameplay
producer cutover: no ordinary terrain, building, tree, or prop producer yet
submits its full production source set to this coordinator. Per-slot promotion
is atomic; cross-section source readiness waits for the full affected receipt
set. If an active boundary becomes stale after one slot has promoted, that
section keeps its successfully installed complete generation and must be
replaced by the retry boundary; this temporary section-generation skew and
retry path still need a dedicated stale-mid-boundary test. Stages 1–6 remain
open.

**Runtime-owned terrain shadow producer (2026-10-04):** game commit
`2b9e0f69e308738fa832a8942e58c1fbbdafa5db`.
`TerrainSectionShadowPublisher` now owns a bounded, retryable capture → candidate
build → renderer-install queue, advanced once per `VoxelTerrainRuntime` frame.
The test runner only submits and polls a request; it no longer meshes or stages
the candidate itself. Source copy and stale-revision checks remain on the
runtime thread, renderer installation is budgeted over frames, and the original
VoxelTerrain visuals/collision remain authoritative. Candidate results expose
mesh-build timing. This still publishes terrain-only under a shadow world ID,
does not have the full shared source census, and does not switch visible terrain.

Command:
`node tools/run-playtest.mjs --only resident_terrain_section_capture --seed section-shadow-runtime-queue-20261004-r3 --visible true`.
It passed capture integrity plus native section installation through the
runtime-owned queue and produced `artifacts/test-runners/playtest.png` and
`playtest-report.json`. A named contract check retired the residency registry
entry after sealing: the resident validator rejected the snapshot while the
world/local-authority validator accepted it; this is synthetic registry
retirement, not a live native unload/reload. The focused fixture observed an all-air vertical
neighbor as an explicit `empty` result, then installed the surface-bearing
block. The installed mesh bounds were inside the expected 16-cell local block
(`P=(0,11.21539,0)`, `S=(16,4.784615,16)`), so the capture padding did not shift
this sample's mesh origin. This is one-block coordinate evidence, not seam
parity. The screenshot is still behind the startup loading overlay, so visible
parity is untested. Traversal, save/replay, collision/edit, fluid/light and
runtime-performance gates remain open. The capture still requires a resident
source block at capture; real native source unload/reload and replay remain
unverified. This is a Stage 3 producer integration increment, not migration
completion; Stages 1–6 remain open.

**Loaded-world headed gate (2026-10-04):**
`node tools/run-playtest.mjs --only terrain_section_shadow_live_install --seed section-shadow-live-install-20261004 --visible true`
did not reach the terrain candidate phase because ordinary startup readiness
failed. At the 240-second readiness limit it remained at 2074/2082 visuals, with
eight pending visuals classified as generated structures; 29/31 structure
sources were complete, all 31 prop sources were complete, and terrain, trees,
foliage and wildlife counts had no pending representations. Report:
`playtest-report.json`; live progress: `playtest-progress.txt`; owned-process
watchdog:
`artifacts/node-tools/process-runs/godot-QXCTkr/watchdog.json` (exit 1, cleanup
passed, authoritative zero job members). No loaded-world terrain screenshot or
candidate visual result was obtained. This repeats the earlier late-startup
failure signature, so it is classified as a pre-existing/unresolved startup
readiness issue for this candidate change; the shadow queue was not invoked in
this run. The fast headed subgate above remains the only passing producer proof.

**Sealed terrain source after residency retirement (2026-10-04):** game commit
`2b9e0f69e308738fa832a8942e58c1fbbdafa5db` separates resident-at-copy
validation from sealed-candidate authority validation. The latter checks
payload digest, world/generator/terrain identity, mesher/material identity, and
all 27 intersecting terrain-volume section revisions without requiring the
native mesh block to remain resident. The headed contract gate
`node tools/run-playtest.mjs --only resident_terrain_section_capture --seed section-shadow-runtime-queue-20261004-r3 --visible true`
passed on the same seed, including synthetic retirement of the runtime
residency registry and rejection of tampered payloads. It still ran before
startup readiness; registry retirement is not an actual native unload/reload or
renderer replay test. The runtime also received an installed native section
receipt with mesh bounds inside the expected 16-cell local block. Real unload,
re-entry, replay, visual parity, traversal and performance remain open.

**World-lifetime source roster admission (2026-10-04, r14):** game source now has
`StaticSectionSourceRoster`, owned by `WorldStaticSectionCoordinator` and
instantiated by `MainCore` for the seeded world. Four required producer domains
are declared: terrain, ordinary structures, blueprint buildings, and
ecology/static props. The roster accepts providers by weakly held authority
owner/method, requires explicit complete or empty coverage for every requested
section, merges stable source revisions, rejects duplicate owners and missing
answers, and binds a deterministic census digest to staged coordinator work.
The coordinator rechecks that digest on each advance and cancels a staged
boundary when authority revision or membership changes; the producer receives
`requiresResubmit` because its prepared declaration/segment may now be stale.

Review found a cross-section replacement hazard: the coordinator can install
section slots independently, so cancelling after one slot commits would leave
that replaced slot without rollback. Roster admission now refuses requests
containing more than one section until atomic or rollback-safe promotion is
implemented. If a provider becomes pending while a single-section boundary is
active, that stage is cancelled with `requiresResubmit`, preserving the last
installed slot. The roster also rejects non-string revision keys and values
and requires revision IDs to exactly match contributor membership in the
requested sections.

Game code commit: `6acc709b` (`Add revisioned static section source roster`).

The native contract command
`node tools/run-native-chunk-render-packet-contract.mjs --outputdirectory artifacts/citadel-runtime-integration/native-chunk-packet-source-roster-20261004-r14`
passed all checks. It installed a single-section candidate through the native
section renderer using fixture providers, refused missing/omitted source
coverage and multi-section admission, changed a source revision during staged
installation, cancelled work when a provider became pending, and confirmed the
prior installed generation remained current. Exact report:
`artifacts/citadel-runtime-integration/native-chunk-packet-source-roster-20261004-r14/report.json`.
This is coordinator/native integration evidence, not normal-world producer or
gameplay evidence: at this r14 checkpoint, no authority provider had been
registered in `MainCore`. No production section can yet pass the complete roster, and no headed
visual/traversal/performance gate ran. Stages 1–6 remain open. The next gate is
one real producer authority provider plus refusal of incomplete cross-domain
sections. Cross-section-safe promotion must be designed before multi-section
sources can pass roster admission. Later gates still require a complete
all-domain section and live parity before any legacy visual retirement.

**Blueprint source census provider (2026-10-04):** game work adds a provider
owned by `CitadelPublicationService` and registers it from `MainCore` for the
`blueprint_buildings` domain. It uses admission's deterministic region
decisions and the immutable `CitadelPublicationPlan` member manifest; it does
not inspect scene Nodes, job instance IDs, visual omissions, collision,
interactions or gameplay readiness. Membership is clipped in exact 3D to the
16-cell section grid after conservative XZ admission, with stable
site/member/section IDs and revisions bound to source binding, plan signature,
member/group bounds and section key. Unknown admission or plan state stays
pending. The three other required domains remain unregistered and therefore
prevent production roster admission.

Game code commit: `2d29d9f8` (`Register blueprint section census provider`).

The native chunk packet contract passed at
`artifacts/citadel-runtime-integration/native-chunk-packet-section-provider-20261004-r5/report.json`.
It includes negative-coordinate, vertical-separation, flat-plane ownership,
half-open section-edge checks, and a synthetic admission-backed explicit-empty
provider response alongside the existing native renderer/slot lifecycle
checks. This proves the immutable membership query, provider response shape,
and native install fixture separately; it does not feed blueprint geometry
into that renderer. The building-preparation contract could not run
its existing actual-source fixture because the referenced files
`artifacts/citadel-runtime-integration/actual-site-source-05/result.bin` and
`artifacts/citadel-runtime-integration/publication-preflight-02/blueprint-mutation.bin`
are absent from this checkout. No headed world, traversal, save/replay or
performance gate ran. This remains a census-provider increment only; all
migration stages remain open.

Minecraft 26.2 source review confirms the useful lifecycle comparison: its
`SectionCompiler` compiles a center section against an immutable 3×3×3
`RenderSectionRegion`, emits independently tracked render layers, and its
`SectionRenderDispatcher` switches the section mesh only after every present
layer's vertex and index buffers have uploaded. The Godot migration should keep
that whole-section ownership and swap rule while retaining this game's smooth
terrain mesher and native collision/edit authority; vanilla block meshing is
not a fit. The live renderer proof above remains only the generic native slot
contract, not a normal-game producer cutover.

## Next implementation charter — terrain section candidate handoff

### User-visible outcome and non-goals

Move one resident smooth-terrain mesh block from the private
`TerrainSectionShadowPublisher` install path into the shared world section
coordinator's staged install. The captured candidate must use the same 16-cell
interior and section key as the current Transvoxel block. Keep the current
`VoxelTerrain` mesh and collision installed until a current shared-slot receipt
exists. This is an integration step, not permission to retire the existing
terrain renderer.

Do not change terrain generation, SDF/material authority, Transvoxel output,
collision, digging, lighting, fluids, save deltas, structure/tree/prop
producers, or mob/NPC simulation. Sections whose fluid contribution is not
proven empty or prepared remain ineligible for a complete shared candidate.
Missing source domains remain pending; this step must not fabricate empty
coverage for them.

### Authority and handoff

`VoxelTerrainRuntime` owns the terrain value snapshot and its source epoch:
world/generator/mesher/material identity, the durable revisions for all 27
intersecting volume sections, and the captured padded SDF/material payload
digest. Mesh-block residency and instance IDs fence an in-flight capture only;
they are not stable save/replay revisions. `TerrainSectionShadowPublisher`
continues to own bounded capture/mesh/install work scheduling, but returns a
value-only prepared source declaration and immutable instance segment to the
world coordinator instead of creating a private ledger or installing
independently. The coordinator recaptures the complete required-domain roster
before and during section installation, rejects stale revisions, and retains
the previous slot if any check or layer upload fails.

`VoxelTerrain` remains the terrain collision/edit/save authority. Existing
fluid, ordinary-structure, blueprint, tree/foliage and prop sources keep their
owners and must either contribute revisioned payloads or remain pending before
the shared replacement. The 16-cell source and section alignment means this
first handoff affects exactly one section; spanning-source promotion is a
separate chartered step and must keep per-section promotion atomic.

### Acceptance evidence and exit criteria

1. A focused live-runtime contract requests a resident, dry section through the
   production runtime owner, validates capture revisions before/after meshing,
   stages the immutable section payload, and installs it through the native
   section renderer using the existing owner/layer manifest.
2. Edit a source volume section after capture and during staged upload; stale
   work must cancel, the old slot must remain installed, and collision/edit
   authority must remain unchanged.
3. Prove a missing required provider and unresolved fluid dependency block the
   candidate instead of allowing a terrain-only replacement. Label fixture
   providers as such; this contract does not count as normal-world cross-domain
   readiness.
4. Compare the installed terrain mesh at the real world transform against the
   still-visible Transvoxel mesh for the tested section. A contract-only green
   result is not visual parity.

Passing this gate adds one producer handoff only. It does not prove fluid or
LOD parity, save/load, actual native chunk unload/replay, traversal,
performance, or permit retiring the old terrain visual. Run headed visuals and
representative performance when an actual production section can pass the
complete authority roster. Until terrain, ordinary structures, blueprint
buildings, and ecology/static props all have truthful providers and payloads,
normal gameplay remains on its current rendering path.

## Producer-audit refinement — 2026-10-04

Follow-up read-only audits checked the terrain, ordinary-structure, and
ecology/prop handoff against the local Minecraft 26.2 `SectionCompiler`,
`RenderSectionRegion`, and `SectionRenderDispatcher`. Minecraft's relevant
lesson remains a whole-section candidate compiled from an immutable 3×3×3
neighborhood, with independently accounted layers and a slot swap after all
present buffers upload. Its block-state mesher is not applicable to our smooth
Transvoxel terrain or procedural tree recipes.

The audits refine the production admission gates:

| Domain | Reusable authority or data | Still blocks complete section admission |
|---|---|---|
| Terrain | `VoxelTerrainRuntime` can seal one resident block's immutable 19³ SDF/material payload and 27 volume-section revisions. Validation can continue after native block unload. | Census must use stable authority revisions, not resident mesh-block instance/revision epochs. The current fluid renderer is separate and chunk-shaped; absent fluid nodes/meshes are not proof of empty section fluid. Until an exact section/halo fluid probe or payload is revision-current, the terrain provider stays pending. Never copy/hash the 19³ payload during each roster recapture. |
| Ordinary generated structures | `StructureSystem` has stable source revisions, completed/omitted records, per-cell identities, and durable generated-block tombstones. | `OrdinaryStructureVisualSourceCapture` is a readiness observer, not geometry. Expected cell/type records omit some resolved visual options. A candidate needs copied mesh/material/layer/bounds/transforms from an allowlisted static visual class, bound to source/cell/tombstone revisions. Unknown, failed, or ungenerated records stay pending. Keep the per-cell gameplay bodies, collision, doors, interactions, light, navigation and saves. |
| Trees, flora and static props | `EcologySourceValueLedger` captures tree recipe inputs and surface detail transforms/custom attributes from the existing seeded producer. Tree recipes and removal IDs already have deterministic/gameplay owners. | That ledger omits rocks, ore, forage and underground props; its current chunk-instance manifest revision is not replay-stable, and several producer/admission/material revisions are not bound. Do not run a second RNG producer to fill those holes. Wildlife stays an independent actor. A tree/detail subset may be tested in a clearly separate partial slot, but cannot satisfy or replace the complete ecology/props section manifest. |
| Blueprint buildings | The immutable Citadel publication plan supplies exact 3D membership and stable source/plan revisions. | The registered provider is census-only; its current prepared packet covers only selected masonry/paving/roof groups and is not yet a complete section geometry payload. Missing blueprint members and other domains remain pending. |

The ordinary-structure and ecology audits are design evidence only; neither
changes production code nor passes a renderer/gameplay gate. Their practical
consequence is that the shared coordinator must not receive an explicit-empty
claim from a readiness manifest, absent node, unbuilt chunk, or partial ecology
ledger. The next code increment must declare its exact producer subset and
remain visibly non-authoritative until a complete all-domain section can pass
the roster, install through the native section slot, and retain the old
production visuals on stale or failed work.

## Next implementation charter — producer geometry handoff wave

### Outcome and candidate contract

Replace the private terrain-shadow installation with producer submissions to
`WorldStaticSectionCoordinator`, then grow the same section candidate to cover
the current terrain/fluid, ordinary-structure, blueprint-building, and
ecology/static-prop sources. The candidate remains one immutable 16-cell
section replacement with a sorted, revisioned contributor manifest and
explicit opaque, cutout, and translucent/fluid layer accounting. Native
section-slot ownership and GPU-facing batches are section-keyed, not
source-keyed. Mobs, wildlife actors and NPCs remain independent.

This is a progressive cutover. Existing VoxelTerrain and static producer
visuals remain visible while any producer is being migrated. A partial packet
may be prepared and tested in an explicitly non-authoritative renderer slot,
but it cannot satisfy complete roster admission, replace a production section,
or be called gameplay acceptance. A source may say `empty` only when its own
deterministic authority proves exact section coverage at a current revision.

### Producer handoff rules

- Terrain values come from `VoxelTerrainRuntime`'s sealed 19³ SDF/material
  capture and intersecting volume revisions. Mesh-block residency is a capture
  fence, not a replay revision. Fluid is independently scanned or prepared from
  `TerrainVolumeService`; no mesh/node absence proves empty. A fluid-bearing
  section stays pending until its exact native fluid geometry is included in a
  supported section layer. Godot's transparent-object ordering is not
  per-face sorting inside a section mesh; keep water/lava pending until their
  camera-dependent quad order can be updated under the installed section
  generation, source revision and sort token. A camera change must not rebuild
  terrain or switch away from the last accepted translucent order while its
  replacement is incomplete.
- Ordinary generated structure geometry is copied from an admitted immutable
  recipe/mesh source and bound to stable source, cell and tombstone revisions.
  Initially allow only stateless masonry/path visuals. Keep per-cell bodies,
  collision, doors, interactions, lights, navigation and saves under their
  existing owners.
- Blueprint geometry comes from prepared Citadel packet values and exact
  section partitioning. Every member/group must be represented; selected
  masonry/paving/roof groups alone cannot claim blueprint completeness.
- Ecology values must come from the existing admitted seeded producer. Bind
  canonical tree recipe, terrain/admission, environment/material schema and
  removed-ID revisions. Account for branches/opaque and foliage/cutout,
  details, rocks, ore, forage and underground static props. Do not rerun RNG to
  fill a missing record. Wildlife remains an actor.

#### Instance attribute parity

The current shared section instance ABI is
`static-instance-transform-color-custom/v2`: 12 transform floats, 4 independent
instance-color floats, and 4 custom-data floats. Native `MultiMesh` publication
enables and writes color and custom lanes separately. The ABI contract passes
through section installation and replacement; default white preserves older
packets while custom data remains in its original lane. Producer adapters must
preserve the exact v2 schema in their candidate identity and never overload
custom data as tint. This does not yet prove production ecology install or
visual parity.

All async work carries world, section-slot, source-part and dependency
revisions. Recapture the complete producer census before/during installation
and immediately before promotion. Any changed or unavailable producer cancels
staged work while retaining the old slot. Capture and compile once per admitted
candidate; roster recapture must be metadata-only, bounded, and must not recopy
the 19³ terrain payload every frame. Cross-section meshes use exact transformed
bounds and per-section fragments/receipts.

### Ordered implementation and acceptance

1. **Shared terrain handoff.** Replace the shadow publisher's private ledger
   and direct install with immutable declaration/segment submission to the
   world coordinator. Add a metadata-only stable terrain census and
   revision-current exact fluid coverage. An unresolved or fluid-bearing
   section remains pending until its fluid layer is prepared. Prove stale
   capture cancellation and old-slot retention with a live resident block.
2. **Structure payload adapters.** Adapt one ordinary static masonry/path
   source and complete blueprint packet membership into section-local native
   batches. Preserve source bodies and all gameplay records. Missing/failed
   source completion remains pending; tombstones and explicit empty removals
   are covered.
3. **Ecology/static prop adapters.** Extend the same existing seeded capture
   rather than introducing a second generator. Include all static producer
   families or keep the section pending. Prove canonical recipe/revision parity,
   harvested-ID exclusion, and cross-section ownership.
4. **Complete renderer gate.** Install a normal-world section that includes
   every producer's current contributors and all required layers through the
   native section slot. Compare the installed result at its real transform
   against legacy visuals, inject a source edit during upload, and verify stale
   rejection with the old slot and gameplay collision intact. A controlled
   fixture provider may prove the coordinator API/native upload seam only; it
   cannot satisfy this normal-world gate.
5. **Live cutover.** Run headed forest/structure traversal, edit/harvest,
   unload/recreate and save/reload checks plus a representative runtime
   performance observation. Retire legacy static visuals only category by
   category after the relevant complete manifests and receipts pass.

The next production edit must implement item 1 without treating it as migration
completion. Items 2–5 stay open until their named evidence exists. Continue
rechecking Minecraft 26.2's immutable neighborhood, per-layer upload receipts,
cancellation and retain-old-until-ready behavior; keep Godot's smooth terrain,
tree grammar, gameplay/save authorities and renderer semantics native to this
game.

## Architecture restart charter — whole-section production candidates

**Status:** required next production design; implementation remains in progress.

### Outcome and non-goals

Replace source-by-source production publication as the static world render
authority with one immutable candidate for a complete 16-cell render section.
Each candidate is assembled from every current terrain, ordinary-structure,
blueprint-building, tree/foliage and static-prop contributor in that section,
then partitioned and batched once across categories. It installs through the
existing section-slot/native renderer seam. A partial source subset may be
captured for diagnosis but cannot replace production visuals or pass section
readiness. Keep the old section visible until every required layer has an
accepted receipt. Keep mobs, wildlife and NPC simulation/rendering independent.

This replaces the per-source ledger/publication flow as a production candidate
builder where it cannot express one complete section. Existing ledger and
producer packet contracts remain useful regression evidence during migration;
they do not constrain the new production architecture. Do not change seeded
generation, terrain/SDF or biome authority, structure/tree/prop identity,
collision, interactions, doors, harvesting, navigation or save format. Saves
continue to store durable edits and removals; generated render buffers are
reconstructed from authoritative producers after load.

### Candidate and data path

```text
authoritative producer snapshots + exact section census
  -> provider-specific value adapters
  -> merged immutable section inputs and resource bindings
  -> one cross-domain section partition/batch pass
  -> complete layer/manifest snapshot + digest
  -> source/owner recapture and revision validation
  -> staged native section install and per-section receipt
  -> provider acknowledgement and retirement of matching old visuals
```

The complete candidate records world/session epoch, stable 3D section key,
slot generation, sorted exact contributor IDs/revisions, provider authority and
coverage revisions, canonical gameplay owners, section-local ranges, actual
mesh/material/pipeline digests, transform/color/custom attributes, bounds,
layer/policy, stream dependencies, counts/bytes and explicit empty output.
Opaque and cutout are supported only when the real material semantics match;
translucent/fluid stays pending until ordering and revision updates are
implemented. Smooth terrain is meshed by the existing Transvoxel path from
owned authoritative volume bytes and its required halo revisions; Minecraft's
block-state mesher is not reused.

All inputs are sealed values, not Nodes, RIDs, Callables or mutable producer
containers. Every candidate source ID must match exactly one entry in the
authoritative section census. Census includes current live contributor
revisions and separate source-identified removal tombstones; removed members
are not current contributors. Tombstones are consumed only after the matching
section receipt is accepted. Bind resource contents and section owner/backend
epochs to the candidate. Recheck provider epochs, exact census digest, source
revisions, world identity, resources and owner before upload and immediately
before promotion. Any changed or unavailable input cancels staged work while
leaving the old installed section untouched. One source instance has one
center-owned section under `StaticRenderSectionGrid`; the captured world bounds
and intersecting stream dependencies are retained in its manifest.

### Stage plan and exit gates

1. **Freeze provider contracts.** Terrain, ordinary structures, blueprint
   buildings, ecology/tree/detail and props expose immutable section-local
   inputs, current contributor IDs/revisions, removal tombstones, layers and
   resource bindings. Unsupported members remain pending. Prove each exact
   identity and source-to-section mapping with focused contracts.
2. **Build complete section candidates.** Add one assembler that unions all
   provider inputs, rejects missing/duplicate/unowned or stale members, merges
   compatible cross-domain batches, partitions once, and builds the final
   immutable snapshot. Prove exact roster-to-manifest equality, explicit
   empty, stale recapture, resource binding, size/backpressure and cross-section
   center ownership.
3. **Install the production candidate.** Drive visible section demand through
   the assembler and existing `NativeStaticSectionInstallSession`, retaining
   old roots through cancellation and acknowledging providers only after a
   matching native receipt. Exercise a real world-produced complete candidate
   through the actual renderer; synthetic snapshot or fixture-empty evidence
   does not pass this gate.
4. **Move domain production.** Cut over smooth terrain, then both structure
   paths, then trees/foliage/detail and static props, each only after the
   previous layer retains its source authority and passes live appearance,
   edit/removal, unload/replay and save/reload checks.
5. **Retire old visual authorities.** Run headed forest/structure traversal,
   turning, editing, harvesting, unload/recreate and representative runtime
   performance checks. Remove old per-source visual publication only when each
   affected section has a current complete manifest and receipt with no
   readiness gaps. Do not call any preceding stage full migration completion.

### Current entry state

The branch at implementation entry is `codex/chunk-owned-world-rendering-migration`
at game revision `2d29d9f849bb5f26622bdc340101853f2e24d586`, with uncommitted
section ABI/provider work and pre-existing generated `.import` churn. The
canonical docs repository is on `main` at `683be945fe17e29c904f05af1db2d08fab024c08`;
its unrelated visible-world readiness edits must be preserved. A headed
terrain-only candidate previously installed through the native section owner,
but its fixture explicitly treated static providers as empty. It proves only
the renderer seam, not a complete production section. Focused producer evidence
is being added for ordinary structures, Citadel packet groups and partial
ecology detail; tree geometry, full natural-prop census, translucent/fluid
layers, production demand orchestration, normal-world candidate installation,
live traversal and performance remain open.

### Implementation evidence update — 2026-10-04

The charter is committed at documentation revision
`be2005b81f6baa2c1f54da2402a27287241c1dec`. Game implementation remains on
`codex/chunk-owned-world-rendering-migration`, uncommitted, with unrelated
`.import` churn excluded from task staging.

| Gate | Result | Scope and remaining proof |
|---|---|---|
| Provider contracts | Partial | Ordinary structure, Citadel packet, surface detail, and one-tree captures have focused contracts. Ecology's section census deliberately remains pending while tree, rock/ore, forage, and underground props are not all represented. |
| Whole-section assembler | Passed, synthetic | `whole-section-candidate-assembler-final-20261004/report.json`: 6 checks. Two provider inputs share one compatibility batch; exact manifest, explicit empty, retryable missing provider, stale provider epoch, and compatibility conflict are checked. Does not prove live provider parity. |
| Native install seam | Passed, headed controlled fixture | `whole-section-candidate-native-install-main-runtime-compile-20261004/report.json`: 8 checks through the real GDExtension renderer, including compilation of the normal runtime loop. It observes the receipt, retained previous slot before commit, replacement after commit, stale-census rejection with prior generation still installed, and replay after owner recreation from the last committed candidate. Providers are controlled fixtures, so this does not pass the normal-world producer gate. |
| Production wiring and world cutover | Open | `WorldStaticSectionCoordinator` has a bounded normal-runtime advancement hook and a provider-census → contribution → shared-assembly → native-queue entry point. The headed 8-check install fixture exercises this full coordinator path with synthetic providers. There is still no normal visible-section-demand caller because terrain/ecology contributions and post-receipt retirement are incomplete. Legacy visual publication remains. Terrain fluid, full ecology/static-prop census, producer receipts/retirement and whole-world unload/replay are not integrated end to end. |
| Playtest and performance | Untested / unresolved | `node tools/run-playtest.mjs -OutputDirectory artifacts/citadel-runtime-integration/whole-section-runtime-playtest-20261004` was stopped by the runner after headless dummy-renderer `Initializing already initialized RID` and null-mesh errors. Progress remained at `runtime_loading_Drawing_nearby_terrain`; no `playtest-report.json` was emitted. The owned-process watchdog proved zero remaining members. This is not gameplay acceptance, and current evidence does not classify the renderer errors as introduced by the migration. |

Minecraft 26.2 source checks continue to support immutable section snapshots,
worker preparation separated from staged upload, independent layer receipts,
cancellation, and keeping the previous installed section until replacement
acknowledgment. Godot retains its smooth Transvoxel terrain path. The controlled
native fixture validates provider-census and contribution dispatch, shared
assembly, and installation of the candidate envelope into the real GDExtension
slot, but both fixture providers are synthetic. Ordinary structure and Citadel
adapters separately have focused value-contribution evidence. The production
stage remains open until terrain and ecology providers produce complete current
values, normal visible-section demand drives the coordinator, old visuals retire
after accepted section receipts, and gameplay visuals/traversal are connected
and measured.

### Implementation evidence update — coordinator composition (2026-10-04)

Game commits `4a0da843` and `3d41673d` add a world-lifetime complete-candidate
composition API. It recaptures the required provider census, asks each provider
for a sealed contribution, invokes the one cross-domain assembler, and submits
only the resulting whole candidate. Missing contribution APIs remain retryable
pending. The normal runtime advances queued native installs in bounded steps;
the last committed candidate is retained for section-owner recreation replay.

- `whole-section-candidate-native-install-producer-api-20261004/report.json`
  passed 8/8 headed GDExtension checks through the coordinator composition API:
  native section-slot receipt, old-slot retention until commit, stale-census
  rejection, and committed-candidate replay on a new native owner. Its terrain
  and ordinary providers are synthetic test authorities; it is not gameplay
  visual acceptance.
- `ordinary-static-section-provider-contribution-recheck-20261004/report.json`
  passed 15/15 checks. Ordinary structures return sealed source revisions,
  geometry inputs, compatibility and resource bindings matching their exact
  section census. This fixture does not prove production installation.
- `citadel-section-geometry-service-contribution-20261004/report.json` passed
  8/8 checks. Prepared Citadel packet groups enter the common immutable
  contribution shape with census-bound revisions. This does not install them
  or authorize retirement of current building visuals.
- The broad Playtest runner remains unresolved: the headless dummy renderer
  emitted RID initialization and null-mesh errors during nearby-terrain loading,
  and no playtest report was produced. The owned-process watchdog confirmed no
  surviving runner processes. This is not live gameplay evidence.

The source-producer callsite and visual handoff remain open: terrain must supply
complete smooth-mesh/fluid revisions; tree, detail, rock/ore, forage and
underground-prop sources need exact census coverage and their render layers;
visible-section demand must drive composition without unbounded repeated scans;
and each producer must retain its old view until the section receipt is
accepted, then retire only matching visuals while keeping collision,
interaction, navigation, harvest and save authorities alive. Headed world
traversal, startup, edit/harvest, save/reload, and performance evidence are
required before the migration can advance to completion.

### Next implementation charter — source-change invalidation

**Outcome:** when any admitted static source changes after section installation,
enqueue the affected visible section for a complete replacement candidate while
leaving its current native slot visible. Publish removal/empty results only
after current census, contribution, layer upload, and native receipt all agree.

**Authorities and scope:** terrain section revisions originate in
`VoxelTerrainRuntime`; generated building revisions/removals originate in
`StructureSystem` and the Citadel publication authority; tree and ecology
changes originate in `TreePublicationQueue`, the realized ecology ledger, and
durable prop-removal revisions. These authorities identify the exact changed
source and affected render sections. `WorldStaticSectionCoordinator` owns only
the retryable per-section invalidation queue and candidate-generation fence.
It must not take over generation, collision, interaction, navigation, or save
ownership. Keep mobs and NPCs independent. Minecraft's section dispatcher
recompiles a reassigned section and retains its previous meshes until layer
uploads acknowledge; apply that transactional replacement property, not its
block mesher.

**Stages and proof:** (1) add an exact source/provider/section invalidation API
and a focused coordinator contract for source changes, stale staged work,
repeat invalidation, and unchanged old-slot receipt; (2) route the existing
terrain signal and realized static-prop/tree removals or revisions through it
without scanning all loaded chunks; (3) use a real native receipt test to prove
that the new complete generation replaces the old slot and that a stale worker
cannot promote; (4) add a headed edit/harvest/tree-change visual check. Unknown
affected sections remain pending/retryable; never approximate them as empty.
This increment cannot claim unload/replay/save parity, full producer cutover,
traversal acceptance, or migration completion.

**Exact affected-section contract:** on accepted candidate installation, keep a
source-ID-to-section reverse index derived from that candidate's complete
manifest. Replace its entries only after the new section receipt is accepted.
A removal or revision event can then invalidate the prior sections containing
that source without enumerating all loaded gameplay chunks. A new or moved
source also supplies its current geometry bounds so every intersected visible
section is demanded; an unknown bound or missing source membership remains
pending and must not be collapsed to an empty replacement. If an affected
section currently has no visible demand, retain a durable dirty-source marker
for its installed slot; do not discard invalidation when the live demand record
is withdrawn. Unload/replay must revalidate that marker against the provider
census and rebuild before replaying the stale candidate. This guard must cover
both production section candidates and the older committed-candidate replay
queue, including already active replay sessions. Before clearing dirty state,
verify the accepted receipt still belongs to the live section owner/backend.
Terrain mesh-block
revision signals already identify exact 3D sections and continue through their
existing path. The ecology harvest writer must submit its stable recorded
source ID/bounds before it frees the gameplay prop body; collision, harvesting,
and durable removal still remain under their current owners.

**Implementation progress (2026-10-04, updated):** the coordinator now derives its
accepted source-to-section reverse index from each installed candidate's
`candidate.snapshot.manifest` (`sourceId` / `sourceRevision`), replacing index
entries only after the matching native install receipt. This deliberately does
not use candidate `sourceRevisions` keys: those are source-part IDs and are a
different identity. The prior 18-check visible-demand contract covered unequal
source/source-part IDs, demanded invalidation, old receipt retention, and index
replacement; the prior 26-check realized-prop capture contract covered stable
source IDs and world bounds. A read-only lifecycle review then found a
withdrawn-demand replay bug: the top-level resolver skipped undemanded sections,
and a recreated owner could replay the old accepted candidate. The coordinator
now records dirty source revisions even without live demand and suppresses
production-candidate replay until a fresh demand assembles a replacement. The
earlier 19-check report
`artifacts/citadel-runtime-integration/visible-section-demand-driver-dirty-replay-20261004-r4/report.json`
covered only the production-candidate route. A follow-up review then exposed
legacy queued/active replay bypasses plus promotion without a current-owner
receipt check. Those are now guarded too: invalidation cancels active and
queued legacy replay; stream unload/reload and replay admission preserve dirty
state; legacy replay advancement defers dirty sections; and production
candidate promotion verifies the receipt still belongs to the live backend
before clearing dirty state. The expanded 19-check report
`artifacts/citadel-runtime-integration/visible-section-demand-driver-source-invalidation-20261004-r2/report.json`
passes with every check green, including withdrawn demand, unload/reload,
queued/active replay cancellation and a deliberately late active replay. This
is coordinator lifecycle evidence, not a real stream-recreated visual proof.
Harvest's existing `complete_destroy_target` path submits source identity,
bounds, and durable removal revision before the prop body is retired;
NPC/navigation, rewards, collision and save ownership remain in their existing
systems.

The headed production diagnostic was rerun after adding dirty-source tracking
using `node tools/run-playtest.mjs -Only
production_section_candidate_diagnostic -Seed
source-invalidation-proof-20261004 -Visible true -ReportPath
artifacts/chunk-owned-rendering/dirty-source-replay-live-20261004-r1/playtest-report.json
-ProgressPath
artifacts/chunk-owned-rendering/dirty-source-replay-live-20261004-r1/playtest-progress.txt
-ScreenshotPath
artifacts/chunk-owned-rendering/dirty-source-replay-live-20261004-r1/playtest.png
-- --skip-tutorial`. It installs a current native receipt for `(0, 1, 0)` at
generation 9 after a complete 47-contributor census. Its accepted demand
records a tree-source invalidation during startup, then reaches `installed`;
the owned watchdog exits 0, proves zero job members, and reports
`cleanupPassed=true`. This proves producer notifications can coexist with real
startup section assembly/installation; it does not prove that a previously
installed slot was replaced after a real harvest or edit. The inspected
screenshot remains too dark and occluded by the spawn tree and held item for
visual acceptance. A prior underground scan-revision conversion error was
corrected from `String(...)` to `str(...)`.

The same headed command was rerun after the replay and live-receipt guards at
`artifacts/chunk-owned-rendering/dirty-source-replay-live-20261004-r2/`.
It passes in the Main production scene, installs generation 19 for section
`(0, 1, 0)`, and records a current native backend receipt after the selected
tree-source revision changed during startup. The owned watchdog at
`artifacts/node-tools/process-runs/godot-MEzUp0/watchdog.json` reports exit 0,
authoritative zero job members, and cleanup passed. The receipt records all
four required providers (`terrain`, `blueprint_buildings`,
`ordinary-structures`, and `ecology_and_static_props`); the selected section
contains 14 installed opaque batches / 172 instances and an explicit empty
translucent layer, with ordinary structures explicitly empty in that section.
This verifies a real renderer install of the shared cross-domain candidate and
current-owner receipt gate, but still does not perform a real harvest/edit
replacement. The runner's default headed screenshot/progress
artifacts are reused and overwritten by later runs; no visual-parity image is
being claimed or retained as acceptance evidence.

The separate seed `source-additions-diagnostic-20261004` remained at
`Scanning 360° view · 5/32 prop sources` for over three minutes. Its production
startup-failure dictionary was empty: this was still pending when the headed
playtest observer reached its own 240-second observation limit, not an
authoritative game failure. The playtest failure report now includes current
visible-world prop-capture and readiness diagnostics. Its loading screenshot
and progress were later overwritten by the runner's default paths and are not
preserved. The run therefore does not establish
why that seed's sixth prop source is slow. Real headed harvest/edit replacement,
stale worker rejection in the production path, legacy committed replay while
dirty, building edits, gameplay visual parity, save/reload, traversal and
performance acceptance remain open. This is an active stage, not a
migration-complete gate.

**Ordinary-structure source invalidation (2026-10-04):**
`StructureSystem.generated_visual_block_removed` now advances the durable
source revision/tombstone before notifying `MainPropFactory`'s existing
`invalidate_static_render_source` bridge with the removed mesh's world bounds.
The section system can therefore invalidate prior installed source membership
and build a replacement while collision, interaction, navigation and save
tombstone ownership stay in their existing systems. The source contract passes
14/14 at
`artifacts/citadel-runtime-integration/ordinary-structure-visual-source-invalidation-20261004-r1/report.json`;
the coordinator source-invalidation/replay contract passes 19/19 at
`artifacts/citadel-runtime-integration/visible-section-demand-driver-source-invalidation-20261004-r2/report.json`.
This is focused synthetic integration with the existing invalidation API. A
headed ordinary-block removal followed through section replacement and visual
retirement has not yet been run. Generated building packet retirement,
gameplay visual parity, save/reload, traversal and performance acceptance remain
open. This is an active stage, not a migration-complete gate.

### Next implementation charter — realized ecology prop capture

**Outcome:** extend the existing seeded ecology value authority with the
actual realized visual inputs for surface rocks, ore clusters and forage, plus
their underground counterparts. Record creator outputs after normal gameplay
creation has made its existing decisions. The section census may declare one of
these families empty only after that producer's bounded spawn/scan work is
complete for the exact chunk and source revision; unavailable scan results,
unsupported material semantics or missing member recipes stay pending.

**Authoritative path and boundaries:** surface candidates are selected by
`MainPlaytestTools.spawn_chunk_prop_attempt`; underground candidates are
selected by `process_underground_chunk_prop_spawn_state` from the authoritative
exposed-floor scan and terrain-volume revision. Existing `make_rock`,
`make_ore_cluster`/`MainInteractionFlow.make_ore`, and
`MainInteractionFlow.make_forage` calls remain the identity, transform,
material, collision, nav, harvest and drop authorities. Capture each realized
visual recipe/material/transform and stable member ID from the returned
producer result or a value emitted by its creator. Do not scan instantiated
scene trees, replay RNG, change creator order, or move collision, interaction,
navigation, harvesting, removal or save ownership into the render adapter.
Wildlife stays an actor. Tree geometry remains the `TreePublicationQueue`
snapshot; flower/detail contributors without a bound supported layer remain
pending.

**Acceptance evidence:** focused deterministic contracts prove exact realized
member IDs, transforms, mesh/material bindings, chunk/scan provenance, explicit
empty-family proof, removed-prop tombstones, unload/replay from seed plus
durable removals, and rejection after a terrain/source revision change. Add an
RNG-order regression comparing the existing creator path before and after
capture. A passing capture contract does not prove complete ecology census,
native section installation, visual parity, live traversal or performance; all
remain open for the later whole-world gates. Preserve `artifacts/` as ignored
test evidence and stage only reviewed task files, never generated `.import`
churn or the unrelated visible-world readiness edits in this repository.

**Surface versus underground readiness:** the shared section provider must
seal a surface-only value snapshot as soon as the deterministic surface prop,
tree-membership, and detail passes are complete, without sealing or losing the
still-running underground ledger. Aboveground section candidates may defer
`underground_props` explicitly because terrain hides them; they must not claim
that family is empty. When the player is underground, a surface-only snapshot
is pending until the authoritative exposed-floor scan and underground prop
publication complete. Replace the partial surface snapshot with the final
full snapshot after that scan; both remain bound to terrain and durable-removal
revisions. Focused acceptance must prove this split and the transition into
underground demand before attempting another long production-install run.

### Next implementation charter — multi-surface static render candidates

**Outcome:** represent every admitted static source as the exact set of its
mesh surfaces, materials and render layers so a whole section candidate can be
compiled and installed without rejecting valid production assets or merging
different material semantics. This immediately addresses the production
`flowerStem` / `flowerBloom` detail mesh, whose creator returns two surfaces
with different materials.

**Authority and boundary:** keep the current deterministic detail creator,
mesh, transforms, colors, custom data, source IDs, gameplay collision,
interaction, harvest and save authority. At capture time, derive immutable
surface fragments from the actual mesh surface arrays and bound material
resources; bind each fragment to the original source ID plus stable surface
index, its exact material key, primitive, transformed bounds, and render layer.
Do not infer a material from a detail-family label, consume RNG, duplicate the
whole mesh once per advertised material, or let a fragment acknowledge a
different source revision. Missing/unsupported surface material or primitive
semantics keep the section pending. Deduplicate the resulting fragments only
by their complete geometry/material/layer identity.

**Minecraft reference and stages:** in Minecraft 26.2,
`SectionCompiler` produces section geometry grouped by actual render layer,
and `SectionRenderDispatcher` installs the replacement only after all expected
layer uploads acknowledge while the old section remains visible. Apply those
layer grouping and transactional replacement principles while retaining this
game's smooth voxel terrain mesher. Stage A: add pure contract coverage for
surface enumeration, material identity, per-surface bounds/layer, deterministic
fragment IDs, and rejection of missing material or stale source revision.
Stage B: capture the live creator's actual flower mesh surfaces into the
section candidate and prove parity with the existing visible MultiMesh.
Stage C: install those candidates through the real native renderer, with a
receipt for every expected layer and the prior section retained through a
stale or failed replacement. Stage D: headed visual/traversal and performance
checks. These stages do not close source-change invalidation, unload/replay,
terrain/fluid parity, building coverage, save/reload, collision or interaction
authority, or full migration acceptance.

**Implementation finding (2026-10-04):** the flower mesh really has two
surfaces, `detailGrass` and `detailFlower`, and its `ShaderMaterial` is opaque:
`detail_material.gdshader` has no blending, `ALPHA`, or `discard` path. The
previous ecology digest delegated to the generic Citadel material identity,
which correctly refused the shader `Resource` reference and therefore kept all
flower candidates pending. Production capture now records one stable source
fragment per actual mesh surface and derives the render layer from its bound
material semantics. The ecology adapter contract passes 33/33 checks, including
shader code/uniform digest identity and both flower surfaces. A new headed,
skip-tutorial diagnostic is still in its real source-census wait; no native
section installation or visual parity is claimed by these focused checks.

### Next implementation charter — generated prop asset member capture

**Outcome:** generated GLB-backed rocks and other eligible static props expose
the exact render members needed by the section provider when the normal asset
creator selects and publishes them. A selected scene with no usable mesh
members, unsupported surface material, or incomplete imported resource binding
must keep its ecology source pending rather than claiming the family complete.

**Authority and ownership:** `VisualAssetRegistry` remains the asset selection,
PackedScene, and render-policy authority; the existing prop creator remains the
owner of stable prop ID, recipe transform, `StaticBody3D`, collision, navigation,
interaction, harvesting, and saved removal. The registry's selected-asset
capture API must return a stable, immutable value manifest of source member ID,
mesh surface, material, local transform, bounds, and render semantics to the
creator. Capture from the selected asset's declared scene representation under
registry ownership, not by scanning world/chunk nodes, replaying selection, or
asking the section adapter to discover scene contents. Bind each returned
resource pair to the realized prop source and current asset/mesh/material
revision, including the contents of referenced textures. Do not replace the
current generated rock art with a primitive to satisfy capture. Headless
imported-scene proxies remain pending for geometry.

**Stages and proof:** (1) add an asset-manifest contract covering a nested,
multi-surface scene, stable member identities, exact composed transforms,
materials/layers/bounds, missing imports, and stale resources; (2) attach the
registry manifest to the existing prop creator output without changing asset
selection, seed/RNG order, collision or gameplay behavior; (3) prove those
members enter the full section census/contribution and native slot while the
old per-prop view remains until receipt; (4) headed prop visuals, movement,
harvest/removal, save/reload, and performance. This does not complete terrain,
buildings, tree geometry, unload/replay, or whole-world migration gates.

### Next implementation charter — canonical tree request revision checks

**Outcome:** a tree enters a static section census only when its queued request,
captured recipe, installed body metadata, and current LOD agree under the same
canonical `TreeSpawnService` normalization used to build the recipe. Raw caller
inputs may contain values that the service clamps or defaults; recomputing a
signature directly from that raw dictionary must not reject a valid published
tree or admit a stale one.

**Authority and boundary:** `TreeRuntimeRequestBuilder` supplies the stable
world/seed/ecology request; `TreeSpawnService.normalize_request` is the current
canonicalizer and owns recipe identity. `TreePublicationQueue` owns the current
accepted request/recipe/body/LOD receipt; `TreeSectionValueAdapter` validates
that receipt before producing immutable render inputs. Preserve tree IDs,
procedural recipe/RNG order, geometry and body collision, navigation, removal
and save ownership. Do not weaken signature equality, substitute a fresh recipe
for the committed one, or make missing queue geometry look complete.

**Stages and proof:** (1) normalize the queue's immutable request through the
service before recomputing its signature; retain bounded raw-versus-canonical
signature diagnostics for pending mismatches; (2) make the focused tree fixture
construct its expected recipe with the same normalization and prove stale body,
recipe, request, or tier identities remain pending; (3) run the headed real
section-census diagnostic and confirm this tree now advances to the next actual
provider blocker. This does not prove installed section rendering, tree visual
parity, traversal, save/reload, or full migration acceptance.

**Implementation finding (2026-10-04):** the adapter now normalizes the
queue-acknowledged request through `TreeSpawnService` before recomputing the
recipe signature. The focused tree value-adapter contract passed 11/11. A headed
no-tutorial production diagnostic on seed `section-roster-diagnostic-20261004`
advanced beyond the tree mismatch and stopped at terrain census admission,
`terrain_exact_fluid_section_probe_pending`; it did not queue a native install.
The run exited with code 1 and the owned-process watchdog proved zero remaining
job members (`cleanupPassed: true`). This confirms the tree revision check only;
it does not prove a render install or the generated-rock capture hidden behind
the provider ordering.

### Next implementation charter — retry ownership for deferred terrain contributions

**Outcome:** a deferred terrain section contribution remains retryable with its
complete world, section, provider, coverage, and source revision identity. A
retry must not be mutated when the current attempt clears its working state.

**Authority and boundary:** `TerrainSectionShadowPublisher` owns its bounded
contribution queue and active work item; `VoxelTerrainRuntime` remains the
terrain/fluid authority and `WorldStaticSectionCoordinator` owns candidate
admission. Preserve exact-fluid revisions, the smooth-terrain capture, old
terrain visibility, collision and gameplay ownership. No “fluid absent” shortcut
or synchronous scan is allowed.

**Stages and proof:** (1) preserve a headed baseline before edits; (2) prove a
deferred active request re-enters the queue as an intact independent value,
including a later retry after the fluid proof changes to current; (3) rerun the
headed production candidate diagnostic to see whether terrain advances to
candidate preparation and native receipt. No visual, traversal, save/replay, or
performance acceptance is implied.

**Baseline (2026-10-04):** `node tools/run-playtest.mjs -Only
production_section_candidate_diagnostic -Seed section-roster-diagnostic-20261004
-Visible true -ArtifactDir artifacts/chunk-owned-rendering/production-roster-diagnostic-20261004-r19
-- --skip-tutorial` reached the production terrain contribution scheduler and
crashed at `TerrainSectionShadowPublisher.gd:240` while reading `sectionKey`
from an empty queued Dictionary. No playtest report was emitted. The owned
watchdog proved zero remaining job members, but `cleanupPassed` was false after
the runner requested termination. This failure predates the retry fix in this
stage; root cause is being verified against the enqueue/clear ownership path.

### Next implementation charter — terrain membership in a cross-domain section manifest

**Outcome:** terrain contribution admission validates the terrain-owned source
membership and revision in the complete section manifest while allowing
building, ecology, tree, and prop sources to share that section.

**Authority and boundary:** the source roster owns complete per-section source
membership; terrain owns its sole terrain source ID and fluid/volume revisions.
Do not use a global contributor count as a terrain-only count. Keep deterministic
terrain generation, Transvoxel geometry, collision and the existing surface
visible until the shared candidate receipt.

**Stages and proof:** (1) add a contract with one terrain source plus unrelated
building/ecology contributors in the same complete roster and prove terrain
capture accepts the roster while still requiring its exact source ID; (2) run
the focused terrain contribution contract; (3) run the headed diagnostic through
the normal visible-demand scheduler and inspect the first subsequent producer
or renderer blocker. Renderer receipt and visual/performance gates remain open.

**Finding (2026-10-04):** headed census evidence showed 22 cross-domain
contributors in one section, while terrain's adapter had a guard requiring the
complete section's expected contributor array to have size exactly one. Its
terrain-only fixture contained just one expected source, masking the production
manifest shape. A second headed retry run was stopped through its owned watchdog
after the demand remained waiting; it did not produce a final report, and the
watchdog proved zero job members but marked cleanup unsuccessful. This stage
will replace the total-count assumption with exact terrain source membership.

**Focused verification (2026-10-04):** the terrain contribution contract passes
9/9 after the provider guard now counts its own source ID in the complete
expected manifest. It also proves retry records survive defer/clear by value and
produce a sealed contribution after a current no-fluid proof is supplied. This
is synthetic producer/assembler evidence only; the headed native install is
still open.

### Next implementation charter — identify the production batch rejected by native append

**Outcome:** a native append failure reports the exact section batch/segment and
bounded payload facts needed to identify the invalid field. Fix only the
producer or packet boundary that violates the backend contract.

**Authority and boundary:** `NativeStaticSectionInstallSession` owns the mapping
from immutable shared snapshot batches to native append calls; the C++ packet
backend owns its payload acceptance rules. Keep complete candidate membership,
source identity, materials, meshes, visibility policy, and old-slot retention
unchanged while diagnosing.

**Stages and proof:** (1) preserve the headed batch rejection report; (2) attach
bounded failing batch/segment metadata to the returned diagnostic; (3) reproduce
the same seed and identify which backend precondition fails; (4) add a focused
contract for that field and only then fix its actual producer. A render receipt
is still required after the fix.

**Baseline (2026-10-04):** headed no-tutorial diagnostic
`node tools/run-playtest.mjs -Only production_section_candidate_diagnostic
-Seed section-roster-diagnostic-20261004 -Visible true -ArtifactDir
artifacts/chunk-owned-rendering/production-roster-diagnostic-20261004-r21 --
--skip-tutorial` captured the full 22-source, 29-input section candidate, queued
it through the live demand scheduler, and reached native append. The first
renderer append failed with `invalid_batch_payload`; no receipt or report pass.
See the runner's root `playtest-report.json` and owned watchdog report. This is a
real integration failure, not a synthetic producer failure.

**Finding (2026-10-04):** replay `r22` attached bounded append metadata to that
same real rejection: a 195-instance opaque `ArrayMesh` batch had finite,
positive bounds, 3900 floats (20 per instance), a 64-character mesh digest, and
the `ShaderMaterial`; the renderer rejected its `renderTier: "near"`. The native
contract supports `silhouette`, `structural`, `detail`, and `horizon`. The tree
adapter had forwarded procedural LOD tier (`near/mid/far/impostor`) into the
renderer category field. Its report is `playtest-report.json`; the owned
watchdog proved zero remaining job members and cleanup passed.

### Next implementation charter — keep procedural LOD separate from renderer category

**Outcome:** tree geometry carries both its committed recipe LOD and the
renderer-supported semantic category for each member role. LOD still owns the
recipe, visibility, and deterministic source revision; renderer category owns
only the batch/material submission classification.

**Authority and boundary:** `TreeSpawnService` remains LOD/recipe authority;
`TreeSectionValueAdapter` maps the existing bole/branch/foliage role to the
backend category contract. Use structural for collision-relevant trunk/branch
visuals and detail for foliage. Preserve exact tree geometry, LOD visibility,
collision body, interaction, harvest, navigation, removal, and save identity.
Do not broaden C++ tier acceptance to include LOD names.

**Stages and proof:** (1) add a focused captured-tree contract asserting exact
role-to-category mapping and retained LOD revisions; (2) run the tree and
section installation contracts; (3) replay the headed production candidate and
require the installed receipt, then inspect visual parity before any retirement.

**Implementation finding (2026-10-04):** the focused tree contract passes 12/12,
and the headed production diagnostic moved past `invalid_batch_payload` to
`duplicate_batch_id`; it still has no install receipt. The next failure is in
the merged snapshot path, not tree LOD mapping.

### Next implementation charter — stable identities for merged native batch segments

**Outcome:** every final merged section-batch segment has a deterministic,
nonempty identity unique within its batch, including batches coalesced from
multiple source contributors or split at the native instance limit.

**Authority and boundary:** `ChunkStaticRenderSectionSnapshot` owns stable
snapshot segment identity after compatible inputs are merged. The install
session combines that segment identity with the compatibility batch key for
native uniqueness. Preserve source-range manifests, merged buffer order,
content digest determinism, and whole-candidate transaction semantics.

**Stages and proof:** (1) extend the snapshot contract to require deterministic,
unique merged segment IDs under input reordering and native-limit splitting;
(2) run the snapshot, builder, and controlled native installation contracts;
(3) replay the same headed production candidate and require a receipt. Any
visual parity, traversal, and performance work remains a separate gate.

**Baseline (2026-10-04):** headed `r23`, same command/seed as `r22`, captured a
complete 22-source/29-input candidate, advanced visible demand, and failed at
native append with `duplicate_batch_id`. Append telemetry identified a repeated
batch key with an empty merged segment ID. No receipt was committed. The owned
watchdog proved zero remaining job members and cleanup passed.

**Implementation finding (2026-10-04):** `VisualAssetRegistry` now emits
surface member values during its existing render-policy traversal of a selected
rock asset, and the normal rock creator binds those rows to the realized
prop's source ID, body transform, and live gameplay body. Imported mesh
fragments preserve each actual surface material. Standard and shader material
digests now bind supported parameters and referenced texture pixels. The
existing canopy/import contract passed 14/14; the later headed production
section census also reached all 22 source contributors and native installation
for its selected section, so live section publication includes its captured
rock/foliage/tree contributors in the candidate manifest and receipt. The
headed diagnostic does not yet prove old-visual replacement/retirement, terrain
collision/fluid/light parity, save/reload parity, traversal, or performance.

**Implementation finding (2026-10-04):** `ChunkStaticRenderSectionSnapshot`
now assigns each merged/split output segment a deterministic ID derived from its
compatibility batch key and output ordinal. The focused immutable snapshot
contract passes 29/29, including nonempty/unique IDs on a native-limit split and
repeat assembly stability. The headed no-tutorial main-scene diagnostic
`node tools/run-playtest.mjs -Only production_section_candidate_diagnostic
-Seed section-roster-diagnostic-20261004 -Visible true -ArtifactDir
artifacts/chunk-owned-rendering/production-roster-diagnostic-20261004-r24 --
--skip-tutorial` passed with a current native chunk-renderer receipt for section
`(-1, 0, 0)`, generation 29, 22 complete source contributors and 29 captured
inputs across terrain, blueprint buildings, ecology/static props, and ordinary
structures. `playtest-report.json` records `receiptBackedInstall: true`; the
owned watchdog at `artifacts/node-tools/process-runs/godot-NjWsJE/watchdog.json`
records exit 0, authoritative zero job members, and cleanup passed. This is
production renderer installation evidence for one section in `Playtest.tscn`,
not full-world visual/gameplay acceptance.

### Next implementation charter — validate section replacement, stale rejection, and old-visual retention

**Outcome:** when a current section candidate replaces an installed generation,
the old generation remains visible until the renderer acknowledges the whole
replacement; a stale generation or mismatched owner/backend receipt cannot
publish or retire the current representation. Unload/replay must remove the
section's static visual packet while preserving independent terrain collision,
gameplay interaction/save authorities, and independent mob/NPC simulation.

**Authority and boundary:** `WorldStaticSectionCoordinator`,
`NativeStaticSectionInstallSession`, and the native chunk renderer own candidate
generation, install acknowledgement, and visual slot replacement/retirement.
TerrainVolumeService and existing building/ecology/prop/structure systems remain
the authority for collision, edits, interaction, deterministic generation, and
durable save deltas. Do not detach those contracts merely to prove rendering.

**Stages and proof:** (1) inspect the current receipt, install, removal, and
replacement APIs end to end and compare their revision/backend/owner guards;
(2) add or refine a production-boundary contract for stale generation rejection,
failed partial replacement preserving the previous slot, current replacement
publishing before prior-slot retirement, and unload/replay idempotence; (3) prove
the sequence through a headed native renderer fixture with actual prior/current
visual generations and screenshots or visible state evidence; (4) exercise one
real source revision/edit and reload path to ensure gameplay/save authority is
unchanged. Do not begin full traversal/performance acceptance until these
lifecycle checks expose no unsafe replacement path.

**Entry evidence:** same-seed headed section candidate install passed at `r24`
with current native receipt, after the split-segment identity contract passed
29/29. No old-slot replacement, stale completion, unload/replay, edit/save,
traversal, or runtime performance claim exists yet.

**Lifecycle finding (2026-10-04):** the controlled headed native candidate
fixture now passes 8/8 after the snapshot segment-ID change. Its generation 2
replacement keeps generation 1 installed until commit acknowledgement; changing
the provider revision after generation 3 has already entered native append
cancels the staged candidate and leaves the generation 2 installed root ID
unchanged; unloading the source chunk and recreating its renderer backend then
replays the last accepted generation 2 candidate. Report:
`artifacts/citadel-runtime-integration/whole-section-candidate-native-install-stale-replacement-20261004-r5/report.json`.
The first attempt (`r4`) did not load because the added fixture expression had
an inferred-Variant type error; it was corrected before the passing run. These
are headed fixture checks with native rendering resources, not real-world
source edit/save or player traversal evidence.

**Visual observation (2026-10-04):** the production r24 screenshot is preserved
at `artifacts/test-runners/playtest.png`. It shows the diagnostic running in a
forest, but heavy dark foliage/shadow coverage and a large dark held-item shape
make it unsuitable as visual-parity acceptance. This remains a gameplay visual
issue to investigate separately from the successful native receipt.

**Checkpoint commit:** the game implementation and current stage evidence are
committed on `codex/chunk-owned-world-rendering-migration` as
`fd389f48` (`Wire whole-section candidates into production rendering`). The
commit intentionally excludes the 204 changed generated `.import` sidecars.
Focused validation also passes the 15-check demand driver contract at
`artifacts/citadel-runtime-integration/visible-section-demand-driver-migration-20261004-r1/report.json`
and the 24-check realized-prop contract at
`artifacts/citadel-runtime-integration/ecology-realized-prop-capture-migration-20261004-r2/report.json`.
The realized-prop run `r1` failed because its expectation used the wrong reason
string; production already correctly returned `pending`, an empty render layer,
and `shader_prop_render_semantics_unsupported` for the unknown shader. The
assertion was corrected to the implementation's exact contract and rerun. No
production material fallback was added.

### Next implementation charter — terrain edit invalidation of installed sections

**Outcome:** every authoritative terrain geometry edit that intersects a live
static render section re-demands that section and any smooth-mesher halo
neighbors whose captured inputs changed. A stale in-flight candidate cannot
replace the installed receipt; the old section and Voxel Tools terrain/collision
remain available until a fresh complete candidate is acknowledged.

**Non-goals:** do not disable or retire Voxel Tools terrain publication; do not
claim fluid-layer support, visual parity, save/reload parity, traversal, or
performance acceptance from this slice. Mobs/NPCs remain independent.

**Authority and consumers:** `TerrainVolumeService` is authoritative for edited
cell/material/fluid/light state and section revisions. `VoxelTerrainRuntime`
owns the resident native sample buffers, smooth-mesher halo capture, collision,
and terrain candidate admission. `MainRuntimeTools` bridges changed resident
terrain-block revisions to `WorldStaticSectionCoordinator`; the coordinator
owns demand, stale-source rejection, and installed section receipts. Save
snapshots continue to persist durable terrain deltas from `TerrainVolumeService`.
The changed-cell/batch path must notify the bridge only after authoritative
revision mutation and must not bypass edit batching or collision publication.

**Baseline:** game branch `codex/chunk-owned-world-rendering-migration`, HEAD
`88e495a1`; existing generated `.import` churn is pre-existing and excluded.
The headed Main-scene diagnostic has installed a current native cross-domain
section receipt, and focused static-source invalidation contracts pass. The
terrain audit found that cell edits update authoritative revisions but do not
currently trigger visible-section demand; only native mesh-block enter/exit
does. Current terrain candidates still reject exact fluid and retain Voxel
Tools visuals/collision.

**Stages and exit evidence:** (1) identify every authoritative terrain mutation
path and its affected SDF sample/core/halo sections; (2) expose a bounded,
value-only invalidation event from the terrain authority/runtime and connect it
to existing visible demand without making absent/unloaded sections permanent
demand; (3) contract-check resident core+halo invalidation, stale-candidate
rejection, retained old receipt, and retryable fresh demand; (4) run a headed
production terrain edit through the live section candidate and require a newer
current renderer receipt while proving native collision and saved delta remain
authoritative. Report each passed/failed/blocked/untested row separately.
Fluid support, explicit empty terrain replacement, Voxel Tools visual handoff,
visual parity, traversal, save/reload, and runtime performance remain separate
gates; this charter is not a stage-completion claim.

**Implementation finding (2026-10-04):** `TerrainVolumeService` now emits a
value-only dirty-cell bound with each terrain section revision, and
`VoxelTerrainRuntime` maps that bound through the Transvoxel one-low/two-high
sample halo before bumping resident mesh-block revisions and re-demanding their
section candidates. The edit batch emits a second notification after its
`VoxelTool.paste`, binding reassembly to installed voxel data rather than only
the earlier durable-volume update. Revisions for already-installed candidates
are marked urgent so first-time distant content cannot starve replacement.
The focused visible-demand driver passes 21/21 at
`artifacts/citadel-runtime-integration/visible-section-demand-driver-exact-terrain-halo-20261004-r1/report.json`;
it covers an eight-section boundary-cell halo and urgent recompile selection
ahead of nearby initial candidates.

**Headed result (2026-10-04):**
`node tools/run-playtest.mjs -Only production_section_candidate_diagnostic
-Seed terrain-section-refresh-proof-20261004-r5 -Visible true -ReportPath
artifacts/chunk-owned-rendering/terrain-section-edit-refresh-20261004-r5/playtest-report.json
-ProgressPath artifacts/chunk-owned-rendering/terrain-section-edit-refresh-20261004-r5/playtest-progress.txt
-ScreenshotPath artifacts/chunk-owned-rendering/terrain-section-edit-refresh-20261004-r5/playtest.png -- --skip-tutorial`
installed an initial generation-30 cross-domain candidate with a live native
receipt, then performed a real `WorldGenerationSystem.set_cell_state` edit.
The edit reached resident VoxelData, appears in its durable section delta, and
leaves Voxel Tools collision generation enabled. At the end of 3,600 frames the
selected demand still reported `waiting`; generation 30 and its native receipt
remained live, so the changed section did not receive a fresh renderer receipt.
The runner exited 1 with cleanup passed and authoritative zero job members.
This fails the replacement gate; it is not implementation-complete evidence.
The screenshot is gameplay-visible but not visual-parity evidence: strong dark
canopy/shadow coverage and the held item obscure much of the view. A separate
headless retry failed during test setup because the project's native mesh path
is unsupported by Godot's dummy renderer (uninitialized mesh RID); it is not
counted as a production result.

**Follow-up (2026-10-04):** source invalidations and changed terrain revisions
are separate re-demand entry points. `invalidate_visible_section_source` now
marks a section with an installed candidate as an urgent recompile and gives it
priority ahead of first-time nearby sections, while retaining the installed
candidate. The focused demand contract passes 21/21 after this change at
`artifacts/citadel-runtime-integration/visible-section-demand-driver-source-invalidation-priority-20261004-r1/report.json`.
The source invalidation assertions cover the urgent flag and priority in
addition to retained generation/receipt.

The follow-up headed run `r6` used the same production diagnostic and skip-
tutorial launch path, but it did not reach the edit gate. It spent 600 seconds
waiting for `production_section_candidate_waiting_terrain_empty_section_requires_exact_empty_manifest`,
then the outer runner timed out without a playtest report. The owned-process
watchdog records forced cleanup and authoritative zero job members, but
`cleanupPassed` is false due to timeout. Treat r6 as a stalled headed-test
readiness/setup result; it neither proves nor disproves the urgent replacement.
Report/progress paths are
`artifacts/chunk-owned-rendering/terrain-section-edit-refresh-20261004-r6/playtest-report.json`
(not produced) and `playtest-progress.txt`; watchdog:
`artifacts/node-tools/process-runs/godot-0HM095/watchdog.json`.

The representative performance observation
`node tools/run-runtime-performance-observation.mjs -Diagnostic true -DurationSeconds 45 -WarmupFrames 120 -ReportPath artifacts/performance/chunk-owned-terrain-edit-invalidation-20261004/runtime-observation.json -ProgressPath artifacts/performance/chunk-owned-terrain-edit-invalidation-20261004/runtime-observation-progress.txt -Visible true -TimeoutSeconds 300`
also failed before collecting samples: `DayWork` reported
`startup_not_ready`, and `startupLoadingFailureResult` was empty. Its owned
process shut down naturally with cleanup and zero membership proven. Classify
this as an unfulfilled runtime-startup/performance gate, not a performance
measurement or an attributed regression. The fresh terrain renderer receipt,
visual handoff, save/reload, traversal and representative performance gates
remain open.

**Code checkpoint:** the terrain edit invalidation and replacement-priority
slice is committed in game commit `e2811ae8` on
`codex/chunk-owned-world-rendering-migration`. Generated/import `.import`
churn remains excluded and untouched. This is an intermediate stage commit;
the terrain replacement and whole-migration gates above remain open.

### Next implementation charter — guaranteed scheduling for visible replacements

**Outcome:** an urgent replacement for an installed visible section gets a
bounded production admission opportunity during ordinary gameplay even when
the shared publication frame is occupied. Continue retaining the previous
section receipt until a current whole-section replacement is accepted.

**Non-goals:** do not bypass provider completeness, native receipts, stale-work
rejection, the shared gameplay budget, or queue backpressure; do not make
capture synchronous as a loading fallback; do not change terrain generation,
fluid support, collision authority, or actor/NPC scheduling.

**Authority and path:** `TerrainVolumeService` -> `VoxelTerrainRuntime` ->
`MainRuntimeTools.on_visible_terrain_mesh_section_revision_changed` ->
`WorldStaticSectionCoordinator` -> complete provider capture -> native section
install session. `MainRuntimeTools` currently calls candidate admission only
when lane 2 of its shared 6 ms publication schedule has more than 2 ms
remaining, after structure and prop work. The coordinator tracks urgent
replacement demands but the current headed evidence does not report whether an
admission attempt was made during the edit window.

**Baseline and discovery:** game commit `e2811ae8`, branch
`codex/chunk-owned-world-rendering-migration`. Focused demand lifecycle tests
pass 21/21. The headed same-seed run `r8` at
`artifacts/chunk-owned-rendering/terrain-section-edit-refresh-20261004-r8/playtest-report.json`
still has the edited, resident, saved cell at native candidate generation 20
with its old receipt live. Its demand is `urgentRecompile=true` and `queued=true`,
but target attempts remain zero; only seven total coordinator attempts occur
while 1,000 demands are pending over the 3,600-frame replacement window. The
provider was never asked to rebuild that section, so classify this as demand
admission/selection starvation, not a provider-pending result. Preserve these
bounded counters in the diagnostic. The queue is FIFO with a bounded 32-entry
scan, while production calls the shared lane-2 admission after structure and
prop work and only when more than 2 ms remains in its 6 ms frame budget.

**Stages and exit evidence:** (1) bounded admission/queue telemetry is now
available in the existing real headed diagnostic; (2) put urgent rebuild
demands at the front of a bounded priority path and reserve one lane-2
admission opportunity before unrelated structure/prop work, without stealing
work from player-safety collision or bypassing provider/receipt checks; (3)
contract-test normal and saturated lanes, pending-provider retry, and
retained-old receipt; (4) repeat the headed same-seed edit and require a newer
live receipt while checking resident voxel data, saved delta, and collision publication. Treat
visual handoff, save/reload, traversal, and representative performance as
separate gates. No implementation-complete claim until those gates and later
migration stages pass.

### Next implementation charter — current ecology snapshot during section capture

**Outcome:** when a whole-section candidate is assembled, the ecology provider
captures a value snapshot whose chunk source revision and removed-props revision
still match the authoritative chunk at admission and install. If that input
changes, retain demand and retry from a newly published chunk snapshot.

**Non-goals:** do not accept stale snapshot bytes, relax the complete source
census, regenerate gameplay props or consume RNG from the renderer, or retire
the existing ecology publisher. Preserve prop IDs, harvest/removal, save
deltas, canonical tree recipes, and actor/NPC simulation.

**Authority and path:** chunk gameplay/prop state and removed-prop deltas feed
the static ecology source value snapshot on the chunk owner; `EcologySectionValueAdapter`
validates that snapshot against `_ecology_chunk_source_revision` and
`removed_props_revision`; `WorldStaticSectionCoordinator` combines it with
terrain/building providers and hands a complete candidate to the native section
install session. A stale ecology revision is a retryable provider result, not
an install acknowledgement.

**Baseline:** game code checkpoint `e2811ae8`, plus the current uncommitted
urgent-lane/telemetry slice. Focused queue/halo contracts pass 22/22 at
`artifacts/citadel-runtime-integration/visible-section-demand-driver-priority-queue-20261004-r2/report.json`.
The headed same-seed run `r9` reaches the edited resident cell and keeps its
saved delta, collision publisher, and previous live native receipt. The urgent
section gets 113 admission attempts in 3,600 frames, but every observed
terminal demand reason is `ecology_chunk_source_snapshot_revision_stale`; no
new candidate is installed. The ecology adapter returns mismatch details,
but `StaticSectionSourceRoster.capture_sections` wraps pending provider output
and its `_pending` merge replaces the wrapper reason with the nested provider
reason while dropping the revision detail fields. First preserve the bounded
snapshot/current revision pair through that wrapper and determine whether prop
publication or removed-prop revision is changing the snapshot during capture.

**Stages and exit evidence:** (1) retain only the latest provider pending
details in the demand state and headed report; (2) identify and fix the source
snapshot publication/update contract while preserving authoritative content
and no-RNG renderer capture; (3) test stable capture, stale rejection, retry to
a refreshed snapshot, and old-receipt retention; (4) repeat same-seed headed
terrain edit replacement and require a newer live native receipt. Follow with
ecology removal/save replay, visual handoff, traversal, and performance gates.

**Diagnostic update (2026-10-04):** r11 confirms the urgent lane is working and
identifies the stale pair: the ecology chunk snapshot carries
`ecology-v1:terrain-section-refresh-proof-20261004-r5:0,0:terrain-0`, while the
authoritative current chunk revision is the same identity with `terrain-1`.
Both removed-props revisions are `0`, and the old snapshot passes its internal
validation. The terrain edit changed the source revision, but the completed
chunk ecology snapshot was not republished. The headed run made 113 target
admission attempts, all held pending by that stale source snapshot; its native
generation-8 receipt stayed live and the edited cell remained in VoxelData and
the saved delta. Report:
`artifacts/chunk-owned-rendering/terrain-section-edit-refresh-20261004-r11/playtest-report.json`.

The production lane and queue change also passes 24/24 focused contracts at
`artifacts/citadel-runtime-integration/visible-section-demand-driver-stale-provider-detail-20261004-r3/report.json`.
This advances demand selection but not terrain replacement. The next source
owner decision is how ecology values are refreshed from the durable terrain,
realized prop, and removal authorities after an edit without replaying mutable
RNG or accepting stale producer completeness. Until that contract is defined
and implemented, section capture must keep returning pending and leave the old
receipt installed.

The corresponding game-repository implementation is committed as
`e2811ae8` (terrain edit invalidation) and `545c467d` (urgent whole-section
replacement scheduling and diagnostics). The working tree also contains
pre-existing generated `.import` churn, excluded from both commits.

**Producer revision decision (2026-10-04, before implementation):** a loaded
physical chunk's ecology source identity must not be the terrain-volume
revision. The existing prop/tree/detail nodes remain the live producer output
while their chunk owner remains installed; terrain edits revise terrain and
invalidate section candidates, but do not mutate those props. Keep the
ecology producer identity stable for the same seed/chunk/producer schema and
validate snapshot content plus chunk-owner identity and removed-prop revision.
Continue recording the terrain revision at which each producer pass ran as
provenance, not as a blanket invalidation of already-installed physical output.
A replacement/unload of the chunk owner invalidates its snapshots; the newly
generated owner captures against current terrain and publishes a new content
revision. Horizon-only ecology is a different owner: its cache witness already
includes terrain revision and it retires/regenerates on edits, so keep that
dependency. Never relabel stale contents as fresh: tests must prove the
physical snapshot remains byte/content-identical across a terrain-only edit,
becomes stale on removed-prop revision or owner replacement, and a reloaded
chunk can publish changed deterministic content under the same stable producer
identity. The headed edit gate must then install a newer live native receipt
while retaining the old receipt until commit acknowledgment. If the existing
source owners do not expose a verifiable owner identity/content revision, stop
and extend discovery rather than weakening the stale check.

**Production implementation result (2026-10-04):** physical ecology source
identity now uses the stable `ecology-v2:<seed>:<chunk>:static-props-v1`
producer revision; each snapshot retains its terrain revision as provenance
and carries a runtime chunk-owner instance identity outside deterministic
content hashing. Capture requires that the current chunk map still points to
that exact owner and still checks removed-prop revision and the sealed content
digest. A terrain-only edit no longer invalidates unchanged installed ecology
values; owner replacement or durable removals still reject stale values. The
focused adapter contract passes all 39 checks, including those freshness
boundaries:
`artifacts/citadel-runtime-integration/ecology-section-value-adapter-producer-revision-20261004-r3/report.json`.

The real headed Main-scene diagnostic also passes on seed
`terrain-section-refresh-proof-20261004-r5` with `-SkipTutorial` forwarded as a
Godot user argument. Its complete census contains 49 contributors; generation
9 remains the old installed candidate until the edited-cell candidate reaches
a live native receipt at generation 37. The target cell was applied to resident
VoxelData, appears in the durable section delta, and collision publication
remained enabled. The owned process exited 0 with authoritative zero job
members. Exact report and screenshot:
`artifacts/chunk-owned-rendering/terrain-section-edit-refresh-20261004-r12/playtest-report.json`
and `artifacts/chunk-owned-rendering/terrain-section-edit-refresh-20261004-r12/playtest.png`.
The screenshot still has dense dark canopy/shadow coverage. This proves live
candidate install after an edit; it does not prove collision-contact, fluid or
light parity, legacy terrain-renderer retirement, save/reload replay, traversal,
visual parity, or performance. Keep Stage 3 partial until those exits pass.

The implementation is committed in the game repository as `26565d98`
(`Decouple ecology producer identity from terrain edits`). The r12 headed run
was made before that commit but against the exact committed production-source
changes; r3 adds a contract that same stable producer identity can carry a
different content revision after regeneration on a changed terrain revision.

### Next implementation charter — spatial ecology removal projection

**Outcome:** harvesting one durable static prop changes the ecology contribution
and replacement demand only for section bounds intersecting that prop. Other
resident chunks remain capturable even though the world-wide save removal
revision advanced. Reload/regeneration still filters the stable prop identity
from deterministic producer output.

**Non-goals:** do not split or weaken the durable `removed_props` authority,
change prop IDs/RNG, keep harvested visuals, re-render unrelated chunks, or
retire the old per-source publishers in this slice. Keep actor/NPC simulation
and collision/navigation removal under their existing gameplay owners.

**Authority and path:** harvest records `prop_id` in `removed_props`, increments
the world save revision, and invalidates that source's bounds. The resident
chunk's sealed ecology values contain stable source/prop IDs and content digests.
The section adapter must derive an immutable per-chunk removal projection from
those values plus the authoritative removed-ID set, then use that local
projection digest when binding source revisions. A removal absent from a
chunk's producer output must not stale that chunk; a present removed ID must be
projected out and represented as a tombstone before it can enter replacement
geometry. Save state keeps the global revision; producer source identity stays
seed/chunk/schema based. Candidate capture, coordinator install, and source
invalidation continue to use exact source bounds, complete census and owner
identity checks.

**Baseline:** game branch `codex/chunk-owned-world-rendering-migration`, HEAD
`19e52664`; tracked source is clean and generated `.import` churn is pre-existing
and excluded. Ecology adapter contract r3 passed 39 checks and proves stable
producer identity across terrain-only edits, but did not test unrelated versus
local durable removal. MainPropFactory's real harvest increments the global
revision and invalidates only one prop's bounds; `_capture_production_chunk`
currently requires every snapshot's global revision to equal the current one.
Latest production proof remains headed r12, report
`artifacts/chunk-owned-rendering/terrain-section-edit-refresh-20261004-r12/playtest-report.json`;
it proves terrain candidate replacement, not ecology harvest/replay.

**Stages and exit evidence:** (1) add a failing synthetic contract with two
resident chunk snapshots and one removed prop, proving unaffected-chunk source
revision stability and exact affected-chunk omission/tombstone; (2) implement
the adapter's content-verified removal projection and ensure save's global
revision is not misused as local source identity; test stale owner rejection,
replacement receipts and removed-ID replay; (3) run a real Main headed harvest
path and confirm affected section replaces while unrelated resident section
remains current and collision/interaction/save identity are unchanged; (4)
run appropriate visual/traversal and runtime performance coverage before using
this as Stage 5 acceptance. Preserve old accepted section contents until the
complete current candidate has a native install receipt. No migration stage is
complete from this focused fix alone.

**Implementation review note:** Minecraft 26.2's `SectionCompiler`,
`RenderSectionRegion`, and `SectionRenderDispatcher` support the candidate
transaction model: section inputs are copied, stale compilation is cancelled,
and old layer buffers remain installed until all replacement uploads complete.
This removal fix changes only source projection/invalidation; Minecraft's block
mesher is not applicable to our smooth terrain or procedural tree meshes.

**Implementation evidence (2026-10-04):** the ecology adapter now captures the
authoritative removed-ID set as a stable value for the duration of section
census and projects matching detail/static-prop/tree candidates out before
section membership and contribution assembly. The global save removal counter
is no longer part of unrelated candidate/tree source revisions; exact removed
source IDs create source-specific section tombstones. Chunk owner, source
schema, snapshot content digest, and removed-set capture freshness checks remain
required. The capture path must enumerate only candidate prop IDs belonging to
the chunk and use bounded per-ID snapshots against the authoritative map; it
must not rescan/copy the full world removal set once for every chunk. The durable
`removed_props` map and revision remain unchanged as save authority. The ecology adapter contract passes 39/39 at
`artifacts/citadel-runtime-integration/ecology-section-value-adapter-spatial-removal-20261004-r7/report.json`,
including an exact dynamic removal from one section with other source revisions
unchanged and two chunk snapshots still capturable after a world-wide removal.
The tree adapter contract passes 13/13 at
`artifacts/citadel-runtime-integration/tree-section-value-adapter-spatial-removal-20261004-r3/report.json`,
including stable tree geometry under an unrelated durable removal. These are
synthetic source/recipe contracts, not live harvest acceptance.

The final-source tutorial-free headed Main diagnostic passes with seed
`terrain-section-refresh-proof-20261004-r5`: a current native section candidate
was installed at generation 29, with 49 contributors and a live coordinator
and backend receipt. Report/screenshot:
`artifacts/chunk-owned-rendering/spatial-removal-20261004-r6/playtest-report.json`
and `artifacts/chunk-owned-rendering/spatial-removal-20261004-r6/playtest.png`.
The owned process exited 0 and job membership reached authoritative zero. This
proves production candidate install after this source change, not harvest,
save/reload, traversal, performance, parity, or legacy publisher retirement.
The screenshot still has dense dark canopy/shadow coverage. A second fresh-seed
diagnostic remained pending at `terrain_empty_section_requires_exact_empty_manifest`
until stopped via its owned stop-request channel; the watchdog confirmed
authoritative zero job members. This is an unresolved terrain census blocker
separate from ecology removal. Replaying the known r5 seed later passed its
receipt gate, so retain both observations rather than treating the first
pending attempt as a code regression.

Live harvest/save-reload remains unverified. The existing resource-lifecycle
headed runner currently fails to parse because it calls the coroutine
`wooded_surface_candidates()` without `await`; its normal New Game path also
starts the tutorial town, so it is not an acceptable runner under the standing
tutorial-free playtest constraint. No harvest action was counted as evidence.

The implementation is in the game repository's current worktree slice; the
charter was committed separately as `5f23a60` before production edits and this
evidence update is `2f55c80`. Stage 5 remains partial and the overall migration
remains active.

## Next implementation stage: ordinary generated-structure recipes (4A)

**User-visible outcome:** generated ordinary buildings and structures are
rendered from the same section-owned candidate path as terrain/ecology while
their existing per-cell bodies continue to own collision, interactions, doors,
light, navigation and gameplay state. Keep the current accepted section visible
until the replacement section receives its native install receipt. Mobs/NPCs
remain independently simulated and rendered.

**Current source path:** `StructureSystem` deterministically produces town and
standalone placements through `place_block`, `place_path`, `place_utility`,
and `place_door`; `_record_ordinary_visual_block` records source membership,
cell and block type plus a source revision. Durable generated-block removal is
in `removed_generated_structure_blocks` and feeds the save snapshot. Replay
regenerates ordinary structures from seed and filters those stable removal
keys. `MainChunkTerrain.create_block` creates each live `StaticBody3D`, collider,
special visual children, material/light and interaction metadata. Current
`OrdinaryStructureStaticSectionProvider` discovers the expected records, then
`OrdinaryStructureSectionGeometryAdapter.capture_block` requires the live body
and copies one visible child mesh/material; only three plain opaque block types
are admitted, visual metadata and multisurface content are rejected. The
candidate goes through `StaticSectionSourceRoster`, cross-domain candidate
assembly, the section coordinator and C++ backend. The old body-child visuals
remain published; no ordinary structure visual has been retired.

**First bounded discovery (no production edits until it exits):** enumerate the
actual generated ordinary block type and option combinations from StructureSystem
producers, including roof/fence/window/torch options and interactive furnishings;
map each to the exact `create_block` mesh/material child tree, collision bounds,
transform, render layer, shadow and light behavior, live interaction owner,
save fields and deterministic replay key. Classify unsupported/interactive or
translucent cases explicitly. Identify one representative populated generated
structure fixture that avoids tutorial startup and produces a real Main-scene
section receipt. Compare those boundaries with Minecraft 26.2's copied section
region and layered compile result; retain our smooth terrain mesher and gameplay
authorities.

**Implementation stages and gates:** (1) finish the inventory above and choose
the smallest recipe family with exact parity; (2) add one immutable,
revision/digest-bound ordinary visual recipe authority consumed by both
`create_block` and section capture, with all recipe inputs included in source
identity and no live Node/Resource access in the worker packet; (3) contract-test
populated multi-mesh/special variants, section-boundary ownership, stale source
and owner rejection, explicit empty/tombstone replay and unchanged collision/
interaction/save authority; (4) install a populated ordinary-structure candidate
through the real Main scene and native backend, verify old visuals remain until
receipt acknowledgement, then perform a real visual/replacement/unload-replay
check; (5) remove only the proven ordinary source visual publisher, then run
headed generated-structure traversal and representative performance checks.
Blueprint/landmark packet migration remains a separate 4B stage and cannot be
claimed by 4A. Stage 4 exits only after both 4A and 4B cover their actual
production producers and old-visual handoff.

**Risks/non-goals:** do not synthesize visuals from block type alone where
`create_block` uses metadata or multi-mesh children. Do not drop doors,
furnishings, roof details, translucency, lighting, or collision to satisfy an
opaque candidate. Do not replace seed generation or durable delta ownership,
change RNG order, move NPC simulation into render sections, or retire existing
visuals before native replacement acknowledgement.

**Minecraft source review:** in 26.2, `SectionCompiler.compile` reads a
`RenderSectionRegion` and builds separate `ChunkSectionLayer` output; the
dispatcher owns cancellation and installation. Use that copied-input/layered
transaction boundary, not the block-specific meshing rules. Our immutable
recipe result must describe the exact section contributors and layers while
keeping gameplay colliders and interactions under current scene authorities.

The worktree inventory above is the discovery baseline. No implementation
change for stage 4A has started; the ordinary-provider synthetic fixture proves
only its supported adapter contract and not a populated production building
receipt. The separate blueprint provider also remains incomplete. The migration
is active.

**Stage 4A discovery exit (2026-10-04):** producer review covered town/cabin
walls, roofs, windows, doors and furnishings; ruin/camp blocks and paths; and
trader-stall, torch, spike-trap, campfire and loot-chest output. The source
ledger stores only expected `cell -> blockType`, omitted/failed keys and a
revision. Per-cell visual options (roof role/axis/side/material/edge/chimney,
window/fence/corner accents, torch mount/scale and structure transform offsets)
are currently written to live body metadata but are not part of the producer
value snapshot or its source revision. `create_block` can publish one or many
mesh children; furnishing, door and other interactive nodes also own gameplay
state and collision. Therefore the present three-type allowlist is not a
complete structure recipe path, and merely broadening it would create false
coverage. Start the production cutover with ordinary opaque base-block visual
recipes, with option-bearing and multi-mesh members still pending until their
inputs are captured authoritatively and represented by the same recipe used by
`create_block`.

**Unchanged-baseline navigation check:** `node tools/npc/run-npc-contract-tests.mjs
-TimeMode Both` passed its 88-check aggregate at
`artifacts/npc/reports/contract-both.json` (seed `atlas-1492`, 0 failures).
`node tools/npc/run-all-npc-tests.mjs -TimeMode Both` produced passing contract,
motor, route, repair, door, avoidance, traffic, behavior, interaction, and soak
reports (seed `atlas-1492`). Its streaming-save report had two failures, both
`npc_save_world_signature_unchanged`: the tracked baseline exists (423894 bytes)
but the required generated `artifacts/world-signature/latest/atlas-1492.json`
does not. The run-all registry does not include the world-signature producer, so
this is a missing test prerequisite, not evidence of a save or pathfinding
regression. Preserve it as a pre-existing test/setup blocker unless reproduced
after first generating the prerequisite artifact.

The aggregate then began headed generated-world `go_home_visual` and remained at
startup readiness (`2090/2098` visuals ready, 8 waiting) after 182 seconds with
no gameplay result. It was interrupted before the following real tutorial
playthroughs to honor the standing tutorial-free instruction. Thus the full
aggregate and headed door sequence remain untested, not passed. The active Godot
processes launched by this run were absent after interruption; no final
watchdog/job-membership cleanup report was obtained, so authoritative cleanup
proof is missing. No structure edits had begun and protected NPC/pathfinding
code remains untouched.

**Bounded removal projection follow-up (2026-10-04):** the first implementation
copied the full world removal set for each chunk. That would make chunk capture
cost scale with all previously harvested props, so it was replaced before
acceptance with `ActiveRemovedPropsSnapshot.capture_for_ids`: ecology collects
the unique prop IDs already present in its sealed chunk snapshot and double-
checks only those IDs; a tree capture checks only that tree's stable prop ID.
Unrelated global removal revisions no longer invalidate resident chunk capture,
while owner/seed/revision checks still reject changes during the bounded read.
The save authority remains the existing `removed_props` map and global revision.

Focused evidence on the final source: ecology adapter 40/40 at
`artifacts/citadel-runtime-integration/ecology-section-value-adapter-spatial-removal-20261004-r9/report.json`;
tree adapter 13/13 at
`artifacts/citadel-runtime-integration/tree-section-value-adapter-spatial-removal-20261004-r4/report.json`.
These prove scoped removal capture, exact local projection/tombstone behavior,
unrelated chunk snapshot stability, and tree recipe stability under unrelated
removal. They are synthetic contracts, not gameplay/save-reload acceptance.

The final-source, tutorial-free headed Main diagnostic passes with seed
`terrain-section-refresh-proof-20261004-r5` at
`artifacts/chunk-owned-rendering/spatial-removal-20261004-r7/playtest-report.json`.
It installed candidate generation 50 through the native renderer with a current
coordinator/backend receipt and 49 contributors, then saved the screenshot at
`artifacts/chunk-owned-rendering/spatial-removal-20261004-r7/playtest.png`.
The owned process exited successfully and reported authoritative zero job
members. This proves production candidate installation, not harvest, save/reload,
visual parity, traversal, performance, or retirement of the per-source
publishers. The screenshot retains dense dark foliage/shadow coverage. A second
seed's earlier `terrain_empty_section_requires_exact_empty_manifest` pending
result is still an independent unresolved census blocker; the passing r5 seed
does not clear it. Stage 5 remains partial, and the overall migration remains
active.

## Ordinary source-recipe input increment (2026-10-04)

`StructureSystem._record_ordinary_visual_block` now stores a recursively sealed
per-cell record containing block type, the exact option dictionary passed to
`MainChunkTerrain.create_block`, and a SHA-256 digest. Accepted structure-source
revisions advance when either membership/type or any visual recipe input
changes. The ordinary source census fails closed if an expected member lacks a
valid recipe record. `OrdinaryStructureSectionGeometryAdapter` verifies the
sealed record and digest, includes the digest in its geometry source revision,
and carries the sealed recipe input/digest through the prepared section
partitioner. `create_block` still renders by its existing implementation and
the section adapter still reads one live mesh child; this is a value-input
capture step, not the geometry recipe/renderer cutover.

Final-source contracts pass: producer ledger 16/16 at
`artifacts/citadel-runtime-integration/ordinary-structure-visual-source-revision-20261004-r3/report.json`;
geometry adapter 15/15 at
`artifacts/citadel-runtime-integration/ordinary-section-geometry-adapter-recipe-20261004-r3/report.json`;
section provider 15/15 at
`artifacts/citadel-runtime-integration/ordinary-static-section-provider-recipe-input-20261004-r3/report.json`.
They prove deep read-only recipe inputs, stable revision under identical input,
stale/mutable recipe rejection, digest propagation into prepared section
geometry, boundary/removal census behavior and collision owner retention. These
remain synthetic contracts. Earlier r1/r2 adapter attempts found a missing
fixture `CELL` and unpositioned synthetic member bodies; both stopped under the
owned watchdog with authoritative zero, and the corrected r3 runner passes.

The current-source tutorial-free headed production diagnostic also passes at
`artifacts/chunk-owned-rendering/ordinary-recipe-input-20261004-r1/playtest-report.json`.
It installed section `(0,0,0)` through the native backend at generation 9 with a
current receipt and 49 contributors in 41.2 seconds of game runtime. This
selected section's ordinary-structure coverage is explicitly `empty` with
source count 0, so it proves unrelated production integration stability, not a
populated generated building. The watchdog is
`artifacts/node-tools/process-runs/godot-fmqv5L/watchdog.json` (exit 0,
`cleanupPassed=true`, authoritative zero, no job members). The screenshot is
`artifacts/chunk-owned-rendering/ordinary-recipe-input-20261004-r1/playtest.png`;
it still shows heavy dark canopy/shadow coverage and a large held item. No
ordinary per-cell visuals have been retired; collision, interactions, doors,
lights, navigation, save and replay behavior remain under their current owners.

The NPC contract baseline after this source edit passes 88/88 at
`artifacts/npc/reports/contract-both.json` for seed `atlas-1492`. The broader
pre-edit baseline and its missing world-signature prerequisite plus the
tutorial-free interruption of the aggregate's headed door test are recorded in
the stage 4A discovery note above. Stage 4A remains partial: build an actual
shared block visual recipe consumed by both `create_block` and section capture,
capture a populated generated-structure section through the real renderer, and
prove accepted replacement/unload/replay before legacy visual retirement.

## Production candidate and exact-empty increment (2026-10-04)

The ordinary base-block recipe now admits the production `ShaderMaterial` used
by `woodBlock`, `stoneBlock`, and `cobblestonePath`, but only for the known
opaque `resources/visual/building_material.gdshader`. Material identity includes
the shader source and its sorted uniform values; a uniform change changes the
prepared source revision. Unknown shaders, alpha output/blend modes, unsupported
options, and interactive or multi-mesh structure members still remain pending.
The same recipe resolves the source mesh/material/transform for `create_block`
and section capture. The live body remains the collision/gameplay owner.

An exact fluid-free terrain capture that meshes to no surfaces now produces an
explicit empty source row with source revision, section and logical owner. The
whole-section assembler requires each census member to supply either geometry
or exactly one sealed explicit-empty row; a missing member still returns
retryable pending. The section snapshot includes the empty contributor in its
manifest with no batch keys or ranges. This follows Minecraft 26.2's section
compiler/dispatcher behavior where an empty compiled result is still committed
as the next current section result. It does not substitute Minecraft's block
mesher for our smooth SDF/Transvoxel terrain path.

Evidence:

- Building shader adapter contract: 16/16 at
  `artifacts/citadel-runtime-integration/ordinary-section-geometry-adapter-building-shader-20261004-r2/report.json`.
- Cross-provider exact-empty candidate contract: 8/8 at
  `artifacts/citadel-runtime-integration/whole-section-candidate-assembler-empty-manifest-20261004-r1/report.json`. It proves manifest insertion and that omission remains retryable; it is synthetic evidence.
- Tutorial-free headed Main run:
  `node tools/run-playtest.mjs --only production_section_candidate_diagnostic --seed terrain-section-refresh-proof-20261004-r6 --visible true --timeoutSeconds 240 --watchdogSeconds 240 --reportPath artifacts/chunk-owned-rendering/ordinary-recipe-renderer-proof-20261004-r11/playtest-report.json --progressPath artifacts/chunk-owned-rendering/ordinary-recipe-renderer-proof-20261004-r11/progress.txt --screenshotPath artifacts/chunk-owned-rendering/ordinary-recipe-renderer-proof-20261004-r11/playtest.png -- -SkipTutorial`.
  The real Main provider census included the ordinary fixture, the candidate
  installed through the native renderer at generation 42 with a current native
  receipt and the fixture part in its manifest, and the terrain edit-refresh
  installed generation 53 with a different manifest digest and current receipt.
  The edit was also present in the durable terrain save delta. The screenshot is
  the headed forest view at `artifacts/chunk-owned-rendering/ordinary-recipe-renderer-proof-20261004-r11/playtest.png`.

This is not the Stage 4A exit. The fixture is one diagnostic base block, not a
populated generated building; it does not prove old-visual retirement, complete
decoration/material parity, collision/contact parity, unload/replay, ordinary
save/reload, traversal, or runtime performance. Ordinary structure rendering
still uses the per-source publisher alongside the section candidate. The
selected terrain section's exact empty mesh had no terrain batch; a separately
revisioned save edit then replaced that result. The run exposed a separate slow
exact-fluid readiness path on cold startup but completed successfully. Preserve
the complete existing structure producer until all member recipes and the old
visual handoff are proven. Continue with real generated-structure manifest
capture, replacement/unload-replay, then headed traversal and performance before
retiring its publisher.

## Synchronous census profile and ecology optimization (2026-10-04)

The live candidate path is integrated but its source census is not yet suitable
for continuous traversal. A 75-second normal-runtime diagnostic at seed
`atlas-26817380` recorded `whole_section_candidate_phase_census` p95 829.22 ms,
p99 861.152 ms, max 1453.357 ms (112 samples); the full candidate capture p95
was 829.428 ms and max 1453.694 ms. The run failed its cadence/performance
thresholds, so it is diagnostic evidence, not acceptance. Its report is
`artifacts/performance/chunk-owned-rendering-phase-profile-20261004-r2.json`.

A tutorial-free, headed Main diagnostic using the previously proven seed
`terrain-section-refresh-proof-20261004-r6` completed candidate installation
through the native renderer. During a separate demanded section's admission,
the provider profile attributed 104,916 µs of a 104,988 µs census to
`ecology_and_static_props`, which remained pending with
`ecology_tree_queue_geometry_not_committed`. Blueprint capture took 37 µs.
Report and screenshot:
`artifacts/chunk-owned-rendering/candidate-provider-profile-known-seed-20261004-r2/playtest-report.json`
and `playtest.png` beside it. This identifies synchronous ecology preparation
as the immediate census bottleneck; it does not prove a complete visual
replacement or acceptable runtime performance.

The next bounded implementation is a lightweight ecology membership/revision
census using producer-owned bounds and explicit coverage proofs. Census must
not prepare or hash meshes, partition tree geometry, or fabricate empty
coverage. Preserve the live owner/resource checks on the main thread and leave
geometry preparation in the contribution phase until a detached value packet is
proven. Validate with the ecology provider/adapter contracts, the known-seed
headed production candidate-install diagnostic, then a new representative
performance observation. A separate random-seed run ended before gameplay with
`final_terrain_expansion_timeout`; that is an unresolved startup-readiness
failure and is not attributed to this census change:
`artifacts/performance/chunk-owned-rendering-provider-profile-20261004-r3.json`.

Stages remain 0 complete, 1–5 partial, and 6 not started. The short performance
profile is a failed diagnostic, not a Stage 6 performance gate; construction
and ecology still publish through their legacy paths alongside section
candidates. The appropriate Minecraft 26.2 precedent remains the captured
section input and cancellation-safe compile/upload/swap lifecycle in
`SectionCompiler`, `RenderSectionRegion`, and `SectionRenderDispatcher`; its
block mesher is not applicable to our smooth SDF/Transvoxel terrain.

## Section-owner mismatch follow-up (2026-10-04)

An instrumented headed candidate diagnostic reported a surface-detail
`sourcePartId` whose census section was `(-1, 1, 0)` while the shared
partitioner assigned its transformed mesh bounds to `(-1, 0, 0)`. The census
had been recovering mesh bounds by inverse/forward transforming a sealed AABB;
it now validates the sealed bounds against the live source mesh, then uses the
source mesh AABB center transformed by the same `sourceToWorld * instance`
composition as the partitioner. This avoids a round trip at section boundaries.

The focused ecology contract passed 45 checks, including an off-center,
rotated and scaled mesh on a section boundary:
`artifacts/citadel-runtime-integration/ecology-section-value-adapter-detail-owner-parity-20261004-r22/report.json`.
The headed, tutorial-free known-seed diagnostic then installed the demanded
`(-1, 1, 0)` candidate with a current native renderer receipt and 71 instances;
report and screenshot are under
`artifacts/chunk-owned-rendering/candidate-membership-census-known-seed-20261004-r4/`.
This is real production candidate installation evidence, not traversal,
replacement/unload-replay, full producer parity, or performance acceptance.
Its screenshot shows a live forest view. In that admission, ecology census
still took 40,029 µs (total census 40,087 µs) while other background demands
were pending, so the membership fix does not resolve the performance gate.
The exact previously mismatched pebble ID was not retained in the final receipt;
the synthetic owner-parity assertion and successful section installation do
not independently prove its final expected-section membership. Add a focused
live assertion for that source identity while continuing to reduce census cost.

The follow-up removed a redundant candidate-digest pass: production capture
already verifies every candidate digest before the synchronous ownership loop.
The provider still rejects a deliberately corrupted candidate before census;
the ecology contract passed 46 checks at
`artifacts/citadel-runtime-integration/ecology-section-value-adapter-candidate-digest-once-20261004-r23/report.json`.
The same known-seed headed diagnostic installed section `(-1, 1, 0)` with a
current native receipt and 71 instances. Its sampled ecology provider call was
15,715 µs (15,787 µs total census), pending on unrelated tree queue geometry.
This is one non-controlled sample on a different chunk from the earlier 40 ms
sample, so it cannot establish a performance improvement or pass the runtime
gate. Report and screenshot are under
`artifacts/chunk-owned-rendering/candidate-membership-census-known-seed-20261004-r5/`.

### Exact ecology source through section installation (2026-10-04)

The known-seed headed diagnostic now follows the exact boundary source through
the real section path. Its first version treated a retryable census response as
final and failed while the terrain provider was still building the section's
exact fluid proof. The diagnostic now keeps retrying that census, then requests
the exact owner section through the normal visible-demand scheduler and verifies
the installed candidate manifest plus current native receipt.

Run command:
`node tools/run-playtest.mjs --only production_section_candidate_diagnostic --seed terrain-section-refresh-proof-20261004-r6 --visible true --timeoutSeconds 240 --watchdogSeconds 240 --reportPath artifacts/citadel-runtime-integration/exact-ecology-source-section-contribution-20261004-r5/playtest-report.json --progressPath artifacts/citadel-runtime-integration/exact-ecology-source-section-contribution-20261004-r5/playtest-progress.txt --screenshotPath artifacts/citadel-runtime-integration/exact-ecology-source-section-contribution-20261004-r5/screenshot.png -- -SkipTutorial`.

It passed in 117.9 seconds. The exact source
`terrain-section-refresh-proof-20261004-r6:detail:-1,0:pebble:9:surface:0`
was in the census for `(-1, 0, 0)`, partitioned into that section, and present
in the installed candidate manifest; its coordinator source revision and
native receipt were current for the section. The same run passed terrain edit
replacement at `(-1, 1, 0)` with a changed manifest digest and durable edit
delta. The report is a headed diagnostic, not a traversal or
visual/performance gate. It needed 134 census attempts before the exact fluid
proof became ready; this long retry period is an outstanding cold-readiness and
performance observation, not a reason to accept missing data or treat it as an
empty success.

This diagnostic follows the lifecycle lesson from Minecraft 26.2's
`RenderSectionRegion` and `SectionRenderDispatcher`: compile one bounded section
from a captured neighborhood, cancel stale work, and switch the current section
only after the replacement has been acknowledged. It does not port Minecraft's
block mesher or alter the game-wide migration exit criteria.

### Stage 5 tree resource-freshness follow-up plan (2026-10-04)

The post-install audit found a source freshness hole to close before broader
ecology cutover. `TreePublicationQueue` retains frozen member dictionaries whose
Mesh and Material values are still mutable Resources. Ecology census revisions
currently bind recipe signature, LOD, body transform and instance identity, but
not the resource-aware revision already computed by `TreeSectionValueAdapter`
from mesh/material fingerprints and instance values. The ecology contribution
wrapper also currently recomputes its own tree revision without incorporating
`captured.sourceRevision`. A resource mutation can therefore change prepared
geometry while leaving the source roster revision unchanged.

Before editing production code, trace and preserve this boundary: the queue owns
committed recipe/value members; census owns section membership and revision;
the tree adapter fingerprints and partitions geometry; the coordinator rechecks
the census before native installation. The implementation should establish one
producer-owned section-value revision over member mesh/material content and
instance values, use that same identity for census and contribution, and make a
changed retained resource invalidate or reject pending candidate work. Do not
move mesh preparation into the per-section census or weaken the existing
retain-old-until-receipt rule.

Acceptance for this slice is a focused contract that mutates a retained Mesh
and Material after publication and again between contribution capture and
candidate acceptance; each change must either advance the current roster
revision or reject stale work while preserving the previous installed section.
Then rerun the tree/ecology provider contracts, the known-seed headed source
install diagnostic, and a representative performance observation to check the
cost of the new freshness boundary.

#### 2026-10-04 implementation evidence

`TreeSectionValueAdapter.raw_member_content_revision` now hashes the queue's
sealed role/member values together with current Mesh and Material fingerprints,
local/member transforms, colors/custom data, element count, visibility/fade and
shadow policy. Tree contribution preparation binds its source revision to this
digest. A shared helper now builds the complete tree producer source revision
from world/seed/source identity, recipe and pipeline revisions, LOD, owner cell,
body transform, and the raw resource/value digest. Ecology census computes this
same producer revision without encoding instance attributes or partitioning
geometry; the contribution wrapper includes the exact `captured.sourceRevision`
plus the same candidate/body identity. This closes the specific aliasing gap
where a frozen Dictionary still referenced a mutated Godot Resource, including
the pipeline and ownership inputs used by contribution preparation.

The tree value contract passed all checks, including independent mutation of a
retained BoxMesh, StandardMaterial3D color, and its ImageTexture pixels, with
exact census/contribution revision equality after each mutation. The tree
adapter's material fingerprint now covers supported BaseMaterial3D properties
and Texture2D content, matching the ecology adapter's material-content policy.
The ecology provider contract also passed. A
headed synthetic native-install fixture staged a replacement, mutated its
shared mesh, observed a new source revision, cancelled the stale candidate at
source-census revalidation, and retained generation 2 and its prior native
root. The fixture then restored the resource and verified unload/replay. These
are separate proofs: the coordinator fixture does not wire production ecology
into the native candidate path.
Final rerun reports: `artifacts/citadel-runtime-integration/tree-section-value-adapter-resource-freshness-20261004-r6/report.json`, `artifacts/citadel-runtime-integration/ecology-section-value-adapter-resource-freshness-20261004-r4/report.json`, and `artifacts/citadel-runtime-integration/whole-section-candidate-native-install-resource-freshness-20261004-r5/report.json`.

The attempted normal-runtime sprint performance runner remained at
`main_menu_waiting_for_gameplay_ready` through 2,880 frames; its default New
Game path enters the tutorial, so it was stopped to honor the tutorial-free
playtest constraint. The suitable lightweight runner,
`run-visible-world-fast-turn-sprint.mjs -DiagnosticReplaySeed
freshforest20261004 -SkipTutorial -ForceDaytime -ForceClearWeather`, then ran
live on that fresh seed. Its first outdoor view and rapid-turn checkpoint were
ready at 2,366/2,366 candidates. After the attempted sprint, the view was
pending at 2,410/2,417: seven of 324 tree/foliage candidates lacked
representations while terrain, structures, props, and wildlife were
represented. This was not a native section receipt for those exact trees.

The live player moved 42.67 m of the 45 m target before the shared navigator
stopped on `unreachable_static` / `blocked_capsule_probe`. The report identifies
a generated prop collider at `(45.9, 14.84398, 17.55)` in cell `(34, 13)`; the
route authority's collision repair attempted three alternate routes before
terminating. This is a traversal-fixture failure, not evidence that resource
revision hashing caused the stop. The headed diagnostic recorded 1,363 frame
samples with p50 34.6 ms, p95 43.0 ms, p99 54.3 ms and max 79.1 ms. Candidate
census for ecology/static props had only three samples (p50 0.93 ms, max 17.67
ms), insufficient to certify the new hashing cost or representative
performance. Report and screenshots are under
`artifacts/citadel-runtime-integration/tree-resource-freshness-fast-turn-20261004-r1/`.

Minecraft 26.2 `SectionCopy` copies the section's `PalettedContainer` before
`SectionCompiler` reads the bounded 3x3x3 region; its `SectionRenderDispatcher`
cancels stale work and only acknowledges a replacement after compile/upload
work succeeds. That reference exposed why a read-only dictionary around mutable
Godot resource aliases was not a stable section input. Our current repair
revalidates resource content rather than copying complete immutable GPU inputs;
the census cost still needs a representative profile. Stage 5 remains partial:
complete static ecology family coverage, exact tree contribution receipt,
harvest/reload, cross-section replay, legacy visual retirement, and live
traversal still have separate exit gates.

### Cold fluid-proof queue diagnostic plan (2026-10-04)

The exact-source run's 134 census attempts only show that the probe had not yet
published a current proof. Its report does not distinguish FIFO backlog, slow
cell progress, or repeated stale cancellation from volume/fluid revision
changes. Before changing queue policy, add bounded samples to the same headed
diagnostic: target section's queue index/length, probe phase, `cellsProcessed`
against payload size, start/current volume and fluid revisions, and stale or
completion outcome. Keep sampling intervals coarse and reports bounded. One
known-seed tutorial-free replay should decide whether the next change belongs in
queue priority/fairness, per-frame work budget, or revision invalidation.

Minecraft 26.2's `SectionTaskDynamicQueue` orders available work by camera
distance while preserving initial compiles with a small recompile quota;
`SectionRenderDispatcher` requeues work when resources are unavailable and
cancels superseded work. This is a scheduling reference only. Do not change the
terrain proof queue until its measured backlog/progress/staleness identifies
which property limits the exact demanded section.

The sparse-sample replay passed at the same seed in 22.3 seconds of diagnostic
time with seven census attempts. At first observation, the target probe was
already the single queue item at index zero; its payload was 3,888 cells and
its captured volume/fluid revisions (1/0) still matched current revisions. The
next sample found a current proof, with no stale or cancelled state. This replay
does not explain the earlier 134-attempt/117.9-second outlier; it does show that
the exact probe can reach the queue head without backlog or revision drift.
Retain the outlier as unresolved performance evidence and do not change queue
priority based on it alone. Report and screenshot are under
`artifacts/citadel-runtime-integration/fluid-proof-queue-telemetry-20261004-r1/`.

### Tree publication backlog and ordinary-structure recipe follow-up (2026-10-04)

The tutorial-free fast-turn runner now preserves up to 16 pending candidates
and attaches bounded producer/queue diagnostics to the terminal streamed-area
checkpoint. This closes the earlier six-of-seven diagnostic truncation. The
same-seed replay at
`artifacts/citadel-runtime-integration/tree-resource-freshness-fast-turn-20261004-r2/report.json`
reached its outdoor control and rapid-turn checks with the full 2,366-candidate
view ready. After sprint it stopped at 42.67 m on the existing
`blocked_capsule_probe` route repair failure; the later 2,417-candidate view
had seven pending tree candidates. Each had a live physical body and prepared
recipe. The seven exact tasks were in the completed-publication queue at
`renderStage=root` after roughly 40 seconds; the queue had no active workers,
239 completed recipes, one staged task whose identity was not captured, and 87
published recipes. This supports a publication-backlog diagnosis for these
candidates rather than missing source capture, but does not prove queue policy
is the only cause.

Minecraft 26.2's `SectionCompiler` builds every block/fluid render layer for
one section in one compile result; `SectionTaskDynamicQueue` orders that section
work by camera distance with a small recompile quota. Our tree visuals still
spend a separate bounded publication slice on each tree's root, bole, branches,
foliage and commit. That architectural mismatch explains why simply raising a
tree queue cap is not the right migration step: section ownership should batch
the complete static candidate and let section-level scheduling make progress.
Do not copy the Minecraft block mesher or its exact quota; smooth terrain and
our procedural ecology retain their own authorities.

The ordinary generated-base-block bridge now calls the shared visual recipe
resolver during `MainChunkTerrain.create_block` and attaches the exact mesh,
material and local transform used by section capture, while retaining the
original gameplay body, collider and interaction metadata. Focused ordinary
source, adapter, provider and whole-section assembler contracts passed
16/16, 16/16, 15/15 and 8/8 respectively. The headed synthetic native-install
fixture passed 9/9. A post-edit headed Main-scene run reached a ready initial
view and rapid-turn checkpoint; screenshot comparison against the pre-edit
same-seed run shows the same initial presentation. Its traversal still stopped
at 42.67 m on `blocked_capsule_probe`, and after sprint eight tree candidates
were pending. This confirms startup visual continuity, not ordinary-building
recipe/section parity: no generated layout was assembled as a full section
candidate, production receipt retirement remains unwired, and unload/replay,
save, live traversal and performance gates remain open. Artifacts are under
`artifacts/citadel-runtime-integration/ordinary-structure-visual-source-section-recipe-20261004-r1/`,
`ordinary-section-geometry-adapter-recipe-20261004-r6/`,
`ordinary-static-section-provider-recipe-input-20261004-r9/`,
`whole-section-candidate-assembler-migration-20261004-r1/`,
`whole-section-candidate-native-install-migration-20261004-r1/`, and
`ordinary-recipe-live-main-fast-turn-20261004-r1/`.

### Explicit empty coverage and phase timing follow-up (2026-10-04)

An all-air terrain section now enters the shared whole-section candidate as an
explicit empty contributor bound to its source-part revision, logical owner and
section. The assembler requires exactly one geometry input or explicit empty
record for every census member; missing output remains retryable. This avoids
mistaking an omitted producer result for a valid empty section. The focused
terrain contribution contract passed 9/9 at
`artifacts/citadel-runtime-integration/terrain-section-contribution-empty-minecraft-review-20261004-r2/report.json`;
the shared candidate assembler contract passed 8/8 at
`artifacts/citadel-runtime-integration/whole-section-candidate-assembler-empty-minecraft-review-20261004-r2/report.json`.

The coordinator now records census, per-provider capture, contribution,
assembly, and submit/revalidation durations, and preserves exact requested and
owned section/source-part identity in pending-demand diagnostics. The visible
demand contract passed 25/25 at
`artifacts/citadel-runtime-integration/visible-section-demand-driver-minecraft-review-20261004-r2/report.json`.
A headed fixture passed 9/9 at
`artifacts/citadel-runtime-integration/whole-section-candidate-native-install-minecraft-review-20261004-r2/report.json`,
proving the assembled fixture candidate reaches the native section renderer;
it still does not prove generated-world producer parity or visible gameplay.
The ecology realized-prop capture contract passed its production-authority
and RNG replay assertions at
`artifacts/citadel-runtime-integration/ecology-realized-prop-capture-minecraft-review-20261004-r1/report.json`.
Its fixture now derives ecology revisions from the production authority rather
than duplicating the revision string.

These changes do not resolve the live sprint's two distinct symptoms: tree
visual publication can remain backlogged after recipe preparation, while the
shared player navigator can independently stop at a generated-prop capsule
obstacle. Minecraft's section compiler supports replacing per-tree work with
complete section jobs, but the current native-install proof uses fixture
providers; production receipt-based visual retirement and successful live
traversal/performance evidence remain open. Game commits: `e17937e4`,
`7d508d2c`, and `75045fe4`.

## Next implementation charter — admit prepared trees before legacy visual commit

**Outcome:** let a complete normal section candidate consume the exact tree
recipe and renderer-member values already prepared by `TreePublicationQueue`,
without waiting for the per-tree `GeneratedTreeVisual` attachment/commit. The
current installed section remains intact until its replacement has a live
native receipt. Tree `StaticBody3D`, trunk collision, prop ID, harvesting,
interactions, navigation blocking, deterministic ecology and save/removal
authority remain unchanged; mobs and NPCs remain outside static sections.

**Baseline:** game branch `codex/chunk-owned-world-rendering-migration` at
`75045fe4`, with generated `.import` churn only. A tutorial-free fast-turn run
on seed `freshforest20261004` reached 2,366/2,366 initial and rapid-turn view
readiness, then left seven physically present trees with prepared recipes in
`TreePublicationQueue.completed` at `renderStage=root`; queue state was 239
completed, 87 published and no active workers. Report:
`artifacts/citadel-runtime-integration/tree-resource-freshness-fast-turn-20261004-r2/report.json`.
The route separately stopped at 42.67m on a generated-prop capsule collision;
that is not evidence about section admission. Existing tree capture, queue and
ecology contracts pass, but the prior native renderer fixture uses synthetic
providers.

**Source path and change boundary:** deterministic ecology candidates originate
in `MainPlaytestTools.make_tree` and are recorded with stable source/prop IDs,
runtime specification, bounds and revision. `_publish_tree_body` owns the live
gameplay body and enqueues its immutable request in `TreePublicationQueue`.
The queue's recipe worker produces canonical recipe values; staged root/bole/
distal/foliage work builds renderer resources and `_append_section_value_member`
captures immutable mesh/material/transform/instance attributes. Today,
`commit_published_visual` alone calls `remember_published_lod`; the ecology
provider then enumerates `published_lod_records`, and both census and
contribution reject trees without committed per-tree visuals. Separate a
queue-owned, sealed **prepared tree artifact** from the published-LOD record at
the already-existing full-member boundary. Bind it to world/seed, stable source
and prop IDs, exact body instance and current transform, normalized request,
recipe signature, LOD tier, resource-aware member digest, removed-prop snapshot
and producer generation. Census and contribution must consume the same artifact
revision; stale body, request, resource, LOD, removal or world identity stays
pending/rejected. Never infer empty from absent queue work.

**Stages and proof:** (1) focused contract proves a complete artifact exists
before per-tree visual commit and is immutable/revision-bound; absent,
incomplete, stale and removed artifacts fail closed. (2) the normal ecology
provider forms a complete section candidate from prepared tree values mixed
with current terrain/detail/prop contributors, and a headed Main-scene run
installs it through the native renderer with exact manifest membership and a
live receipt while the old per-tree visuals and gameplay bodies remain valid.
(3) cancellation, changed resource/revision, replacement owner and unload/replay
prove staged data is discarded while the old section stays visible. Only after
those pass should a later slice wire provider receipt acknowledgement to
receipt-gated old tree visual retirement and test one-time removal while the
collider, harvest ID and save delta remain live. Each gate is reported
separately; no stage advances on a unit/fixture result alone.

**Minecraft 26.2 check:** `SectionCopy` snapshots section states before worker
compilation; `RenderSectionRegion` exposes its captured neighborhood;
`SectionCompiler` emits one section result across render layers; and
`SectionRenderDispatcher` keeps the prior mesh through upload acknowledgement.
That validates separating prepared input from the old visual commit and
switching only on receipt. This slice only removes a publication-order
dependency: per-tree root/bole/branch/foliage construction is still staged per
tree. A later section-level recipe compiler/batcher must address that remaining
cost; do not copy Minecraft's block mesher or claim this bridge has eliminated
per-tree geometry work.

### Prepared tree admission implementation (2026-10-04)

Game commit `bb32bbf0` adds a sealed queue-owned prepared-tree artifact at the
existing full-member boundary. Ecology census and contribution can now consume
that artifact before a per-tree visual commit. After the shared coordinator
installs a complete candidate and verifies its live native receipt, providers
receive source acknowledgements. A tree spanning multiple sections retains its
legacy visual until every owning section has a still-current receipt; its
accepted queue values then remain available for later revisions. Gameplay body,
collision, prop identity and interaction authority remain in the existing tree
path. The production queue enables this mode only when both the coordinator and
ecology provider are available.

Focused evidence passed:

- Prepared tree capture, resource-aware revision, multi-section receipt
  accumulation, receipt-gated legacy visual retirement, and reuse of accepted
  values: 19/19 checks at
  `artifacts/citadel-runtime-integration/tree-section-value-adapter-minecraft-cutover-20261004-r6/report.json`.
- Ecology producer/value contract: 46/46 checks at
  `artifacts/citadel-runtime-integration/ecology-section-value-adapter-minecraft-cutover-20261004-r3/report.json`.
- Ordinary structure provider: 15/15 checks at
  `artifacts/citadel-runtime-integration/ordinary-static-section-provider-minecraft-cutover-20261004-r1/report.json`.
- Whole-section native install fixture: 9/9 checks at
  `artifacts/citadel-runtime-integration/whole-section-candidate-native-install-minecraft-cutover-20261004-r1/report.json`.
- Queue contract passed with owned-process cleanup evidence at
  `artifacts/node-tools/process-runs/godot-2tCsPg/watchdog.json`.

The tutorial-free headed production gate did **not** reach gameplay, so it does
not prove tree candidates install through the real provider roster or validate
visual traversal. Command:
`node tools/run-visible-world-fast-turn-sprint.mjs --skip-tutorial true --diagnostic-replay-seed sectioncutover20261004a --timeout-seconds 360`.
It exited with `initial_region_readiness_timeout` after about 201 seconds; the
watchdog exited normally and proved owned-process cleanup at
`artifacts/node-tools/process-runs/godot-QEQHdQ/watchdog.json`. The captured
startup state had physical terrain/structure readiness, but only 15/16 terrain
chunks and 360/369 required terrain mesh blocks were ready (9 remained pending
with `native_terrain_visual_coverage_pending`). The structure visual gate also
had one missing recipe in `town:0,0` at cell `(1,23,24)` despite 167 other
renderable candidates being installed. These are unresolved startup blockers;
the run produced no traversal checkpoint. Do not classify the failed live gate
as caused by the tree commit without a baseline comparison.

The production receipt path is implemented but still needs an actual Main-scene
native section receipt containing a prepared tree, including a tree spanning
multiple sections, followed by visual/traversal and runtime performance checks.
Stage 2 and its live acceptance gate remain open; this is an implementation
checkpoint, not migration completion. Minecraft 26.2's retained-until-uploaded
section mesh still supports the same receipt boundary, while its cube block
mesher remains inapplicable to our smooth terrain and procedural tree geometry.

### Section-local tree census and headed backpressure follow-up (2026-10-04)

Local source reinspection of Minecraft 26.2 confirmed that `SectionCompiler`
works from a copied section neighborhood and that `SectionRenderDispatcher`
keeps the prior installed mesh until replacement uploads finish. The relevant
lesson for the current blocker is section-local admission: unrelated tree
geometry elsewhere in the same streamed chunk must not prevent a section from
being compiled. The ecology provider now checks a tree's authoritative
transformed bounds against the requested render sections before requiring its
prepared tree artifact. Intersecting trees still require the current sealed
artifact and exact per-section contribution; missing intersecting geometry
remains pending.

The focused ecology/provider contract passes 48/48 at
`artifacts/citadel-runtime-integration/ecology-section-value-adapter-locality-minecraft-debug-20261004-r27/report.json`.
It proves an uncommitted tree whose bounds do not intersect the requested
section does not block that section census, while the same missing artifact
still blocks when it intersects. This synthetic contract does not prove a
production native installation or visible-gameplay improvement. The focused
fix is game commit `57297787` (`fix: scope ecology readiness to requested
sections`).

A fresh headed tutorial-free replay on seed
`sectioncutover20261004a` still timed out after 240 seconds at
`Preparing 360° view · 2693/2700 visuals ready · 7 waiting`, with zero
production section candidate jobs admitted. Its persisted diagnostics show the
ordinary-structure provider's resumable work did **not** restart from revision
drift: sampled jobs remained current and advanced through source-cell scanning,
validation, and geometry capture. At the last sample, 171 section jobs were
retained; sampled jobs each described 236 candidates and had advanced to
candidate indices 20–160 under the 32-member-per-turn limit. The last admission
status varied among ordinary capture budgets, terrain preparation and ecology
source coverage. This points to aggregate provider throughput/backlog rather
than a proven single stale-revision loop. The watchdog timed out but confirmed
authoritative zero process membership (`cleanupUnresolved=false`) at
`artifacts/node-tools/process-runs/godot-b3SZ3T/watchdog.json`.

Minecraft's section scheduler reinforces that content preparation and
installation should be admitted as complete section jobs with local immutable
inputs; repeatedly scanning overlapping per-section source sets is a likely
cost center here. Profile and redesign ordinary-source census sharing and
section job scheduling next. Do not claim the tree-bound filter fixed the live
gate, raise publication caps blindly, or advance Stage 5. The loading-screen
readiness gate must remain pending until exact native receipts are installed;
the headed diagnostic watchdog is not a production readiness timeout.

### Demand retry cadence investigation (2026-10-04)

The live provider jobs advanced across repeated calls, while the normal runtime
calls `advance_visible_section_candidate_demands(1)` at most once per
publication frame. `WorldStaticSectionCoordinator` sets
`VISIBLE_SECTION_DEMAND_RETRY_FRAMES` to 30 after any retryable source-census
pending result. This leaves a specific, falsifiable throughput hypothesis:
each section with a resumable provider job is deliberately parked for up to 30
frames before it can take another bounded slice, independent of the job's
progress or remaining publication-frame budget.

Minecraft 26.2's `SectionTaskDynamicQueue.poll(cameraPos)` instead chooses the
nearest eligible initial compile/recompile each time the dispatcher requests
work, reserving only a two-task quota for near recompiles. `SectionRenderDispatcher`
requeues a task when a buffer is unavailable and submits completed section
work through its executor; it does not add a fixed frame-based retry delay to
an incremental section task. This is a scheduling comparison, not a claim that
Minecraft's worker/resource policy can be copied directly.

This cooldown is an observed section-admission policy gap, but it is **not yet
the cause of the visible-world 2693/2700 gate**. The persisted fast-turn
snapshot shows those seven waiters are all tree representations with missing
receipts; a separate `sectionAdmission` field reports one section census
pending on an uncommitted tree. That section/provider result may contribute to
publication latency, but the current trace does not bind it to each of the
seven readiness waiters. The same snapshot shows 193 retained ordinary-section
jobs, confirming independent provider work is also accumulating. Keep these
two gates distinct in follow-up evidence.

Minecraft 26.2 offers a closer analogy for the ordinary-source repetition:
`RenderRegionCache` reuses `SectionCopy` values for adjacent
`RenderSectionRegion` requests during one extraction batch, while each
`SectionCompiler` still produces a separate owned result for its target
section. Our ordinary provider currently creates a distinct resumable source
capture per target, and vertical siblings at the same XZ repeat its 2D source
discovery. A revision-bound shared source-window snapshot, then exact
section-local contribution slices, is the appropriate architectural direction;
persistent reuse must validate world/source/member revisions and retire when
its section jobs no longer need it.

Before changing production policy, extend the visible-section demand contract
with a provider that returns pending once and becomes complete. Record the
baseline retry frame and demonstrate whether an otherwise-ready next-frame
attempt is skipped because of the 30-frame cooldown. If confirmed, remove the
fixed delay while retaining the existing one-attempt-per-publication-frame
runtime bound, nearest-section selection, retryable demand ownership and
installed-revision priority. Then run the focused contract and one tutorial-free
headed startup/traversal replay with queue/provider timing; keep Stage 5 open
unless current native receipts and visual traversal pass. Do not add parallel
worker concurrency or enlarge per-frame work as part of this isolated test.

For the exact seven-tree plateau, first capture those candidate IDs over a
short stable interval and record their queue container, enqueue sequence, age,
render stage, prepared section artifact and native receipt status. A completed
tree task still at `root` should advance to `bole` or fail when selected; an
unchanged stage across samples therefore distinguishes selector starvation
from a section-provider census delay without changing queue policy. The live
trace already identifies one candidate task at `root` for 136.7 seconds, but
has only aggregate selection counts, so it does not prove that this task was
starved.

### Exact tree-queue blocker replay (2026-10-04)

The bounded tutorial-free replay was repeated with lightweight blocker
sampling, without per-tree recipe hashing in the checkpoint path:

```text
node tools/run-visible-world-fast-turn-sprint.mjs --skip-tutorial true --diagnostic-replay-seed sectioncutover20261004b --timeout-seconds 150 --reportpath artifacts/visible-world/tree-queue-debug-20261004-r3/report.json --progresspath artifacts/visible-world/tree-queue-debug-20261004-r3/progress.txt --screenshotdir artifacts/visible-world/tree-queue-debug-20261004-r3/screenshots
```

The watchdog reached its 150-second diagnostic limit with authoritative zero
owned-process membership, but the runner did not produce a report. Thus this
is blocker evidence only; it is not a gameplay acceptance result. At frame
2280 the initial-region readiness census had 32/35 visuals, including 20/23
tree visuals, and remained pending. The section admission reason was
`ecology_tree_queue_geometry_not_committed`.

The exact lightweight samples in
`artifacts/visible-world/tree-queue-debug-20261004-r3/progress.txt.tree-queue.jsonl`
show the blocker is an old per-tree build still in the tree queue's completed
backlog, before a prepared section artifact or section receipt exists. For
candidate `sectioncutover20261004b:10,14:19`, enqueue sequence 46, the
`renderStage` remained `root` in the `completed` container from frame 2040
(age 34.7 s) through frame 2280 (age 49.6 s). Other requested trees likewise
appeared at `root` or `bole` with no prepared section artifact. Across later
samples the current blocked candidate changed, while the visual census still
had three pending trees. This confirms publication queue latency/backlog in
the old per-tree path as a direct section-admission blocker; it does not yet
show which of the three initial-region waiters were identical to each sampled
blocker or prove that selector starvation alone caused the full startup delay.

This result makes the architecture gap clearer than the previous
stale-snapshot hypothesis. The ecology provider requires completed per-tree
render geometry before it can admit a whole-section candidate. In Minecraft
26.2, `RenderSectionRegion` supplies copied neighboring section values to
`SectionCompiler`, which produces all applicable render layers for one
section; `SectionTaskDynamicQueue.poll` selects eligible section tasks by
distance and uses a small recompile quota. The relevant design lesson is to
make tree/static input part of the section's immutable compile input and
section-owned result, rather than waiting for a separate per-tree visual
commit. Keep tree gameplay bodies, collision and deterministic recipe
authority intact, and retain any old visible tree until the replacement
section receipt is current. Do not paper over the backlog with a larger
startup timeout, weaker readiness check, or unmeasured per-tree priority
change.

The previous charter's "capture a stable exact-tree timeline before choosing
the queue fix" question is answered enough to stop repeating the same sprint
replay. The next production edit should implement the section-owned tree
geometry handoff described above, then prove its candidate is installed
through the real renderer before another long visual/traversal run. Stage 2
and the live gate remain open.

### Next implementation charter — recipe-fed section tree compiler (2026-10-04)

**Outcome:** move procedural tree visuals from per-tree scene construction to
the existing complete section candidate. A section compile reads immutable,
deterministic tree recipe inputs and produces the section's compatible render
layers. It must not wait for GeneratedTreeVisual construction or a per-tree
visual commit. Keep world generation, recipe/LOD choice, StaticBody3D
collision, prop identity, harvest/drops, navigation notifications, durable
removal, saves, and mob/NPC simulation under their existing authorities.

**Input ownership and content manifest:** seal each completed worker recipe as
a value record before renderer construction, bound to world/seed, stable source
and prop IDs, canonical normalized request, current runtime recipe signature,
recipe content/compiler schema, render tier, exact body weak owner and instance
ID, exact transform, and producer generation. The content revision includes
only deterministic render inputs (recipe content, geometry compiler version,
LOD, material/shader/uniform/texture digests, and layer policy); owner identity
and task generation are independent stale-result guards, not content identity.
The section census retains explicit complete/empty source membership and exact
tree-to-section geometry ownership, including cross-section trees and
cross-stream-chunk supports. Never substitute canopy bounds for actual
partitioned geometry. Each section candidate includes every contributor and
resource binding in its manifest, grouped only where mesh, material, render
layer, shadow and instance-attribute policy match.

**Geometry and scheduling:** add a composed TreeRecipeSectionCompiler that
consumes these frozen records through the same TreeSpawnService,
ProceduralTreeVisualFactory, and procedural recipe rules. Branch and foliage
may share canonical meshes while retaining exact per-instance transform,
custom data, wind, variation, tint and shadow behavior. The continuous
order-zero bole must preserve the current continuous-wood silhouette, bark
variation and shader semantics; merge it at section scope only if per-tree
attributes remain representable, otherwise keep distinct compatible batches
inside the section result. Work is resumable and retryable under the section
job budget; a missing, incomplete or stale recipe is pending/rejected, never an
empty success. Preserve each source's actual render layer; the current tree
adapter accepts opaque batches only, and the native install session rejects
translucent sorting. Close those layer limits explicitly before claiming visual
parity. The existing native install path remains the sole section renderer
and receives the normal complete manifest and current receipt.

**Stale work and replacement:** before enqueue and again after geometry
preparation/upload, validate current world/source revision, normalized request,
body owner/instance, generation token, transform, removed-prop snapshot,
resource/material fingerprint, compiler schema, affected-section set, and
candidate manifest. Cancel stale staged output and retain the old installed
section slot. A newly added tree keeps its collision/safety proxy until the
replacement receipt is current. A replaced/moved/retiered tree retains the old
visible sections until the union of old and new owning sections has current
receipts; only then retire the legacy visual/proxy and mark section ownership.
Removal remains an authoritative removed_props tombstone and becomes an
explicit source removal in every previously owned section. Separate
section/native installation acknowledgement from provider/tree retirement
acknowledgement in telemetry and contracts; the coordinator can currently
report an installed section while a provider acknowledgement is still pending.

**Ordered stages and exits:**

1. **Recipe artifact contract:** seal a completed deterministic recipe before
   per-tree geometry/scene construction. Prove exact immutable identity,
   content revision and stale-owner rejection, including moved, removed,
   replaced-body, resource/compiler revision and absent-recipe cases. Keep the
   old production renderer active.
2. **Section compiler contract:** compile all tree inputs for a target section
   into immutable layer/instance values, without creating or attaching
   GeneratedTreeVisual nodes. Compare mesh/resource fingerprints and exact
   section partitions against canonical tree values. Prove deterministic
   output, cross-section membership and bounded resumability.
3. **Real renderer installation:** capture an ordinary Main-scene forest
   source through ecology census, contribution, whole-section assembly, native
   install and receipt. Include a tree crossing section boundaries; verify all
   owning receipts, layer/material/shadow fields and old-visual retention
   throughout staging/upload.
4. **Gameplay and lifecycle parity:** prove the same live body remains
   collidable and harvestable; a removed tree writes the same durable delta and
   disappears from every installed section; save/reload, resource/revision
   replacement and stream unload/replay reconstruct the same deterministic
   content and reject stale RIDs/owners. Retire per-tree visual scheduling only
   after the preceding real receipt path passes.
5. **Live acceptance:** run headed forest visual and traversal checks, then a
   representative runtime performance profile. Report the exact run, screenshots,
   per-stage receipts, worst frame and tree/section queue work. This is still
   Stage 5 evidence; Stage 6 readiness, performance and legacy retirement is a
   separate goal gate.

**Minecraft 26.2 validation:** SectionCopy owns copied section-state values;
RenderRegionCache reuses adjacent copies during one extraction batch;
RenderSectionRegion exposes the 3×3 neighborhood; SectionCompiler.compile
builds the target section's nonempty render layers in shared builders; and
SectionRenderDispatcher publishes the replacement only after its buffers are
uploaded and accepted, retaining the old installed mesh in the meantime.
SectionTaskDynamicQueue.poll(cameraPos) selects nearest eligible section
work with a small recompile quota. Reuse these value-capture, per-section
compile, scheduling and replacement boundaries. Do not copy the cube/block
mesher for our smooth terrain or flatten the tree recipe into a generic
primitive.

**Entry evidence and stop rule:** the r3 replay recorded tree queue tasks still
in the completed backlog at renderStage=root and no prepared section artifact
for tens of seconds while the initial region stayed at 32/35 visuals. This
proves the old per-tree build can delay section admission, but does not prove it
is the only whole-view blocker. Keep ordinary-source capture throughput,
terrain/fluid probes, and section retry cadence as separate measured
dependencies. Do not repeat the same long sprint before the section compiler
has a real native-receipt proof; do not advance a stage from a snapshot or
synthetic contract alone.

### Tree producer generation and replacement currentness (2026-10-04)

The review of game commit `23f8db57` found that prepared tree artifacts were
sealed and retained without comparing their enqueue generation against the
body's latest expected generation. An older LOD build could therefore replace
a newer retained candidate. It also found that preparation/failure metadata
could make the still-attached accepted tree visual appear non-current to the
section adapter. The production queue now binds tree tasks to the current body
generation, transform, request tier and recipe signature; section-owned tasks
also require their retained immutable recipe-input record to match. Retention
rechecks generation/owner/transform before evicting an earlier artifact.
Preparation and failure states preserve the accepted per-tree/section visual,
and the adapter permits a current prepared replacement to be captured while
that accepted representation remains the installed source.
Game commit `686adb22` contains this currentness fix.

This boundary follows Minecraft 26.2's dispatcher pattern: supersede/cancel
outdated compile work and keep the old section mesh until replacement upload
is accepted. It is a currentness and replacement fix, not proof that our tree
compiler is yet section-scoped or that an artifact reaches the native renderer.

**Focused evidence:** `node tools/run-tree-publication-queue-contract.mjs`
passed on Godot 4.6.1. The headed synthetic adapter command
`node tools/run-tree-section-value-adapter-contract.mjs
--outputdirectory artifacts/citadel-runtime-integration/tree-section-value-adapter-generation-fence-20261004-r4`
passed all 22 checks, including stale-generation seal/replacement rejection
preserving the old visual while the new candidate remains censusable. Its
report is
`artifacts/citadel-runtime-integration/tree-section-value-adapter-generation-fence-20261004-r4/report.json`.
These are queue/adapter contracts and synthetic receipts; they do not prove
normal-world capture, native production installation, live traversal or
performance. A first adapter invocation without its required output directory
did not launch a process; the first headed run caught and then helped resolve a
GDScript parse error, and its owned watchdog proved zero remaining members.

**Stage decision:** this closes a stale-work defect in the Stage 2 tree-value
substage only. Keep Stage 2 partial until a section compiler consumes all tree
contributors and a real Main-scene candidate obtains current native receipts.
The earlier headed readiness timeout remains a separate failed gate; do not
attribute it to this fix or call live rendering/traversal/performance accepted.

### 2026-10-04 producer-value integration checkpoint

**Code evidence:** `TreePublicationQueue` now freezes section-ownership mode per
completed task and, in that mode, sends its existing resumable bole, branch and
foliage builders to the no-node finish methods. It captures mesh/material,
render policy, instance transforms, colors and custom data into sealed section
members, then waits for the section receipt before retiring the prior per-tree
visual. A failed/incomplete seal is rejected; it cannot publish an empty root.
The normal legacy publisher remains the fallback only for tasks admitted in
legacy mode and for impostor recipes that are not yet enumerable as section
values. Game commit `23f8db57` contains this producer change. Game commit
`3b12e1f6` adds receipt- and live-owner-fenced retirement for supported ordinary
generated block visuals; its scope keeps the StaticBody/collider/interaction
authority active and preserves boundary-section ownership through replay.

**Focused evidence:** `node tools/run-tree-publication-queue-contract.mjs`
passed on Godot 4.6.1, but its runner is headless and section-owned mode was
disabled for its tree publication assertions. It proves that legacy queue
behavior still passes, not that the new direct-value branch ran. The focused
ordinary structure retirement contract
`node tools/visible-world/run-ordinary-static-section-provider-contract.mjs
--OutputDirectory artifacts/citadel-runtime-integration/ordinary-static-section-provider-retirement-20261004-r7`
passed 18/18; it proves receipt-fenced visual retirement and boundary replay
only at the provider-contract level.

**Live gate:** `node tools/run-visible-world-fast-turn-sprint.mjs
--skip-tutorial --timeout-seconds 240` did not reach gameplay readiness. Its
progress remained `main_menu_waiting_for_gameplay_ready` through frame 3,720;
it produced no report. The owned-process watchdog timed out at 240 seconds and
proved zero remaining Job Object members. This is a failed startup gate and
does not exercise tree section compilation, sprint traversal or visual parity.
Do not attribute it to this producer change without an earlier comparable
baseline; the source logs contain no engine error beyond the timeout.

The corrected headed diagnostic run used
`node tools/run-visible-world-fast-turn-sprint.mjs --skip-tutorial
--diagnostic-replay-seed tree-section-values-20261004-r1 --timeout-seconds 360`.
The runner completed in 214 seconds and reported
`initial_region_readiness_timeout` on that seed: 34/36 visual candidates were
represented, including 14/16 trees; the remaining two were
`section_candidate_pending` without current receipts. Its timeout report also
observed 84 prepared tree section artifacts and tree member values, but did not
prove those artifacts were accepted by the native renderer. The report is
`artifacts/visible-world/fast-turn-sprint/report.json`; its owned-process record
is `artifacts/node-tools/process-runs/godot-23b6vJ/watchdog.json` (cleanup and
authoritative zero membership passed). A second attempt to run only
`production_section_candidate_diagnostic` through `node tools/run-playtest.mjs`
also stopped before the diagnostic act because Main startup timed out at the
same initial-region visual gate. Neither attempt included traversal or a
renderer receipt for the pending trees.

**Minecraft reference check:** the local 26.2 `SectionCompiler` groups output
by render layer from a captured region, and `SectionRenderDispatcher` keeps the
old section mesh until the replacement upload is accepted. That matches the
new producer-value boundary and fail-closed replacement behavior. Minecraft's
layer classification also confirms current tree foliage is safe as opaque:
its shader has no alpha/discard/blend semantics, while truly translucent
outputs require sorting that our native section installer does not yet support.

**Stage decision:** the code now reaches recipe-derived mesh values before
per-tree render-node construction, advancing the Stage 2 implementation
substage. The seeded headed run proves prepared tree value artifacts were
created in the real Main path; Stage 2 still needs proof those values enter a
complete real section candidate. Stage 3 native receipt, Stage 4 gameplay/save/
unload parity, Stage 5 headed visual/traversal/performance acceptance, and
Stage 6 remain open. Preserve the startup failure as a separate baseline and
continue migration work without attributing it to this producer change.

### Recipe-fed tree section compiler progress (2026-10-04)

Game commit `c3ee1619` adds the first queue-to-compiler handoff. While
section-owned publication is enabled, a completed recipe is sealed before the
legacy per-tree visual stages. `tree-section-recipe-input/v1` captures the
normalized request and recipe, stable seed/prop/source identity, render tier,
runtime recipe signature, deterministic content digest, compiler revision,
body weak owner/instance, producer generation, transform and logical owner
cell. A re-enqueue generation change, body replacement/cancellation or moved
transform rejects the artifact. The old per-tree visual path remains active as
the fallback; this capture is not yet connected to ecology census or native
section installation.

`ProceduralTreeVisualFactory` now exposes node-free resource/value finish
methods for bole, distal branches and foliage. Existing scene-node finish
methods wrap those same outputs and share the render policy. This provides the
compiler boundary needed to reuse deterministic runtime geometry without
creating `GeneratedTreeVisual` nodes inside section compilation.

**Evidence:** `node tools/run-tree-publication-queue-contract.mjs` passed,
including early recipe availability before `GeneratedTreeVisual` exists and
stale-owner rejection after movement. Captured stdout and watchdog evidence are
under `artifacts/node-tools/process-runs/godot-PFPSPC/`; the runner proves a
queue contract only, not installed rendering. A broader
`node tools/run-canopy-runtime-contract-tests.mjs` attempt did not produce a
report: its output showed missing `static_ecology_source_value_ledger`
metadata from `MainPlaytestTools`, and the owned-process watchdog confirmed
zero remaining processes. Treat that result as unresolved setup/integration
evidence, not as a failure caused by this change.

This advances only the tree recipe-artifact substage. The compiler job queue,
exact per-section recipe geometry and coverage, ecology census/contribution
integration, real native receipt, old-visual retirement, foliage layer parity,
and headed visual/traversal/performance checks remain open. Overall migration
status remains 7 stages total: Stage 0 complete; Stages 1–5 partial; Stage 6
not started.

### Next implementation charter — recipe-fed section tree compiler (2026-10-04)

**Outcome:** move procedural tree visuals from per-tree scene construction to
the existing complete section candidate. A section compile reads immutable,
deterministic tree recipe inputs and produces the section's compatible render
layers. It must not wait for GeneratedTreeVisual construction or a per-tree
visual commit. Keep world generation, recipe/LOD choice, StaticBody3D
collision, prop identity, harvest/drops, navigation notifications, durable
removal, saves, and mob/NPC simulation under their existing authorities.

**Input ownership and content manifest:** seal each completed worker recipe as
a value record before renderer construction, bound to world/seed, stable source
and prop IDs, canonical normalized request, current runtime recipe signature,
recipe content/compiler schema, render tier, exact body weak owner and instance
ID, exact transform, and producer generation. The content revision includes
only deterministic render inputs (recipe content, geometry compiler version,
LOD, material/shader/uniform/texture digests, and layer policy); owner identity
and task generation are independent stale-result guards, not content identity.
The section census retains explicit complete/empty source membership and exact
tree-to-section geometry ownership, including cross-section trees and
cross-stream-chunk supports. Never substitute canopy bounds for actual
partitioned geometry. Each section candidate includes every contributor and
resource binding in its manifest, grouped only where mesh, material, render
layer, shadow and instance-attribute policy match.

**Geometry and scheduling:** add a composed TreeRecipeSectionCompiler that
consumes these frozen records through the same TreeSpawnService,
ProceduralTreeVisualFactory, and procedural recipe rules. Branch and foliage
may share canonical meshes while retaining exact per-instance transform,
custom data, wind, variation, tint and shadow behavior. The continuous
order-zero bole must preserve the current continuous-wood silhouette, bark
variation and shader semantics; merge it at section scope only if per-tree
attributes remain representable, otherwise keep distinct compatible batches
inside the section result. Work is resumable and retryable under the section
job budget; a missing, incomplete or stale recipe is pending/rejected, never an
empty success. Preserve each source's actual render layer; the current tree
adapter accepts opaque batches only, and the native install session rejects
translucent sorting. Close those layer limits explicitly before claiming visual
parity. The existing native install path remains the sole section renderer
and receives the normal complete manifest and current receipt.

**Stale work and replacement:** before enqueue and again after geometry
preparation/upload, validate current world/source revision, normalized request,
body owner/instance, generation token, transform, removed-prop snapshot,
resource/material fingerprint, compiler schema, affected-section set, and
candidate manifest. Cancel stale staged output and retain the old installed
section slot. A newly added tree keeps its collision/safety proxy until the
replacement receipt is current. A replaced/moved/retiered tree retains the old
visible sections until the union of old and new owning sections has current
receipts; only then retire the legacy visual/proxy and mark section ownership.
Removal remains an authoritative removed_props tombstone and becomes an
explicit source removal in every previously owned section. Separate
section/native installation acknowledgement from provider/tree retirement
acknowledgement in telemetry and contracts; the coordinator can currently
report an installed section while a provider acknowledgement is still pending.

**Ordered stages and exits:**

1. **Recipe artifact contract:** seal a completed deterministic recipe before
   per-tree geometry/scene construction. Prove exact immutable identity,
   content revision and stale-owner rejection, including moved, removed,
   replaced-body, resource/compiler revision and absent-recipe cases. Keep the
   old production renderer active.
2. **Section compiler contract:** compile all tree inputs for a target section
   into immutable layer/instance values, without creating or attaching
   GeneratedTreeVisual nodes. Compare mesh/resource fingerprints and exact
   section partitions against canonical tree values. Prove deterministic
   output, cross-section membership and bounded resumability.
3. **Real renderer installation:** capture an ordinary Main-scene forest
   source through ecology census, contribution, whole-section assembly, native
   install and receipt. Include a tree crossing section boundaries; verify all
   owning receipts, layer/material/shadow fields and old-visual retention
   throughout staging/upload.
4. **Gameplay and lifecycle parity:** prove the same live body remains
   collidable and harvestable; a removed tree writes the same durable delta and
   disappears from every installed section; save/reload, resource/revision
   replacement and stream unload/replay reconstruct the same deterministic
   content and reject stale RIDs/owners. Retire per-tree visual scheduling only
   after the preceding real receipt path passes.
5. **Live acceptance:** run headed forest visual and traversal checks, then a
   representative runtime performance profile. Report the exact run, screenshots,
   per-stage receipts, worst frame and tree/section queue work. This is still
   Stage 5 evidence; Stage 6 readiness, performance and legacy retirement is a
   separate goal gate.

**Minecraft 26.2 validation:** SectionCopy owns copied section-state values;
RenderRegionCache reuses adjacent copies during one extraction batch;
RenderSectionRegion exposes the 3×3 neighborhood; SectionCompiler.compile
builds the target section's nonempty render layers in shared builders; and
SectionRenderDispatcher publishes the replacement only after its buffers are
uploaded and accepted, retaining the old installed mesh in the meantime.
SectionTaskDynamicQueue.poll(cameraPos) selects nearest eligible section
work with a small recompile quota. Reuse these value-capture, per-section
compile, scheduling and replacement boundaries. Do not copy the cube/block
mesher for our smooth terrain or flatten the tree recipe into a generic
primitive.

**Entry evidence and stop rule:** the r3 replay recorded tree queue tasks still
in the completed backlog at renderStage=root and no prepared section artifact
for tens of seconds while the initial region stayed at 32/35 visuals. This
proves the old per-tree build can delay section admission, but does not prove it
is the only whole-view blocker. Keep ordinary-source capture throughput,
terrain/fluid probes, and section retry cadence as separate measured
dependencies. Do not repeat the same long sprint before the section compiler
has a real native-receipt proof; do not advance a stage from a snapshot or
synthetic contract alone.
