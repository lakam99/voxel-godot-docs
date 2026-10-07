# Building producer cutover: prepare before publishing

Status: architecture reviewed; implementation and acceptance pending. This is a
remaining production cutover within the [section rendering charter](section-owned-world-rendering-charter.md),
not a new rendering authority or a completed stage.

## Outcome and baseline

Building geometry must first become visible through the complete section
transaction. Preserve generated shapes, door motion, collision, interaction,
navigation, deterministic source records and durable save deltas. NPC simulation
and routing remain independent; no new Main inheritance layer or renderer is needed.

Game baseline is `831223fb10b86bdbc7dbf2a7ec916b8829e30a98` on
`codex/chunk-owned-world-rendering-migration`, with the existing uncommitted
migration batches and unrelated import churn. Preserve them. Current native
attachment and service gates pass, but full31, live doors and ordinary Main
acceptance remain outstanding. Record those integration results before promoting
this next producer change.

## Actual boundary and reference

`BuildingPartPublisher.publish_part` constructs visible geometry before section
admission. Static batch upload and deferred masonry, paving and roof sinks have
the same publication concern. `BuildingScenePublicationJob.visual_receipt_installed`
currently accepts visible scene witnesses or old per-source packet receipts.
Hiding geometry alone would leave that readiness dependency unsatisfied.

Ordinary structures use a different producer: `StructureSystem` calls
`MainChunkTerrain.create_block`, which builds the body and visual children while
detached, then attaches the body. Stage those visuals before that attachment;
do not force ordinary structures through the Citadel scene job. Citadel attaches
bodies earlier and spans deferred sinks, requiring staging at each sink.
`visual_receipt_installed` is consumed by `GeneratedStructureVisualManifest` and
`ChunkPropVisualManifest`; it must retain its actual-installation meaning. Add
source-prepared evidence at preparation consumers, not by redefining this method.

The local Minecraft 26.2 reference was inspected in `RenderSectionRegion`,
`SectionCompiler` and `SectionRenderDispatcher.checkSectionMesh`: captured inputs
produce a complete set of render layers; publication waits for their uploads,
then swaps and releases the old section. Apply that lifecycle to our smooth
terrain and compound animated attachments without copying the block mesher.

## Single production contract

1. Bind the section publication context before a production building job starts:
   world identity, source revision, publisher incarnation and cancellation owner.
   Missing context leaves production admission pending or explicitly failed.
2. Every geometry sink stages its source hidden before attachment to the scene
   tree, including deferred batches. Preserve intended visibility in the existing
   authoritative capture identity. Keep motion parents, bodies and colliders
   intact; hiding a shared root would hide the native replacement too. Practical
   lights need equivalent staged activation.
3. Distinguish source preparation from visible installation within the existing
   job. Prepared means immutable source artifacts plus live physical/source
   boundary proof. Visible means a current, acknowledged complete section
   installation. Source census must accept prepared hidden inputs; loading and
   movement gates must continue to require actual installation.
4. Carry managed publication and intended visibility through source bindings,
   manifest identity and native claims. Provider reconciliation never reveals
   capture-only source geometry on stale, missing, failed or released receipts.
   Existing historical visible fallback restoration remains explicit and cannot
   become an alternate production path.
5. First-install failure retains hidden preparation and retryable demand, keeping
   readiness pending. Replacement failure keeps the old valid section visible.
   Unload and cancellation drain exact transaction owners before releasing source
   roots. Saves persist durable gameplay deltas, never temporary suppression state.

## Consumer closure and risks

The implementation batch must cover publisher/door capture, mesh upload/static
flush/deferred sinks, scene job witnesses and cancellation, Citadel and ordinary
providers, coordinator/roster/assembler/install session/native claims, and Main,
StructureSystem and tutorial readiness admission. Keep exact body, shape, door,
interaction and navigation identities. Source resources must remain available to
capture without needing a prior visible installation.

Physical preparation before visible ACK is safe only while the existing
playable-region/loading gate retains that dependency. Hidden blockers during
ordinary traversal are a failure. Audit practical lights and originally-hidden
members explicitly; do not infer their intended state from staged visibility.

Further source tracing identifies three prerequisite gaps before promotion:

- `OrdinaryStructureBlockVisualRecipe` classifies door, chest, furnace, campfire
  and torch as `separate_dynamic`; the current ordinary adapter returns pending
  with `ordinary_geometry_dynamic_family_requires_separate_owner` for them.
  Their presentation needs real owner-bound contributors
  before it can be staged. Preserve their independent gameplay behavior.
- `FireLight3D._process` updates visibility from daylight and LOD. A one-time
  hidden flag is insufficient. Evaluate a source-owned presentation mount claimed
  by the existing native transaction, with explicit borrowed ownership; native
  retirement must not free the source-owned mount or replace lighting authority.
- Current terrain motion proof checks terrain/chunk collision and reports
  `supportRequiredForMotion:false`; structures report `structures_physical_only`.
  Extend the existing readiness composition to relevant installed static support
  before hiding source presentation. An unchanged valid previous representation
  can satisfy that dependency; do not expand local movement to an entire town.

Player-placed blocks are also a remaining contributor domain. Placement uses
`MainInteractionFlow.place_selected_block` and `MainChunkTerrain.create_block`;
`MainSaveState.snapshot_player_blocks`/`restore_player_blocks` own their durable
records. Current generated-source registration excludes them. Reuse those
authorities and shared geometry adaptation; do not mislabel player blocks as
generated or invent a second placement/save system.

## Verification and exit

- Entry: current full31/live baseline recorded, every sink and both provider
  admission paths mapped, independent review finds no preparation/visibility cycle.
- Focused proof: cold install and same-geometry replacement-owner cases spanning
  multiple compile/upload frames; unchanged real physical identities; no source
  geometry or staged-light leak; old valid section survives replacement failure.
- Headed proof: full31, real swing/raise door input, already-open capture,
  originally-hidden members, stale revision, provider reconciliation, unload and
  reentry, reset/quit. Inspect captured frames, not only metadata.
- Product proof: ordinary Main and tutorial loading, save/reload and representative
  sprint/traversal performance. No production per-source visual escape remains.

HEAD owns final acceptance. A green source contract or isolated renderer test
does not close this migration stage.
