# Visible-World Readiness Plan

**Status:** implementation in progress; live acceptance remains open
**Recorded:** 2026-10-01  
**Game baseline inspected:** `voxel-godot` master at `3d957f24`

**Implementation record:** [Visible-world readiness implementation, 2026-10-01](visible-world-readiness-implementation-2026-10-01.md)

## Goal

Make visible-world completeness a streaming contract, not a promise to load a few nearby forests sooner. The player should receive control only after the immediate playable world is present. Content farther away may be cheaper, but anything inside the configured view must already have a coherent visual representation. During travel, keep movement responsive and promote detail before content enters the near field.

Keep the existing final view distance and deterministic content/material choices. Do not hide pop-in by increasing draw distance, gating ordinary movement on distant full detail, or adding visual-only substitutes that claim collision, navigation, or interaction readiness.

## Baseline findings

At the inspected baseline, normal startup already waits for final terrain view-distance expansion before enabling player control. That gate is not a complete visual-world receipt: it checks terrain-view expansion conditions, retained gameplay chunks/viewers, native task backlog, and quiet frames, but does not prove that the visible terrain mesh coverage and generated ecology are complete.

The later coordinator readiness query uses the player's foreground region. `WorldStreamingCoordinator.region_readiness()` certifies terrain, physical structures, and navigation (with retained demand bookkeeping); `StructureSystem.region_publication_readiness()` is explicitly physical-only. Neither proves tree/foliage, prop, wildlife, or far-view structure visuals are present. Prop construction is incremental through per-chunk queues; procedural trees can remain queued after their body is created. Existing view-priority and tree LOD machinery are useful, but there is no exact region-level expected-versus-represented visual receipt joining them.

These findings describe the inspected source revision only. Recheck current code before implementation, especially startup ordering and provider interfaces.

## Target contract

Extend `WorldStreamingCoordinator.region_readiness(...)` with a required `visual` domain while preserving the existing physical/navigation checks as separate contracts. A visual readiness result should include:

- structured `ready`, `pending`, or `failed` status and a concrete reason;
- candidate and represented counts, plus pending counts by content kind (terrain, structures, trees/foliage, props, wildlife);
- representation tier/counts (near detail versus far/horizon representation);
- the exact world/source revision and visual-demand/view revision it covers;
- bounded queue and coverage-lag diagnostics.

Readiness must be based on complete candidate enumeration and accepted owner receipts, not a scene-node count alone. An empty candidate set is ready only after the deterministic source has finished describing that region. Revalidate revisions after collecting receipts; stale or replaced work is pending, never success. A far visual proxy may satisfy visual presence only. It must not satisfy collision, navigation, or interaction readiness.

## Implementation sequence

1. **Define the visual ownership boundary.** Add a visual-representation owner/query separate from physical structure publication. Define its immutable candidate-manifest and representation-receipt shape, stable IDs, content kinds, near/far tiers, source identity, and monotonically advancing view-demand revision. Keep physical terrain/structure/navigation readiness unchanged.
2. **Join existing publishers honestly.** Terrain mesh receipts, structure visual receipts, deterministic per-chunk prop candidate completion, tree recipe publication/LOD, and wildlife visuals must each report exact accepted coverage. A queued tree or incomplete prop candidate scan remains pending. Candidate manifests must preserve existing seeded RNG order and material/density decisions.
3. **Cover the view without expanding gameplay simulation.** Use the configured final view and existing near/far ranges. Prioritize the camera/movement corridor for detail promotion while maintaining low-cost representation around the full horizon for turns. Swap representations without a blank interval; retain retryable demand and reject stale source/view completions. Do not require far-world collision or active wildlife simulation beyond existing gameplay ranges.
4. **Gate normal startup on visual readiness.** Keep the loading presentation active until immediate content is fully represented and all content inside the configured view has an appropriate representation. Poll with bounded incremental work and report loading duration, per-kind candidate/represented/pending counts, source/view revision, queue depth, and coverage lag. Diagnostic fast boot remains explicitly excluded from gameplay acceptance.
5. **Sustain coverage during movement.** Reuse the existing streaming demand and directional view-priority paths to prepare content ahead of motion. Preserve responsive movement; if the system cannot guarantee a full-detail representation at the frontier, maintain a valid horizon/LOD representation rather than exposing a blank frontier.

## Acceptance

### Contract and loading

- Coordinator contracts prove that visual readiness is required for gameplay readiness, cannot mask terrain/structure/navigation failures, cannot borrow another request's receipt, and is invalidated by changed source or view revisions.
- Candidate accounting is conserved by kind: every described candidate is represented, explicitly pending, or failed. Incomplete discovery never reports an empty region as ready.
- Tree queue acceptance distinguishes queued/building from committed visuals. Far LOD/proxy receipts count only for visual representation and never for physical readiness.
- Run focused readiness/streaming contracts, the world-streaming loading matrix, and the streaming maturity journey.
- At the first controllable frame, immediate content is fully present; all content within the configured view has a suitable representation. No nearby tree, foliage, or wildlife appears later into a previously empty location.

### Traversal and visuals

- Run headed cold-start and continuous sprint/fast-turn playtests through ordinary gameplay flow.
- Inspect captures/video at first control, after a fast turn, and at a newly streamed area. Confirm no blank frontier and that detail is promoted before content becomes near-field.
- Do not fail a run merely because a playtest actor cannot traverse an entire cave or world region; that is separate fixture/navigation coverage unless the visual streaming behavior itself regresses.

### Performance

- Retain incremental publication and the game's existing frame-time limits; no unbounded synchronous startup or movement-frame work.
- Run representative normal-runtime performance coverage with traversal. Report startup duration, worst and p95 frame time, queue depth, coverage lag, and the precise evidence/report paths.
- Compare equivalent seed, view configuration, and cold/warm cache conditions. Distinguish initial playable readiness from whole-site/world completion.

## Constraints and non-goals

- Keep the current configured final view distance and near/far LOD ranges; verify their actual units and ownership before coding.
- Preserve deterministic generation, IDs, materials, density, and RNG order.
- Do not widen draw distance, add arbitrary foliage/tree placements, add a parallel generated-world authority, or treat physical publication as visual proof.
- Do not gate ordinary movement on distant full detail, collision, navigation, or simulation.
- Lighting polish and unrelated cave/navigation defects are outside this streaming acceptance unless they prevent a valid representation from being seen.

## Open implementation questions

- Which runtime owner can enumerate a complete deterministic visual candidate manifest without duplicating or perturbing the existing per-chunk generation/RNG path?
- What is the exact conversion from the terrain viewer's configured distance to the content manifest's world-space envelope, and how should FOV visibility versus all-direction turn coverage be represented?
- Which structure publication owner can provide a visual receipt distinct from its physical-only acknowledgement?
- Which far-tree/ecology representation is already supported by production, and where is a new stable visual-only proxy justified without masking missing generated content?

Resolve these questions with focused contracts and a small headed probe before expanding implementation. A synthetic readiness green is not live visual acceptance.
