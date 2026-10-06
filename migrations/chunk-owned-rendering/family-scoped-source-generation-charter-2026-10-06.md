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

`ecology-source-domain-family-bundle/v2` retains existing provenance and contains
`requestedFamilies`, an entry for every family in `familyCoverage`, the union of
completed requested `sourceRows`, and a separate surface-pass/actor-intent receipt.
Each `ecology-source-family-result/v1` entry identifies its family, policy revision
and digest, family revision, disposition, member count, manifest digest and rows.
Dispositions are complete nonempty, complete empty, deferred unrequested, pending
dependency, or failed. A ready bundle requires every requested family complete.
`categoriesComplete` during the transition is exactly the complete requested set.
Never mutate a previously published bundle when a request widens.

Census carries `sourceChunkKeysByFamily` with all six canonical keys and exact
sorted owner sets, plus a sorted union for compatibility. Unknown family support
keeps the relevant census pending; it cannot become an empty set certificate.
Capture can advance an independently bounded requested family while another
family policy remains pending. Shared surface RNG decisions remain one retained
pass; later family demand reuses those decisions rather than resampling.
