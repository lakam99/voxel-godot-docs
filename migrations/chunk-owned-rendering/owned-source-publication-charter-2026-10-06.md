# Owned source publication cutover

## Outcome, baseline, and boundaries

Deliver the complete production section candidate through native compilation and
installation without repeatedly rebuilding immutable source proofs on Main.
This is one coordinated producer-to-consumer cutover, as requested by the user.
Keep smooth terrain, deterministic generation/RNG, gameplay collision,
interactions, save deltas, independent actors, and complete family coverage.
Do not change support bounds, readiness rules, deadlines, or frame budgets.
Do not add another world generator or renderer. The native C++ section compiler
and installation backend remain the production rendering destination.

Game: `codex/chunk-owned-world-rendering-migration`, HEAD `5ffac014`, current
worktree `outputs/voxel-biome-world-godot`. Substantial uncommitted migration work
and unrelated import churn predate this unit and must be preserved. Canonical
documentation is this separate repository. Prior unit evidence is recorded in
[the family cutover charter](family-scoped-source-generation-charter-2026-10-06.md).

Entry evidence: seven focused gates pass, including real Main generation parity.
The same-seed headed Main gate still fails readiness at 121.531 s: 60 ready source
jobs, 12 pending, 0 failed, and 0 native installs. Source sealing consumes 21.434 s,
retention 20.757 s, useful generation 7.468 s. Main seals a bundle twice solely to
rotate a transport lease; family/member consumers repeatedly reconstruct it.
All test processes are drained with authoritative zero-member evidence.

## Architectural decision and reference

Minecraft 26.2 `SectionCopy`, `RenderRegionCache`, `RenderSectionRegion`,
`SectionCompiler`, and `SectionRenderDispatcher` establish the reference:
capture owned immutable input, compile against that input, upload required layers,
then swap the accepted result. Do not copy its block mesher or fixed-neighborhood
assumption into our smooth terrain and procedural tree support closure.

The production source publication owns one canonical, deeply immutable value
payload and precomputed family/member lookup tables. It is derived from existing
source authorities. It is an ownership boundary, not a second source of truth.
Acquire its durable catalog lease before the final seal. Separate semantic
payload identity from transport leases; acquiring another consumer lease does
not rebuild source rows or alter family revisions.

The coordinated Main API is fixed before consumer edits:

```text
admit_ecology_source_publication(snapshot, catalog_lease_token, owner_kind, owner_key)
acquire_ecology_source_publication(publication_id, owner_kind, owner_key)
resolve_ecology_source_publication(lease_token, expected_world_id, expected_world_epoch)
ecology_source_publication_is_current(lease_token, expected_world_id,
  expected_world_epoch, expected_owner_receipt, expected_source_revision,
  expected_removed_projection_digest)
ecology_source_publication_local_is_current(view, lease_token)
ecology_source_publication_record_is_current(view, lease_token, record)
release_ecology_source_publication(lease_token)
```

Successful acquire/admit responses include `publicationId`, `leaseToken`, and
the exact retained `view`. The `ecology-source-publication-view/v1` view contains
the exact payload alias, owner receipt, content/source identity, frozen normalized
`familyResultsById`, `memberIndex`, precomputed member digests, support policy,
and family policies. Family entries preserve the established ready/pending/
failed result shape and direct row aliases. Resolve/acquire never reseal content.
Owner-only currentness does not replace local authority currentness: the local
facade checks fresh terrain, structure, removal, and catalog inputs at task and
installation boundaries. Per-record checks use retained exact aliases and the
member index. Independently supplied values may undergo fresh full admission,
but a copied wrapper/row cannot impersonate an existing publication proof.

Admission must prove owned value data, complete source identity, and exact
payload alias. An artifact ID, matching digest, shallow read-only container, or
an equal-looking copied Dictionary is insufficient. Follow the existing catalog
publication pattern: exact owner-issued payload identity plus current owner
receipt. Keep the canonical validating path for independently supplied values.
No unbounded global memo table and no metadata-only admission shortcut.

One validated publication view provides direct family slices and indexed
`(family, sourceId, sourcePartId)` membership. Deep semantic validation happens at
admission; consumers check the exact retained publication, its own live lease,
world/epoch, source revision and current authority receipt. Source changes,
replacement Main/catalog owners, terrain/structure/removal changes, and stale
worker completion reject the prior proof. Recheck these boundaries before
candidate publication and installation. Unchanged content alone never proves
that an earlier installation is current.

Narrow and wide requests retain separate immutable publications and stable shared
family revisions. Private generation cursors remain shared by the existing
source session. No actor/RNG, recipe, material, geometry, support, or save identity
changes are intended. Deferred content remains pending, never empty success.

## Complete path and lifecycle

1. Main captures current generation, catalog, terrain, structure, and removal
   inputs; its retained deterministic source session produces demanded families.
2. The producer constructs and seals one immutable family publication, obtains
   a durable owner receipt, and retains its exact aliases and membership index.
3. Adapter capture jobs acquire publication leases and reuse the view across
   families and member preparation. Index admission and tree compilation use the
   same accepted view; no per-row full-bundle reseal or transport-token deep copy.
4. SupportIndex retains required family coverage and unacknowledged tombstones.
   Tree queue retains publication ownership through async compilation and result
   consumers. Only owned value buffers cross native worker boundaries; Main
   references, Nodes, Resources, WeakRefs, Callables, and RIDs do not.
5. The existing complete section manifest/layers flow to native compile/install.
   The previous valid representation remains installed until current candidate
   identity, layers, frame/native receipt, and provider acknowledgments succeed.
6. Cancellation, reset, unload, owner replacement, and final receipt retirement
   release the exact consumer leases. Bounded cache eviction cannot invalidate
   actively retained demand. Do not move final large-payload destruction onto an
   unbudgeted gameplay frame; reuse existing retirement ownership.

## Stages, ownership, and evidence

HEAD owns architecture, integration, serialized Godot runs, final diff review,
documentation, explicit staging, and all acceptance decisions.

- Producer lead owns Main source capture, `EcologyProducerDomain`, and a composed
  publication owner if needed. Define exact shared API signatures with consumer
  lead before implementation; preserve existing snapshot schemas where possible.
- Consumer lead owns Adapter and SupportIndex integration and lease handoffs.
- Tree/review lead owns TreePublicationQueue/compiler integration, coordinated
  fixture updates, and independent review of producer/consumer changes. HEAD
  independently reviews this lead's production diff.
- No overlapping mutable scopes. No concurrent Godot tests or edits to a running
  gate's dependencies. Preserve all substantive fixture assertions.

**Stage A — coordinated implementation:** one seal and owner publication,
consumer view/lease adoption, currentness and retirement integrated together.
Entry: shared API fixed and dependency owners known. Proof: engine compilation
and exact final diff review; this alone does not advance gameplay acceptance.

**Stage B — focused falsifiers:** prove single sealing/admission despite multiple
family/member consumers; immutable alias integrity; forged/copied values rejected;
catalog/Main replacement, terrain/structure/removal revision changes, empty and
deferred families, cancellation, narrow/wide reuse, and complete lease drainage.
Retain real source-pass row/RNG/actor parity and all prior adapter/index/recipe
checks. Bounded telemetry reports seal/full-validation counts and time, alias
checks, publication/lease counts. Do not infer success from lower work counts.

**Stage C — actual renderer and live acceptance:** repeat the exact same-seed
Main gate and compare source completion and queue service timings. Require a
current complete production candidate compiled and installed through the native
renderer with settled provider acknowledgments. Then run live visuals/traversal,
interaction/edit/harvest, unload/replay/save, and representative performance gates
from the controlling migration matrix. Inspect screenshots and traces. Report
failed and untested rows separately; a focused proof cannot waive this exit.

Commit coherent verified scope with explicit file staging and retain known failed
reports. Publish canonical documentation to main. No game push is implied.
