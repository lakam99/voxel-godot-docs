# Fluid Section Rendering Plan (2026-10-05)

**Status:** active implementation substage; fluid-bearing terrain remains fail-closed in the section candidate path.

## Outcome and authority

Move water and lava visuals into the terrain section candidate as section-local,
camera-sorted baked meshes. `TerrainVolumeService` and the deterministic world
generator remain the material/solidity/fluid authority. Exact section volume
payloads and all intersecting volume/fluid revisions must bind the mesh and its
face groups. Water and lava retain separate compatible material batches. The
section slot owns only their visuals; Voxel Tools remains terrain collision and
smooth SDF authority until broader parity gates pass. Saves, fluid simulation,
terrain edits, lights, mobs/NPCs, and gameplay interaction are out of scope.

## Current path and gap

`TerrainVolumeService` / `WorldGenerationSystem` ->
`VoxelTerrainRuntime.request_terrain_section_fluid_probe` requests exact section
payload and halo -> runtime validates and retains a revision/signature proof,
but discards the payload -> `TerrainSectionShadowPublisher` blocks sections
with `hasFluid` -> the separate `TerrainMeshingService` requests exact
28-cell-wide chunk payload -> native `build_chunk_fluid_surface_data_from_sections`
emits unindexed water/lava vertex arrays -> `TerrainMeshingService` finalizes
chunk ArrayMeshes -> `MainRuntimeTools.apply_chunk_fluid_mesh` attaches them to
the streamed gameplay chunk.

Thus the current section census has fluid revisions but no fluid mesh content;
the chunk fluid visual has mesh content but no section ownership or camera-sort
face-group manifest. The section install session now accepts only a sealed
translucent face-group descriptor tied to actual mesh arrays, section generation,
and POV revision. HEAD owns the coordinator integration
`current_translucent_pov_snapshot(section_key)` returning
`{status:'ready', cameraPosition:Vector3, povClass:Vector3i, revision:int}` plus
the current receipt/session handoff. `povClass` is section-relative and clamped
to {-1,0,1} per axis, matching Minecraft 26.2's `TranslucencyPointOfView`;
mesh sorting uses the supplied exact camera position. POV changes request
replacement from retained canonical face groups while the old slot stays visible.

## Ordered implementation and evidence

1. **Exact source value:** retain or recapture one bounded immutable section
   payload only for an active section candidate. Bind section key, fluid and
   volume revisions, halo revision rows, and signature. Validate freshness
   before and after meshing. Do not retain an unbounded map of dense payloads.
   Prove empty, fluid-bearing, stale-edit, cancellation, and retry behavior.
2. **Section-owned mesh preparation:** extend native fluid output to emit only
   faces owned by the requested 16-cell 3D section, with water/lava material
   separation and stable face groups preserving each quad's two triangles.
   Include exact section bounds and halo validation; adjacent sections must not
   omit or duplicate boundary faces. Sort groups far-to-near from an immutable
   camera snapshot and emit actual-mesh centroid/range descriptors. Keep this
   producer resumable/budgeted and preserve existing chunk fluid visuals.
   Prove seam partition, determinism, mesh digest, group tampering and stale
   source rejection with focused native mesh and terrain publisher contracts.
3. **Complete section candidate:** submit explicit empty fluid rows when a
   current exact proof has no fluid; otherwise add the prepared translucent
   water/lava batches to the same complete terrain section candidate. Reject
   missing, duplicate, stale, unsupported, or partially meshed fluid content.
   Prove the real native renderer installs first ordering and retains the old
   slot through failed/stale replacement.
4. **POV and lifecycle integration:** consume a HEAD-owned current POV snapshot
   and revision at install advancement; reject stale POV before upload and
   commit. Preserve current Voxel Tools/fluid visuals until a matching section
   receipt, then retire only the replaced visual representation. Verify edit,
   save/reload, section/chunk unload and replay behavior; collision and fluid
   simulation stay on their current authorities.
5. **Live acceptance:** headed water/lava visual and traversal checks across
   section and gameplay-chunk boundaries, then representative streaming
   performance. Report screenshots, source/section receipts, stale rejection,
   worst frame, and queue work. Earlier synthetic/native fixtures do not pass
   this gate.

## Architecture reference

Minecraft 26.2 `SectionCompiler` extracts a section from `RenderSectionRegion`,
emits render-layer meshes, and captures translucent `MeshData.SortState`.
`MeshData` sorts quad groups by centroid and produces indices;
`SectionRenderDispatcher` replaces sort indices only after async sort/upload is
accepted and keeps the old mesh visible during replacement. Apply those
ownership, layer, group-sort, and old-until-accepted boundaries. Keep the
game's exact fluid authority and smooth SDF terrain mesher; Minecraft's block
mesher is not a fit for this terrain.

## Baseline and stage rule

Game worktree: `codex/chunk-owned-world-rendering-migration`; pre-existing tree
contains concurrent ecology/structure changes and generated import churn. The
native packet/session fixture command is
`node tools/run-native-chunk-render-packet-contract.mjs -OutputDirectory artifacts/citadel-runtime-integration/native-chunk-packet-translucent-session-20261005-r8`;
its report passed the section-baked backend swap and session validation gates,
but it contains synthetic fluid geometry and proves no production fluid capture
or live visual. Record the exact game `HEAD` and focused source/native reports
with each subsequent stage. HEAD owns stage exits; do not mark this substage or
the overall migration complete on contract evidence alone.
