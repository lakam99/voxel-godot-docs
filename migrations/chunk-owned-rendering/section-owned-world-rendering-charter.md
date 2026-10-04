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
| 2. Implement section-slot installation | Replace the source-keyed production packet primitive where necessary with a section-keyed staged owner independent of gameplay-chunk lifetime. Support compatible multi-batch/multi-layer payloads, exact manifests, source-revision-bound capture dependencies, cancellation, owner recreation, explicit empty, and old-slot retention until complete install. | Native/engine contract exercises real section owner/resource installation and receipt checks; cross-chunk captured candidates install without retaining source chunks; stage aborts leave old content installed; native build and shutdown/retirement pass. |
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
tombstones for all intersecting sections, and apply multi-section atomic
replacement for contributors crossing section boundaries. Preserve gameplay
chunk unload/replay and save/delta authorities. Minecraft 26.2 validates the
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
game's translucent/fluid policies; (3) multi-section staging and one atomic
promotion after every impacted section has an exact live receipt; (4) stale
source/owner cancellation and replay after source/render-owner unload; and
(5) production producer routing that retains each old representation until
the complete replacement is acknowledged. Mobs/NPCs, collision, interactions,
navigation and saves stay with their established authorities.

Stage exits remain the seven rows in the table above: Stage 0 source map;
Stage 1 complete immutable manifest/census contract; Stage 2 section slot,
multi-layer/multi-section install lifecycle; Stage 3 real terrain source and
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
remains partial (opaque-only native installation and no atomic multi-section
group commit); Stage 3 visual/edit/collision/fluid/light parity and Voxel Tools
visual retirement remain untested. Stages 4–6 remain open. Keep the migration
active. The verified game slice is committed at
`9f1168ecac0760eed458e68eda7ae5984a36cb46` on
`codex/chunk-owned-world-rendering-migration`; generated `.import` churn remains
unstaged in the game worktree.
