# Section-Owned World Rendering: Architecture Charter and Stage Plan

**Status:** active migration charter; production cutover not started
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
| Buildings and generated structures | `CitadelSiteBuildQueue` → `BuildingPublicationPreparation` → `BuildingScenePublicationJob` → `BuildingPartPublisher`. Current static visuals group by material, tier, logical owner cell and source part, then `BuildingStaticBatchFlush` commits individual `ChunkRenderPacketBackend` packets. | Source-part revision is derived from its structural snapshot. 32-cell logical owner, 28-cell stream owner and render-section identity differ. Collision batches, door state/interactions, furnishings and navigation manifests remain structure-owned. Generated structures are reconstructed; durable player block edits/removals are saved separately. Per-source packet replay recipes are retained across chunk recreation. | Current packet commits are atomic only per source/material/tier group; a later group can fail after earlier visuals are visible. Replace this production grouping with complete section candidates and per-section contributor receipts. Do not merge collision or door authority into render buffers. |
| Trees and foliage | Deterministic chunk RNG and stable prop IDs feed `TreeRuntimeRequestBuilder`, `TreeEcologySampler`, `TreeSpawnService` and `TreePublicationQueue`. Near/mid visuals install per tree; horizon ecology has separate batches/impostors. `TreeChunkBatchRenderer` is a prototype, not production authority. | Tree body/trunk collision, harvest/drop metadata, prop ID and navigation blocking are gameplay-owned. Branch/leaf collision is not required. Removed prop IDs are saved; recipes and geometry regenerate from seed/authority. Preserve legacy RNG draw order, including compatibility draws. Horizon sources are retired/promoted separately from physical chunks. | Capture admitted canonical recipe/LOD outputs without re-running ecology or reading visuals back from nodes. Include branch/canopy bounds and cross-section coverage. Replace old tree visuals only after the complete affected section set is accepted; keep body, harvest identity and collider alive. |
| Props and decorative detail | `MainPlaytestTools` produces deterministic props and detail transforms per chunk. Rocks/ore/forage become per-prop bodies with visual children; decorative detail becomes typed `MultiMesh` batches under `DecorBatches`. `ChunkPropVisualManifest` observes candidate/represented state for readiness. | Prop source revisions include seed, chunk identity and completed candidate content. Harvested IDs are stored in `removedProps`; visual geometry regenerates. Physical props keep their collision and interaction bodies. Horizon-only roots are visual-only. | Capture sealed producer values before scene-node materialization, preserving transforms, colors, custom data, wind/material policy and stable IDs. A manifest/readiness observer is not the renderer. |
| Wildlife / NPCs | Wildlife and NPCs use animated, collision-aware actor bodies and independent behavior/movement schedules; distant visual-only wildlife may be separately represented. | Actor state, simulation distance, collision, interaction and save data are actor-owned. | Keep live mobs/NPCs outside static section buffers. Do not mistake their visual-only far proxies for simulation or physical readiness. |

Current application-level gaps and source evidence are recorded in the active
game migration document and its reports. The per-source packet baseline passes
24/24 checks at
`artifacts/citadel-runtime-integration/native-chunk-packet-baseline-pre-section-cutover-20261004/report.json`;
the static replay fixture passes 5/5 at
`artifacts/citadel-runtime-integration/static-flush-baseline-pre-section-cutover-20261004/report.json`.
The latter uses a fake backend. Neither is whole-section, terrain, or live
gameplay acceptance. The snapshot/grid/partition/ledger contracts are likewise
data-level evidence only. The game worktree is at `879c2e15`; it has uncommitted
building packet bridge and contract changes plus generated `.import` churn.
Review those changes before replacing the production path; preserve unrelated
files and import churn. The documentation repository is on `main` at `47d03ea`
with pre-existing edits to the visible-world readiness plan and implementation;
those edits are outside this charter.

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
- canonical streamed-chunk owner identity and every intersecting residency
  dependency, with each dependency's current generation. Cross-section geometry
  has one canonical visual owner and explicit coverage ranges/dependencies.

Preparation consumes owned value snapshots, never live Nodes, mutable producer
containers, or scene scans. Work is bound to world, section, source, and owner
revisions. Cancellation or section-slot reassignment invalidates older work.
Immediately before install and before promotion, revalidate the exact source
set, world epoch, candidate generation, live owner chunk/backend identity, and
all dependency residency generations. A stale or missing source is pending or
failed, never empty success.

Build every required layer in staging while retaining the old installed
section. Atomically replace the section slot only when all present layer
resources and the exact contributor manifest have install acknowledgements.
Publish an explicit empty replacement when the current manifest has no content.
On failure/cancellation, retire only staged data and keep the last valid
section. Receipts map contributors to all affected sections; contributor
readiness is promoted only after those section receipts are live. A Godot node
installation receipt is not a GPU fence and must be reported as such.

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
| 2. Implement section-slot installation | Replace the source-keyed production packet primitive where necessary with a section-keyed staged owner. Support compatible multi-batch/multi-layer payloads, exact manifests, dependency pins, cancellation, owner recreation, explicit empty, and old-slot retention until complete install. | Native/engine contract exercises real section owner/resource installation and receipt checks; stage aborts leave old content installed; native build and shutdown/retirement pass. |
| 3. Integrate smooth terrain | Feed mesh artifacts from the existing authoritative SDF/material source into the section candidate using the game's mesher and correct halo/seam contract. Keep collision, edit, fluid/light and nav authorities separate but revision-linked. Do not disable Voxel Tools visuals until parity and replacement behavior are proven. | Real-scene section receipt for terrain; edit/remesh/empty/stale/cancel/unload/reload fixtures; seams, collision, lighting and fluid captures show source parity. Then retire the old terrain visual slot with no duplicate visual authority. |
| 4. Cut over construction | Aggregate every affected building's opaque/material groups into section candidates. Remove per-source packet commits as the final visual authority. Reconcile source-part revisions, moved owners, all affected sections, empty removals, replay and scene-boundary readiness; preserve collision/doors/furnishings/nav. | Real generated structure visual playtest and captures; replacement/removal/cancel/replay and chunk recreation tests; boundary readiness cannot succeed on a partial section. |
| 5. Admit ecology and static props | Feed accepted tree recipes/LODs, decorative detail buffers and prop visual recipes through the same section manifest. Preserve color/custom/wind attributes and material layers. Replace per-tree/per-prop visual nodes only after all affected section receipts are accepted; retain their bodies/interactions. Keep wildlife/NPC actors independent. | Deterministic source parity and removal/save/reload tests; live forest/prop visual captures and traversal show complete coverage without pop or hitch. |
| 6. Readiness, performance and legacy retirement | Wire section receipts into visible-world readiness and chunk unload/replay. Remove obsolete production per-source visual publication only after all consumers have migrated. Run normal startup, movement/turn, edit, harvest, save/reload and unload/recreate journeys. | Headed live visual/traversal evidence across representative seeds; visual readiness has no candidate/receipt gaps; performance report includes startup, p95/max cadence, queue/backlog and streaming spikes; no legacy production renderer remains for migrated categories. |

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
  bridge supports opaque content only, rejects cross-chunk dependencies until
  their demand can be pinned, and has no normal-world producer callsite. No
  headed generated-world/traversal or performance report demonstrates live
  section-owned rendering yet.
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

Do not advance a stage because a pure contract or source scan passes. For each
stage report passed, failed, blocked and untested gates separately, preserve
prior installed content on failed replacement, and keep the overall migration
active until all required domains pass live acceptance.
