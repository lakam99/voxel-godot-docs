# Cave authority replacement handoff

## Goal and product decision

Replace the failing native cave-generation authority with the working
`ProceduralCaveField.gd` system. The GDScript field is the intended generation
authority going forward; do not preserve the old native cave behavior as a
fallback. Preserve the gameplay contracts built on terrain volume: solid/air
state, material selection, collision, digging, navigation, lighting, and save
deltas. Tutorial-town startup may be excluded from cave-focused diagnostics,
but that evidence is not ordinary-startup acceptance.

## Current state

- The GDScript cave field, generation contract, authority contract, native
  source contract, physical fixture, and cave walkthrough are present in the
  game repository.
- Native terrain still owns `NativeProceduralCaveField`, a C++ port of the
  GDScript generation policy. It is used by effective, natural, and standalone
  terrain source paths. Do not delete it until native sampling consumes
  immutable, identity-bound inputs from the GDScript authority; deleting it
  prematurely would make terrain, collision, navigation, and digging disagree.
- The current direction is to pass canonical cave recipe/field inputs from
  GDScript through the native source request and immutable source identity, then
  have native code evaluate/consume those inputs without independently
  generating cave policy. Resolve whether bounded recipe pages are sufficient
  or whether sampled density pages are required for exact authority. Keep the
  handoff bounded and revision/digest-bound; do not substitute durable save
  deltas or scene overlays for generated caves.
- Latest focused live diagnostic passed:
  `node tools/run-cave-walkthrough.mjs --diagnostic --record false --seed atlas-1492 --timeout-seconds 300`
  Report:
  `artifacts/caves/walkthrough-2026-09-30T00-35-03-791Z-427ec3dea6/report.json`
- It proves live traversal and mining through production gameplay APIs in
  diagnostic fast boot: generated stone became edited underground air, a stone
  reward was granted, the edit entered the save delta, publication drained,
  and a later player physics ray hit adjacent terrain. It does not prove
  ordinary menu/New Game startup, torch-on-spawn, disk save/reload, or native
  replacement completion.
- The critic found no new exterior-sky leak at the dig site, but the before/after
  screenshots do not make the removed voxel visually obvious. Camera position
  shifts between captures and the diagnostic fill light is enabled. Next visual
  proof should use a fixed camera origin/look, a slight oblique angle, and a
  matched non-diagnostic light configuration when feasible.
- Godot reports a small ObjectDB resource leak at diagnostic process exit;
  runner watchdog exit and authoritative cleanup/zero-member evidence passed.

## Immediate next work

1. Inspect the current branch diff before changing anything. This checkout has
   pre-existing import churn and unrelated local artifacts; do not reset, clean,
   or blanket-stage them.
2. Finish the cave source contract: define the GDScript-produced immutable
   cave input, validate canonical ordering/bounds/overlap, bind digest and cave
   generation revision into source identity, and reject stale pages/receipts.
3. Wire that input through `NativeWorldSourceRequest`, native source definition
   and page dependency closure. Preserve bounded demand, deterministic query
   order, and durable-edit precedence (durable edit over generated cave; clear
   reveals generated cave).
4. Add focused contracts for recipe/input parity, negative coordinates,
   entrance continuity, regional overlap, page clipping, source revision
   invalidation, durable solid/air/clear behavior, and exclusion of scene-only
   overlays.
5. Validate the exact native build and production backend against the same
   fixed seed/coordinates; compare signed density/material and then verify
   rendered terrain, real collision, navigation occupancy, mining, and save
   round-trip. Only then retire/delete `NativeProceduralCaveField` and its tests
   and manifest entries.
6. Complete visual acceptance with paired, fixed-origin screenshots and
   inspect them. A green source contract or changed physics ray alone is not
   sufficient proof of cave visual quality.
7. Separately resolve the player request that a torch be equipped at ordinary
   spawn. The cave fixture currently stages its torch; that is not proof the
   production spawn path equips one. Also verify the HUD is visible after
   gameplay readiness, not merely that its node was constructed.

## Key code locations

Game repository (master checkout):

- `scripts/world/ProceduralCaveField.gd`
- `scripts/WorldGenerationSystem.gd`
- `scripts/terrain/NativeWorldSourceRequest.gd`
- `native/world_backend/core/native_procedural_cave_field.{hpp,cpp}`
- `native/world_backend/core/native_effective_terrain_source.cpp`
- `native/world_backend/core/native_natural_terrain_source.cpp`
- `native/world_backend/core/world_source.{hpp,cpp}`
- `scripts/testing/terrain/CaveWalkthroughRunner.gd`
- `scripts/testing/native_world/NativeCaveSourceContractRunner.gd`
- `scripts/testing/native_world/NativeCavePhysicalFixture.gd`

## Verification and reporting rules

Use the cave-specific Node runners in `tools/`; keep test runs focused and
owned by their watchdog. Distinguish unit/contract, diagnostic live gameplay,
and ordinary-startup acceptance in every report. Record command, exact source
revision/build identity, report path, seed, screenshots, and cleanup evidence.
Do not describe this migration as complete until the independent native cave
generator is removed, native terrain consumes the canonical GDScript authority,
and visual/physical/digging/save/navigation proof passes on the exact integrated
build.
