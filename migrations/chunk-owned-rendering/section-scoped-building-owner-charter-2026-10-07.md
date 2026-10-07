# Section-scoped building owner publication

**Status:** next production cutover; design recorded before implementation.

This narrows building source identity and readiness to the world section that
owns each static geometry slice. A local visible section must not require the
complete roster of a distant wall, foundation, or landmark merely because the
same authored building contains both.

## Baseline and evidence

The game worktree is `codex/chunk-owned-world-rendering-migration` at
`934cbdbf75b75dd1b41ae596d3003173d398de8e`, with extensive pre-existing
migration edits and Godot import churn. Preserve them. The headed
`atlas-1492` Citadel r18 result passed 33/35 checks, but its exact owner closure
failed within the 32-frame bound. It installed 3 of 31 required owner sections;
the selected `castle_tower_01_foundation` itself occupies two sections. The
closure contained 9,529 retained per-member demands because the candidate
sections also contain full rosters for several castle wall, compound foundation,
and paving parts. These are real geometry owners, not false bounds. Evidence is
in the game worktree at
`artifacts/citadel-runtime-integration/citadel-nonempty-section-receipt-direct-mesh-r18/report.json`.

The same report measured 10.99 s in source-owner reconciliation. The synchronous
snapshot-scope change passed the focused 114-check Citadel service contract. The
completed headed r19 run used seed `atlas-1492`, ran 500.968 s, and passed 33/35
checks; the exact-owner closure and complete owner lifecycle checks failed. It
installed 3 of 31 required sections, leaving 30 visible demands pending and
9,529 retained owner demands. Cleanup passed and the owned-process census proved
zero remaining processes. A comparable early reconciliation sample fell to
about 0.39 s, but the full section closure remained too broad. That optimization
is bounded to one synchronous reconciliation; it does not justify a broad owner
dependency or cache authority across frames. The report is at
`artifacts/citadel-runtime-integration/citadel-nonempty-section-receipt-direct-mesh-r19/report.json`
in the game worktree.

## Authority and candidate identity

`CitadelPublicationService` and `CitadelSectionGeometryAdapter` remain the
authorities for generated plan membership, committed transform artifacts,
materials, mesh content, source revisions, removals, and attachment identity.
`WorldStaticSectionCandidateAssembler` remains the single section candidate
authority. Split each immutable committed static source into deterministic
section-owned slices using the actual mesh support intersecting each owner
section. The slice identity includes the stable parent building/member identity,
owner section key, producer incarnation, and source revision. Its digest covers
the complete sealed geometry and material recipe for that slice.

The per-section manifest lists every opaque/cutout/translucent slice, material
compatibility, mesh-content identity, intended visibility, exact bounds,
collision or declared physical dependencies, and source/tombstone revision.
Neighbor data may be read for smooth meshing and declared support, but it does
not make neighboring sections part of the local installation closure unless the
actual rendered or physical support crosses that boundary.

Parent building identity remains attached to gameplay, collision, doors,
interaction, navigation, and save records. This cutover changes static render
ownership only; it must not move those authorities into the renderer. Door
motion and other animated attachments keep their explicit attachment owner and
revision. Mobs and NPC simulation remain independent.

## Revision, retirement, and replay rules

Capture a complete immutable source snapshot before candidate admission. Include
the exact selected plan member set, publisher/job incarnation, committed
transform/artifact revisions, geometry and material fingerprints, and explicit
empty or removal rows. Derive slice rows from this snapshot without changing
generation order or gameplay RNG.

Reject a slice if its world, owner section, parent source, publisher/job
incarnation, geometry/material digest, removal revision, or candidate generation
is stale. Recheck currentness before commit and after the actual frame-drawn
callback. Cancellation, source invalidation, unload, and replay must retire only
the exact section slice and its retained aliases. Keep the old installed section
visible until the complete replacement packet and required owner receipt are
accepted. An unavailable or missing slice remains pending; it is never empty
success.

## Stages and exit evidence

1. **Slice contract:** deterministic partition of one real committed building
   artifact into exact section slices; completeness, bounds, digest, material,
   explicit-empty, stale revision, and parent identity checks. Prove that the
   tower foundation produces only its two actual owner slices.
2. **Producer/coordinator cutover:** emit slice identities from the existing
   Citadel producer, assemble them into the real section candidate, and replace
   global-roster leases with exact intersecting owner and declared support
   leases. Keep prior section slices until replacement receipts settle.
3. **Native renderer proof:** install the new slice through the existing native
   compile/upload/frame-ack path, reject mutation at each boundary, and prove
   replacement and unload/replay preserve the old valid packet until accepted.
4. **Headed Citadel acceptance:** repeat the same fresh seed and verify all
   demanded section receipts settle within the existing bound, with no visual
   gaps or regressions. Inspect the capture and owner-receipt trace.
5. **Gameplay closure:** verify real door motion, collision, interaction,
   navigation, save/reload, unload/reentry, and representative traversal. Keep
   the ordinary structure producer as a separate domain that uses the same
   section-slice contract; do not route it through the Citadel job.

The exit report must distinguish passed, failed, blocked, and untested rows.
Focused contracts and a native receipt are intermediate evidence; they do not
close the migration stage or overall goal.

## Minecraft 26.2 reference

`SectionCompiler` consumes a bounded section neighborhood, emits the section's
render-layer output, and `SectionRenderDispatcher` installs each section result
after compilation/upload. `RenderSectionRegion` supplies neighbor reads; it does
not make a whole structure the atomic compile unit. Apply that section ownership
and replacement lifecycle to our smooth meshes and real source authorities.
Do not copy Minecraft's block mesher, and do not turn a neighboring read into a
whole-landmark readiness dependency.
