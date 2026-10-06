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

## Integration evidence in progress

The coordinated producer, Adapter/Index, and tree consumer implementation is now
present. It retains publication aliases and consumer leases, removes the v1
intermediate seal, and keeps the existing native compile/install destination.
This is not stage-C acceptance.

- `node tools/run-project-compile-smoke.mjs` passed: MainMenu, Main, and the
  playtest runner load. Owned-process evidence:
  `artifacts/node-tools/process-runs/godot-ggVDIp/watchdog.json`.
- `node tools/run-ecology-producer-catalog-context-contract.mjs --OutputDirectory
  artifacts/citadel-runtime-integration/ecology-producer-catalog-context-owned-r4-20261006`
  passed 54 synthetic checks. This proves the named publication ownership/value
  contracts, not gameplay. The earlier r1/r2 compiler failures are preserved;
  their forced stops have authoritative zero-member evidence. R3 passed 53
  checks before the legitimate immutable `Rect2i` structure-bounds case was added.
- Real Main generation parity is not yet accepted. First run stopped at the
  spatial-source review guard because its already-reviewed compiler hash had
  not been updated. R2 then reached source admission and rejected the production
  structure dependency's `Rect2i` bounds. Both runs exited with cleanup and
  authoritative zero-member evidence. The owner whitelist now accepts that
  immutable integer rectangle; no resource/object admission was relaxed.
- Independent review identified a final local-revision freshness gap in support
  queries. Owner/lease checks alone are insufficient after terrain, structure,
  or removal changes. The query/acknowledgement checks are integrated and pass
  the focused Index gate; old installed visuals remain while replacement is pending.

The frozen-baseline parity command additionally compares exact 64-bit RNG state,
generated rows, actor digests, source/family revisions, and counts with the
passing family-unit r5 report. JavaScript comparison retains raw unsafe-integer
JSON literals rather than rounding them. Full serialized v2 bundle equality is
not claimed: transport lease data and the duplicate legacy `actorIntents` field
are omitted; canonical actor snapshot/digest and generated content are retained.

Remaining risk: large payload retirement still follows the existing synchronous
alias-release behavior. This unit has not established hitch-free teardown or
retirement. Live traversal, screenshots, interactions, save/reload, and runtime
performance remain untested against this coordinated cutover.

### Combined gate results

All paths below are relative to game `artifacts/citadel-runtime-integration/`.
Passing focused runs have exit 0, cleanup passed, and authoritative zero members.

| Gate | Report directory | Result |
|---|---|---|
| Producer catalog/publication contract | `ecology-producer-catalog-context-owned-r4-20261006` | 54 checks passed |
| Adapter | `ecology-section-value-adapter-owned-r2-20261006` | 88 checks passed |
| Support index | `ecology-world-support-index-owned-r2-20261006` | 68 checks passed, including local freshness and stale acknowledgement rejection |
| Tree family | `tree-source-family-coverage-owned-20261006` | 22 checks passed |
| Source owner discovery/replay | `ecology-source-owner-discovery-contract-owned-20261006` | 14 checks passed |
| Real Main generation service parity | `ecology-source-pass-slicing-parity-owned-r4-20261006` | 20 in-run checks passed; older-build digest comparison failed and remains unresolved |
| Headed tree compiler/adapter | `tree-recipe-section-compiler-owned-20261006` | 22/25 checks passed; fixture authority lacked newly required publication APIs; fixture migrated, rerun pending |
| Headed actual Main cohabitation | `main-section-cohabitation-gate-owned-20261006` | Failed initial readiness at 120.162 s; zero native compiles/installations |

The real Main command remains unchanged apart from the fresh output directory:
`node tools/visible-world/run-main-section-cohabitation-gate.mjs --OutputDirectory
artifacts/citadel-runtime-integration/main-section-cohabitation-gate-owned-20261006
--TimeoutSeconds 360 --StartupWaitSeconds 120`. Seed is
`ecology-main-retirement-stage5`, with `-SkipTutorial`. It completed with exit 1,
cleanup passed, and authoritative zero members. The loading gate stayed closed.
There were 61 ready source jobs, three pending, zero failed, 64 captured source
chunks, and 2,355 terrain-backed section demands. The three unfinished source
sessions were in `underground_props`; 60 narrow completed sessions retained their
detail cursor for possible wider demand. No renderer installation is proven.

Diagnostic totals: sealing 17.848 s, accepted-result retention 24.668 s, useful
generation 9.662 s, total capture 59.241 s; maximum queue service 3.235 s. These are
not controlled performance comparisons: approximately ten seconds overlapped an
isolated diagnostic-project materialization. The functional pending phases and
zero-install outcome remain valid evidence.

The baseline mismatch has not been waived. Actor digests, actor/source counts,
family counts, and exact RNG states match r5. The reviewed tree compiler source
hash contributes to the certified support policy and catalog identity, which
also appears in source rows. Reports retain only row hashes, so they cannot prove
unchanged transforms/recipes independently. A separate ignored source-service
replay using the retained prior compiler/queue and their original certificate
pin is being prepared; it must use the unchanged baseline comparison, and must
not be cited as production renderer or gameplay acceptance.

## Next coordinated cutover: preparation before complete-section admission

The user requested larger implementation steps. This unit therefore spans the
producer store, asynchronous recipe preparation, demand retirement, and final
candidate admission together. HEAD owns acceptance; subsystem leads have separate
mutable file scopes. The game branch remains at `5ffac014` plus the preserved
migration work; no reset, generated-import staging, or game push is authorized.

### Resolved prerequisite evidence

The headed tree compiler/adapter rerun
`tree-recipe-section-compiler-owned-r2-20261006` passed all 25 checks, nine batches,
and its real queue-to-adapter mapping. This is compiler integration evidence,
not ordinary-game visual acceptance.

The isolated prior-certificate replay passed all 20 source-service checks and the
unchanged exact baseline comparison. Command: `node
tools/run-ecology-source-pass-slicing-parity.mjs --ProjectPath artifacts/parity
--OutputDirectory artifacts/citadel-runtime-integration/ecology-source-pass-slicing-parity-r1
--TimeoutSeconds 180 --BaselineReport <absolute game path>/artifacts/citadel-runtime-integration/ecology-source-pass-slicing-parity-family-r5-20261006/report.json`.
The diagnostic copy uses current producer code and retained prior compiler/queue
plus their prior certificate pin. This isolates the earlier digest differences
to certified source identity: under the same certificate, row digests, family
revisions, actor digests, and exact RNG states match. It does not establish new
renderer installation or geometry correctness on its own. Both passing runs have
exit 0, cleanup passed, and authoritative zero members. Initial diagnostic launch
failed before process creation because the nested Windows log path was too long;
the copied project was moved to `artifacts/parity`, preserving the setup failure.

### Authority and complete data path

Main's deterministic source pass remains the generation authority. A Context
producer factory will own raw value capture, canonical sealing, the immutable
payload, its catalog hold, and indexed family/member views. The factory invokes
the Domain sealer once. External presealed admission retains full validation;
there is no caller-controlled trusted boolean or certificate shortcut. Payload
digests retain their current meaning. Owner, epoch, terrain, structure, removal,
family and source revisions remain binding before and after asynchronous work.

The adapter can submit an explicitly complete tree-family publication to the
existing recipe queue while other source captures remain pending. Each job is
owned by retained demand; shared demand survives one subscriber leaving, while
last-demand unload, supersession, or world reset detaches/cancels owned work and
releases aliases through the existing retirement owner. An obsolete completion
cannot replace a newer source job. Pending/backpressure retains retryable demand.

The final section candidate still includes the complete current terrain,
building, tree, and prop manifest and required render layers. Early preparation
does not permit partial installation. The existing native compiler/renderer must
acknowledge installed layers and required collision/interaction consumers before
the candidate is current. Old valid publication stays visible until replacement
succeeds. Mobs, source RNG order, durable save deltas and gameplay authority stay
independent of compilation scheduling.

Minecraft reference inspected again: `SectionCopy` copies section state once;
`RenderRegionCache` shares that immutable capture by section identity;
`SectionRenderDispatcher.checkSectionMesh` waits for every required vertex/index
upload before swapping and releasing the old mesh. Its cancellation boundaries
inform demand-owned retirement here. Our smooth terrain and procedural-tree
support bounds remain game-specific.

### Entry, proof, and exit

1. **Owned factory:** exact factory-versus-canonical output parity; one canonical
   seal on the producer path; strict external rejection of forged/stale/object
   inputs; independent catalog holds; alias, reset and local revision checks.
2. **Independent preparation and retirement:** with another family blocked,
   admitted trees prepare while final section readiness remains pending; shared
   demand, last-unload cancellation, backpressure, empty-family proof and stale
   completion rejection have focused evidence. No readiness bounds or timeouts
   are relaxed.
3. **Production admission:** repeat the same actual Main gate, require current
   complete native installation and owner acknowledgements, then inspect live
   visuals/traversal and performance. A failed Main gate keeps this unit open.

Underground applicability requires additional authority evidence: a player above
ground does not by itself prove that cave openings or nearby underground props
are invisible. No blanket family omission or fabricated empty result is allowed.
Section-local generation must preserve deterministic candidate ordering/RNG;
otherwise accelerate preparation at the terrain-volume authority. This is an
explicit remaining architectural dependency, not an excuse to bypass admission.
