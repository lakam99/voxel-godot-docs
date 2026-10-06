# Complete section capture before expanding active work

## Outcome and baseline

Make the real section compiler receive complete nearby terrain/building/ecology
inputs, then install them through the native renderer. Preserve the cozy game,
smooth terrain, deterministic source identities and seeded draws. Do not alter
generation, loading requirements, rendering radius, saves or NPC navigation.

Game branch is `codex/chunk-owned-world-rendering-migration`, base `ff39dd5f`,
with substantial pre-existing migration edits and generated import churn.
Preserve that work. Canonical preceding evidence is
[native compile cutover](native-compile-cutover-2026-10-06.md).
The latest same-seed Main gate at 120.068 seconds had 196 pending source jobs,
zero ready jobs/native installs, one catalog composition and 930 reuses.
Its clean owned-process exit proves this is a production scheduling/capture gap,
not lingering animation invalidation. Treat startup as failed, not accepted.

## Reference and authority

Minecraft 26.2 `SectionCompiler`, `RenderSectionRegion`, `SectionCopy` and
`SectionRenderDispatcher.runTask` separate immutable captured inputs from mutable
task-local work. A bounded buffer admits a task, it completes compilation, then
uploads and replaces the prior representation. Adapt those ownership and
scheduling boundaries; do not copy its mesher or impose its fixed neighborhood
on our genuine tree/building support closure.

Our flow remains generation/terrain/structure/removal authority -> sealed source
inputs and catalog lease -> source capture cursor -> immutable source manifest ->
section contributor census -> native compile -> current-census install -> provider
acknowledgement. Mobs remain actors outside the static section candidate.

## Integrated implementation boundaries

1. The adapter retains all demand and exact section-to-source dependencies.
   Schedule a bounded number of section cohorts, deduplicating overlapping source
   jobs. Service their complete closures across frames before admitting further
   background cohorts. Distance/urgency chooses useful work; explicit age promotion
   and bounded service opportunities prevent starvation. Do not replace this with
   nearest-only scheduling or a larger per-frame budget. Dependency stalls yield
   admission opportunities without dropping their pending requests. Unload,
   supersession and reset retire all scheduler associations and leases.
   Deferred jobs may retain lightweight immutable identity/subscription/lease
   records so multi-section census queries keep their existing all-or-pending
   semantics. The bounded admission applies to active producer work; report queued
   records separately. Do not return a partial census as complete.
2. Main owns mutable capture sessions privately. Retain RNG/cursors and work
   buffers across slices rather than serializing, freezing and reconstructing
   the complete pass each time. Only immutable accepted results cross producer
   boundaries. Session identity includes world/epoch, source revision, catalog
   lease and removal projection. Cancellation/reset drains sessions through the
   existing owner. Preserve existing per-slice local currentness until an owner
   receipt proves a cheaper replacement; a caller digest is insufficient.
3. Capture-phase telemetry distinguishes local input validation, cache lookup,
   useful generation, state retention and final sealing. Scheduler telemetry
   distinguishes demanded, active and deferred source work, cohort completion and
   wait age. Keep reporting bounded.
4. Initialize the existing tree publication service before section capture needs
   it, removing the dependency on an incidental first live tree spawn. Preserve
   its established reset/teardown lifecycle. Do not introduce another service.

## Delegation and gates

HEAD owns design, canonical docs, test launches, integration and acceptance.
Capture lead owns `MainCore.gd`, `MainPlaytestTools.gd`, `MainSetupScene.gd`,
`EcologyProducerDomain.gd` and dedicated capture-session contracts.
Scheduler lead owns `EcologySectionValueAdapter.gd` and its contract. No overlapping
production edits. Reviewer inspects exact completed changes without implementing
them. Freeze sources before HEAD launches tests.

First prove deterministic output parity across slicing, cancellation and owner
replacement; bounded active closure with eventual far-demand service; shared
dependencies, invalidation, reset and cleanup. Then compile and run the same-seed
Main gate. Require actual completed sources/native candidates and retain all
existing readiness assertions. A focused pass or first native receipt is only
stage evidence. Complete live visual/traversal, edits/harvest, unload/replay,
save/reload and representative performance gates remain parent exits.

If measurements show capture admission dominates rather than task-state copying,
repair the measured owner boundary before another unchanged expensive run. Do not
add synchronous fallbacks, invented empty results or timeout success paths.

## Stage evidence and coordinate-contract correction

The integrated project compiles (`godot-FDxm2V` owned-process run). The session
contract passes 14 synthetic lifecycle checks at
`artifacts/citadel-runtime-integration/ecology-source-capture-session-retained-20261006`.
The adapter contract passes 85 checks at
`ecology-section-value-adapter-cohorts-r3-20261006`. Both have functional exit 0,
clean cleanup and authoritative zero process membership. These do not establish
live acceptance or complete generated-output parity across slicing.

The same-seed, tutorial-skipped Main run
`main-section-cohabitation-gate-capture-cohorts-20261006` reached source finalization
and was stopped after terminal support-proof failures. At the last inspected
41.844-second checkpoint, 12 private sessions had been created, 957 resumed,
5 failed and 4 cancelled; none had sealed successfully or reached native install.
The finalization error was `ecology_source_member_exceeds_certified_support_bound`.
That label also covers missing/pending support proof; it is not proof of genuinely
oversized assets. Failures include rocks, forage and detail sources in chunks
(-4,-3), (-3,-4), (-3,-3), (-3,-2). The run's stop has cleanupPassed=false,
functionalExitCode=null and authoritativeZeroProven=true with no members remaining.

Bounded phase observations over 969 calls: useful generation 3,466,987 us total
(41,834 us maximum slice), local input capture 340,990 us total (575 us max),
session retention 11,978 us total (44 us max), final sealing 515,537 us over five
attempts (229,682 us max). Repeated local validation was not the dominant measured
cost in this run. Preserve those checks. No claim about steady gameplay frames.

HEAD isolated a coordinate direction error: Godot's `AABB * Transform3D` applies
the inverse transform, while `Transform3D * AABB` applies the forward transform.
See the [Godot 4.6 operator contract](https://docs.godotengine.org/en/4.6/classes/class_transform3d.html#class-transform3d-operator-mul-aabb).
The owned engine probe `bounds-direction-20261006` passes with clean cleanup and
zero membership: translation (12,3,-9) maps a local box to expected position
(11.5,3,-9.5) only with the forward expression. This is mathematical service
evidence. Main source proof, detail value production and adapter validation contain
inverse expressions where their declared contract is forward local-to-parent/world.

Before another production run, audit and correct this coordinate contract across
the affected producer-to-renderer path as one unit. Capture lead owns Main and
detail value builder; scheduler lead owns adapter consumers after its lifecycle
fix freezes; HEAD owns shared partition/compiler integration if the audit finds
additional consumers. Reviewer independently checks the map. Preserve intentional
world-to-local inverses. Do not blanket replace operators, widen support envelopes,
change source positions/RNG or weaken proof checks. Verify translated, rotated,
scaled, negative-chunk and boundary-crossing cases against independently transformed
vertices/corners, then actual generated source slicing and the same Main gate.
Correct synthetic expected bounds that duplicated the wrong formula without
relaxing substantive containment, identity or renderer assertions.
