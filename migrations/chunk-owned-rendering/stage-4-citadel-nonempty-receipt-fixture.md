# Stage 4 Citadel Non-Empty Receipt Fixture

**Status:** one live non-empty blueprint section now reaches native receipt and visual retirement; Stage 4 remains partial
**Recorded:** 2026-10-05
**Canonical migration charter:** [Section-Owned World Rendering](section-owned-world-rendering-charter.md)

## Scope and acceptance

Prove one deterministic, non-empty blueprint-building section through the real
`CitadelPublicationService`: fresh site generation and immutable plan/member
census; sealed producer transform-artifact contribution; complete shared candidate assembly; live
native section installation; then source-revision and owner-bound visual
retirement acknowledgement. The previous building visual remains visible
while any affected section is pending or stale. Collision, doors, furnishing
interactions, and navigation remain with their existing gameplay owners.

This fixture does not claim normal-game startup, multi-site coverage, save/load
parity, gameplay collision or navigation acceptance, forest/static-prop
cutover, or performance. It does not use or regenerate from the historical
`actual-site-source-05/result.bin` artifact.

## Baseline before this fixture

- Game worktree: `codex/chunk-owned-world-rendering-migration`, HEAD `af5d0e32`.
- The worktree is already dirty. `CitadelPublicationService.gd` contains the
  in-progress receipt/visual-retirement implementation; the native install
  session has concurrent fluid work. Preserve both scopes and all generated
  import churn; this fixture must not edit coordinator, ecology, or fluid files.
- The prior headed runner
  `node tools/visible-world/run-citadel-section-receipt-retirement.mjs
  -OutputDirectory artifacts/citadel-runtime-integration/citadel-section-receipt-retirement-headed-20261004i`
  passed 12/12 checks and proved native installation, stale-coverage and
  owner-recreation receipt rejection, and visual-only retirement using
  synthetic Citadel membership/publisher values. Its actual service branch
  used only explicit empty coverage. It did not prove non-empty real Citadel
  membership, packet contribution, or actual publisher visuals.
- The frozen input required by existing actual packet contracts,
  `artifacts/citadel-runtime-integration/actual-site-source-05/result.bin`, is
  absent. The new fixture must prepare a deterministic fresh source through
  the current real service/plan path and report that source identity and exact
  producer/receipt evidence.
- Documentation `main` already has unrelated in-progress edits in the visible
  world readiness plan and implementation. Preserve those files.

## Stage exit evidence

Report the exact seed/site/source generation and member/section IDs; census and
contribution revisions; immutable candidate manifest; native receipt and live
backend/chunk identity; publisher visual state before, while pending, and after
acknowledgement; collision/door/navigation owner identity before and after;
and stale-source plus owner-recreation negatives. Use the owned headed runner
and retain its report and cleanup proof. If the actual producer cannot admit a
non-empty section, record the concrete pending dependency and leave this stage
open.

## First headed production run — 2026-10-05 r1

Command:

```text
node tools/visible-world/run-citadel-nonempty-section-receipt.mjs --outputdirectory artifacts/citadel-runtime-integration/citadel-nonempty-section-receipt-20261005-r1
```

The fresh deterministic source and immutable plan were prepared from seed
`atlas-1492`. The complete census described 12 building members in section
`(199,1,-342)`. The only failed check was
`actual_plan_member_transform_artifact_contribution_ready`: the first member,
`castle_compound_foundation_segment_00`, returned retryable
`static_transform_artifact_roster_unavailable`. No section candidate, native
receipt, or visual acknowledgement was reached. Report:
`artifacts/citadel-runtime-integration/citadel-nonempty-section-receipt-20261005-r1/report.json`.

At terminal state the service had 15/3,596 physical groups complete, 3,581
deferred, and no foreground packet requests. The owned watchdog exited 1 without
timeout or forced cleanup, passed cleanup, and proved zero job members. This
does not pass the fixture or Stage 4. Next, identify why the demanded producer
group lacks a sealed transform-artifact roster; inspect the prepared-segment
branch and whether the retained section demand reaches packet dispatch. Do not
rerun unchanged or increase the time budget.

## Producer follow-up and r2–r4 findings — 2026-10-05

The prepared-segment focused contract passed 14/14 after adding direct parity
checks against the original segment's buffer, bounds, and instance count. It is
synthetic producer evidence only.

Headed r2 again failed the contribution check. Instrumented r3 proved all 15
explicitly requested groups had physical receipts, but its aggregate rejection
reason did not identify the missing producer state. R4 added bounded
source/group attribution: all 27 rejected groups had `sourcePartId` set to
`<unknown-source>` and empty source suffixes in their batch keys. Their reason
was `transform_group_identity_or_payload_incomplete`; no static transform
roster was committed. Report:
`artifacts/citadel-runtime-integration/citadel-nonempty-section-receipt-20261005-r4/report.json`.

Code tracing explains this identity loss in the resumable producer path:
`publish_static_part` sets the source identity and owner/chunk context, starts a
pending masonry/paving/roof job, and restores the previous context. A later
`publish_part_batch` advances that job without restoring its part's context.
The delayed segment collectors therefore emit blank source/owner metadata and
the transform flush rejects them. The next change must bind the complete
source/owner context around each pending-job advance and restore the caller's
context afterwards. This does not change collision, doors, navigation, or
interaction ownership. No native section candidate, installation, or visual
retirement acknowledgment was reached; Stage 4 remains open.

Reports:

- `artifacts/citadel-runtime-integration/citadel-nonempty-section-receipt-20261005-r2/report.json`
- `artifacts/citadel-runtime-integration/citadel-nonempty-section-receipt-20261005-r3/report.json`
- `artifacts/citadel-runtime-integration/citadel-nonempty-section-receipt-20261005-r4/report.json`
- `artifacts/citadel-runtime-integration/building-transform-artifact-prepared-segments-20261005-r6/report.json`

## Pending-job context repair and headed r5–r6 — 2026-10-05

`BuildingPartPublisher.publish_part_batch` now advances a resumable part job
inside that job's exact source identity, source revision, logical owner cell,
render chunk, and render tier, restoring the caller's prior context after each
bounded slice. A missing part or parent fails closed. The producer contract
uses the real resumable masonry job and verifies the prepared batch's exact
identity on every emitted slice, caller-context restoration, matching sealed
artifact revision, and preserved collision intent. Command:

```text
node tools/run-building-static-section-transform-artifact-contract.mjs --outputdirectory artifacts/citadel-runtime-integration/building-transform-artifact-pending-context-20261005-r4
```

It passed 21 checks, including all seven delayed-resumable-job checks; the
watchdog exited 0, cleanup passed, and zero owned processes were proven. This
is producer-level evidence and does not establish live integration.

The first headed rerun, r5, stopped before its final report because fixture
diagnostics called `Object.get` with a Dictionary-style default argument.
Godot reported the exact site as
`CitadelNonemptySectionReceiptFixture.gd:648`. Its runner requested owned-job
termination; all job members reached zero, but cleanup is correctly marked
failed after forced cleanup. This is a fixture reporting defect, not evidence
of an installed candidate. The fixture diagnostics now read declared publisher
properties directly and use the publisher's declared collision count. The
narrow repair received an independent code review GO.

The next headed run passed:

```text
node tools/visible-world/run-citadel-nonempty-section-receipt.mjs --outputdirectory artifacts/citadel-runtime-integration/citadel-nonempty-section-receipt-20261005-r6
```

The fresh deterministic site came from seed `atlas-1492`, region `(1,-3)`.
The fixture selected the real plan member
`building:castle_tower_01_floor` in section `(199,1,-342)`. All 23 checks
passed: the live service produced the immutable publication plan; the selected
non-empty section received its actual source transform artifacts; native
section installs and real service receipts were accepted; existing building
visuals remained visible until the first receipt and were then retired for the
selected closure; and collision plus door-registration owners remained intact.
The source was generated through the current admission path, with no frozen
fixture. Report:
`artifacts/citadel-runtime-integration/citadel-nonempty-section-receipt-20261005-r6/report.json`.
The watchdog recorded functional exit 0, no timeout or forced cleanup, cleanup
passed, and authoritative zero-member proof.

An independent evidence audit confirmed the launch hashes match the fixture,
publisher, flush, coordinator, and native backend. The target contributed two
transform-artifact groups; the production candidate moved from queued to
installed and its 12-source manifest contains the exact target revision. The
target legacy visual remained visible after queueing, then received a per-source
`retired` acknowledgement and became hidden. The earlier failing
`castle_compound_foundation_segment_00` is present in the same contribution and
candidate manifest, but this run does not select or retire it: its visual
acknowledgement remains pending on neighboring section receipts. Aggregate
provider acknowledgement also remains pending because 27 other visual rows are
waiting on neighboring receipts. The report proves this selected source's
single-section retirement; it does not prove complete provider readiness or
pixel-level screenshot correctness.

This closes the specific non-empty blueprint-section/native-receipt proof that
was missing in r1–r4. It does not cut over ordinary generated structures,
prove full-Citadel or multi-section retirement, normal startup, live player
traversal/interaction, save/reload, broad performance, or remove the old
production renderer for all construction. Stage 4 therefore remains partial;
Stages 5 and 6 remain open. Independent report review returned GO for this
specific receipt and retirement claim with the limits recorded above.

## Ordinary generated-structure lane audit — 2026-10-05

The separate ordinary town/standalone path has a source census, section
partition, coordinator contribution/install, and post-receipt visual-retirement
path, but is not covered by the Citadel r6 result. `OrdinaryStructureVisualSourceCapture`
records stable town/standalone IDs, cell/type/recipe digest, world and regional
revisions, and tombstones. `OrdinaryStructureStaticSectionProvider` partitions
supported geometry by actual section bounds; the coordinator recaptures and
rejects stale census/receipt work before provider acknowledgement. Collision,
interaction and durable removal remain owned by the generated `StaticBody3D`,
`StructureSystem`, and saved removal tombstones.

Coverage remains deliberately partial. The ordinary recipe path currently
supports only `cobblestonePath`, `stoneBlock`, and `woodBlock`; unsupported
members stay pending. Capture depends on resident bodies and their visual
children, so recipe-only compilation while unloaded and unload/replay are not
proved. The next safe step is an explicit classification contract for every
ordinary `blockType`: `section_static`, `separate_dynamic` with its named owner,
or `unknown` that remains pending. Only after this inventory identifies a
common opaque static family should its node-free immutable recipe be added;
filtering unknown or dynamic members would falsely permit empty coverage.
