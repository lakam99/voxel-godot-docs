# Family-scoped source generation for complete render sections

## Outcome and baseline

Deliver a complete nearby section to the production native renderer without
generating every content family in every possible tree-owning chunk. This is one
integrated producer/census/renderer change. Preserve deterministic recipes, IDs,
RNG draws, removals, gameplay collision and interaction, save authority, and old
installed visuals until replacement acknowledgement. Mobs remain independent.
Do not reduce certified support, make deferred work empty, relax readiness,
increase frame budgets to mask the dependency problem, or change tree appearance.

Branch: `codex/chunk-owned-world-rendering-migration`, working tree based on
`ff39dd5f`, with substantial pre-existing migration work and import churn. Preserve
both. The prior forward-bounds Main gate failed readiness at 120.033 seconds:
4 complete source chunks, 60 pending, zero support failures and zero native
installs. See [exact evidence](section-capture-completion-charter-2026-10-06.md).

The certified savanna-tree envelope is approximately 115 m horizontally; the
21.6 m render section therefore intersects possible origins in 64 stream chunks
at this alignment. This is a conservative one-hop inverse census, not recursive
dependency expansion. It must remain complete. The error in work organization
is requiring each of those owners to finish every detail pass and a 96-level
underground scan before any tree-only source can be accepted.

## Reference and authority

Minecraft 26.2 `RenderSectionRegion` captures already-generated section data;
`SectionCompiler` consumes that region's blocks into complete render layers;
`SectionRenderDispatcher` bounds compilation and swaps after upload completion.
Apply the separation between generation, immutable region capture, compilation
and installation. Our smooth terrain and long procedural trees require their
own certified support closure rather than Minecraft's fixed 3x3x3 neighborhood.

The existing deterministic world/terrain, structure admission, ecological
profiles, producer recipes and removal projection remain source authorities.
Add family coverage to their immutable source results; do not establish another
generator or recover source truth from render nodes.

## Shared contract

- An admitted family capture identifies world/epoch, source chunk, source-domain
  revision, catalog lease, removal projection, family, and family policy revision.
  Use an explicit versioned family request/certificate at producer boundaries.
- Section census derives a separate possible-owner set for each family using
  that family's certified support. A tree-only distant owner does not imply
  detail, forage, ore or underground coverage. No family is excluded just because
  it is visually inconvenient; its certified support must not intersect the
  requested section, or a current family result must prove no contributors.
- Immutable family outputs distinguish complete nonempty, explicit complete
  empty, deferred/unrequested, pending dependency, and failed. Missing coverage
  never becomes empty. A section requires all families applicable to it.
- Preserve surface attempt ordering and all existing surface/wildlife/compatibility
  RNG draws. Shared surface decisions may be cached once per source revision;
  retain/materialize only demanded family outputs. Details and underground retain
  separate cursors and completion receipts. Do not resample or reorder generation
  when a second section or family is requested later.
- Cache and scheduler identities include family coverage. A narrow result cannot
  satisfy a wider request. Deduplicate shared family demand; cancellation must
  preserve remaining subscribers. Catalog/terrain/structure/removal changes
  invalidate affected results and reject stale workers and acknowledgements.
- Union accepted family contributors into the existing complete section manifest
  and render layers. Keep native compile/install, collision/interaction owners,
  complete receipt checks and old-publication retention intact.

## Execution and evidence

HEAD owns integration, canonical docs, serialized test launches and acceptance.
Producer lead owns the family contract in `EcologyProducerDomain`, Main capture
sessions and generation entry points. Adapter lead owns `EcologySectionValueAdapter`
and `EcologyWorldSupportIndex`, after the shared request/result shape is frozen.
No overlapping edits. Existing owner-consumer verification is completed separately.

Producer scope also includes `EcologyProducerCatalogContext`: a complete sealed
catalog artifact may describe explicit pending family policies. Its admission
must still prove immutable input completeness, ownership, lease and content
integrity. Only the aggregate policy-ready requirement moves to requested-family
and section-census validation. Missing catalog data never gains admission through
this change. Add focused partial-policy and stale-owner coverage.

Tree compilation lead owns `TreePublicationQueue` and `TreeRecipeSectionCompiler`:
require explicit completed trees-family coverage before treating missing rows as
empty, bind tree-family identity through compile/poll, and revalidate authoritative
empty results too. A details-only bundle is never an empty-tree certificate.
Coordinate shared validation helpers with the producer lead. HEAD independently
reviews the compiler change before updating its reviewed spatial-source digest;
no geometry or recipe change is intended. Native C++ consumes the assembled section
candidate, not these ecology snapshot schemas, and needs no change for this gate.

First establish and review the shared request/result schema, then integrate Main
and adapter/index against it as one production cutover. Contract fixtures must
prove family-specific inverse closure, narrow-to-wide requests, deterministic
combined-vs-sliced output, explicit empty vs deferred, invalidation/removal,
shared subscribers and cancellation. Preserve renderer candidate completeness.
Then require actual native installation in the same-seed Main gate, inspect live
visuals, and continue traversal/edit/harvest/unload/reload/performance acceptance.
Neither new snapshot contracts nor reduced pending counts complete this stage.

Entry condition: forward geometry contract and compile pass; retain the failed
Main baseline and unresolved older fixture rows. Exit requires a current complete
production candidate installed through the real renderer, followed by named live
and performance evidence. If a dependency is still unknown, document it before
changing the authority. Do not relaunch an unchanged expensive gate.

## Approved shared schema

`capture_ecology_source_domain` gains an optional final family request argument;
omitting it explicitly requests all six families through the same generator.
`ecology-source-family-request/v1` contains canonical sorted `requestedFamilies`
and a digest bound to source-domain/catalog/epoch/removal/family-policy identities.
The base `sourceRevision` is invariant across request subsets.

The request digest excludes the incidental catalog lease token and response
status/reason. The token remains separately validated against the current owner;
acquiring another lease for identical demand must not change work identity or
create a circular dependency between lease ownership and request identity. Tree
compile identity uses the base revision and the completed trees-family revision
and manifest, so widening another family's demand reuses unchanged tree work.

`ecology-source-domain-family-bundle/v2` retains existing provenance and contains
`requestedFamilies`, an entry for every family in `familyCoverage`, the union of
completed requested `sourceRows`, and a separate surface-pass/actor-intent receipt.
Each `ecology-source-family-result/v1` entry identifies its family, policy revision
and digest, family revision, disposition, member count, manifest digest and rows.
Dispositions are complete nonempty, complete empty, deferred unrequested, pending
dependency, or failed. A ready bundle requires every requested family complete.
`categoriesComplete` during the transition is exactly the complete requested set.
Never mutate a previously published bundle when a request widens.

Unrequested or pending family entries publish no rows. Shared surface decisions
and generated-but-unrequested rows stay in the private generation session. This
keeps each sealed bundle independently reconstructible from its published inputs
and prevents private progress from changing a previously accepted manifest.

Actor simulation readiness does not gate static installation. However, the
current shared surface RNG tape includes wildlife presentation branch draws.
`build_wildlife_actor_intent` can fail before scale/animation/direction/timer
draws when its immutable presentation descriptor is unavailable. That remains
an explicit generation-input failure in this unit: accepting later static rows
would otherwise change their deterministic RNG sequence. Do not swallow that
failure as an optional actor sidecar. Separating this dependency later requires
a complete known draw branch and retained retryable actor decisions.

Census carries `sourceChunkKeysByFamily` with all six canonical keys and exact
sorted owner sets, plus a sorted union for compatibility. Unknown family support
keeps the relevant census pending; it cannot become an empty set certificate.
Capture can advance an independently bounded requested family while another
family policy remains pending. Shared surface RNG decisions remain one retained
pass; later family demand reuses those decisions rather than resampling.

## Integrated verification checkpoint

The user requested larger implementation steps. The delivery unit remains the
complete generation, family dependency scheduling, tree compilation, and actual
Main renderer installation path. Individual contract passes are diagnostic
evidence within that unit; they are not stage exits.

On game base `5ffac014` with the current uncommitted family cutover, the engine
compile smoke passed after two GDScript syntax/type corrections. The catalog
contract passed 45 checks and tree-family contract passed 19. Reports are under
`artifacts/citadel-runtime-integration/`:

- `ecology-producer-catalog-context-family-20261006/report.json` — passed,
  synthetic catalog ownership and scope contract.
- `tree-source-family-coverage-20261006/report.json` — passed, synthetic family
  admission and authoritative-empty currentness contract.
- `ecology-section-value-adapter-family-20261006/report.json` — failed; replacement,
  cancellation, and refreshed admission assertions require integrated diagnosis.
  Functional exit 1; clean shutdown and authoritative zero members.
- `ecology-world-support-index-family-20261006/report.json` — failed at
  “new family owner remains queryable and releases old leases.”
- `ecology-source-pass-slicing-parity-family-20261006/report.json` — setup could
  not find the required naturally generated static/tree/actor case in its bounded
  search. This is unresolved service-fixture evidence, not a gameplay failure or
  proof of changed generation.

The failed same-seed Main baseline remains unchanged. Do not rerun that expensive
gate until the narrower evidence supports a new attempt. Minecraft `ChunkStep`
and `SectionRenderDispatcher` were reviewed again: completion follows task
completion, and all required nonempty layer uploads precede replacement of the
old section mesh. Preserve those boundaries through the family optimization.

The integrated lifecycle review identified two production corrections: the
latest-subscription lookup must bind world/chunk/family selection independently
of the versioned work digest, and an equal-content catalog owner replacement
must refresh validated source/posting provenance without creating a false
removal. Revisioned job identity and all semantic content checks remain intact.
For partial family publication under replacement owner B, remaining A-family
proof must keep the candidate pending until recaptured under B. The previous
installed representation remains visible during that transition. Do not admit a
mixed stale-owner candidate to avoid a pending state.

The second parity diagnostic isolated the setup omission: all 96 candidates
failed `citadel_town_inputs_unfinalized`, with zero source attempts. The fixture
must generate the required deterministic town-region inputs and use the real
Main finalization command before comparing source-pass budgets.

### Focused cutover results

The coordinated correction now passes these named gates. All six runs below
reported functional exit 0, cleanup passed, and authoritative zero owned members.
Their `launch.json` files bind the uncommitted source hashes to the reports.

| Gate | Checks | Report under `artifacts/citadel-runtime-integration/` |
| --- | ---: | --- |
| Catalog ownership | 45 | `ecology-producer-catalog-context-family-20261006/report.json` |
| Tree family admission | 19 | `tree-source-family-coverage-20261006/report.json` |
| Support index lifecycle | 60 | `ecology-world-support-index-family-r3-20261006/report.json` |
| Adapter/candidate input contract | 88 | `ecology-section-value-adapter-family-r3-20261006/report.json` |
| Real Main source-pass parity | 19 | `ecology-source-pass-slicing-parity-family-r5-20261006/report.json` |
| Source relocation/removal replay contract | 14 | `ecology-source-owner-discovery-contract-family-r3-20261006/report.json` |

Use `node tools/run-ecology-producer-catalog-context-contract.mjs`,
`node tools/visible-world/run-tree-source-family-coverage-contract.mjs`,
`node tools/visible-world/run-ecology-world-support-index-contract.mjs`,
`node tools/run-ecology-section-value-adapter-contract.mjs`,
`node tools/run-ecology-source-pass-slicing-parity.mjs`, and
`node tools/visible-world/run-ecology-source-owner-discovery-contract.mjs`,
respectively, with `--OutputDirectory` equal to the report's directory.
The parity run used `--TimeoutSeconds 180` and the default recorded seed
`ecology-source-pass-slicing-parity-v1`.

The parity gate exposed a production detail handoff bug: a time-budget yield
after completed detail batch publication but before caller phase advancement
could append the completed detail rows again. The producer now performs that
handoff once per session. Public sliced capture equals the full pass, preserves
actor decisions and RNG states, and has unique source/member identities.
Unchanged family publications now leave section revisions stable, while actual
changes invalidate their exact dependents. Final family demand release drains
the catalog lease. The source relocation fixture also corrected its own
world-versus-chunk-local position setup; no production policy was relaxed.

These are contract/service results. Actual Main native installation and later
live visual, traversal, interactions, unload/reload, save, and performance gates
remain required. No stage exit or overall completion is claimed.

The headed recipe compiler contract also passed 25 checks and produced 9 batches:
`node tools/visible-world/run-tree-recipe-section-compiler-contract.mjs --OutputDirectory artifacts/citadel-runtime-integration/tree-recipe-section-compiler-family-20261006`.
Its owned-process cleanup and zero-member proof passed. It is compiler/queue
integration evidence, not live gameplay acceptance.

### Same-seed Main result and next architectural boundary

`node tools/visible-world/run-main-section-cohabitation-gate.mjs --OutputDirectory artifacts/citadel-runtime-integration/main-section-cohabitation-gate-family-20261006 --TimeoutSeconds 360 --StartupWaitSeconds 120`
failed startup readiness at 121.531 seconds, with tutorial skipped and seed
`ecology-main-retirement-stage5`. Functional exit 1; cleanup passed and authoritative
zero members. Report and source/DLL hashes are in that exact output directory.

Compared with the prior 4 ready/60 pending baseline, the new run reached 60 ready,
12 pending, 0 failed source jobs. However, it still had zero completed native
compiles or installed candidates. This does not pass the stage exit.

The final timings isolate the next boundary: 60 final seals consumed 21.434 s
(maximum 915.937 ms); accepted-result retention consumed 20.757 s (maximum
789.453 ms); useful generation consumed 7.468 s. The maximum source-queue service
step was 4288.367 ms. Main seals the same completed payload twice solely to rotate
its transport catalog lease; consumers then reconstruct it again during family
and member validation. The next integrated unit must establish owner-issued
immutable source publication once, reuse exact admitted family/member aliases,
and preserve current owner/revision checks at capture and installation. Do not
increase deadlines or frame budgets to conceal this work.
