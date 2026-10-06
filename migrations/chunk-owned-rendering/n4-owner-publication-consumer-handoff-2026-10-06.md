# N4 owner publication consumer migration

## Contract

The owner catalog publication remains the only catalog authority. `ActiveSurfacePropOwnerBundle` retains sealed biome, visual and animated publication aliases through its owner, chunk and terrain wrappers. Deep copies are not valid owner proofs. Removal, structure and terrain snapshots retain their existing value validation.

`AnimatedAssetRegistry` seals the historical N4 presentation projection at publication time. Its five-key ABI, legacy row fields and JSON SHA-256 domain remain unchanged. Capture returns that sealed projection; currentness requires that exact projection and a current owner publication. Failed setup restores the previous projection together with its publication and invalidation state. Actor-private animations remain independent.

`ActiveSurfacePropOwnerBundle.native_admission_projection(main, bundle)` validates the original bundle before projecting the exact historical native biome and visual schemas and owner revisions. `native_biome_projection` serves the standalone biome adapter. Keep the original bundle for freshness: a native admission receipt does not establish live source currentness. Native C++ schema and checks are unchanged.

All direct GDScript callers of the three N4 catalog admission methods were inspected. They are focused N4 contracts/probes. The direct-source and underground probes now explicitly project their retained original owner bundle. Synthetic native variants (duplicate family membership, stale GLB hash, missing optional fields and missing wildlife entry) mutate detached projected input and recompute its native ABI hash; they are not represented as live registry mutations.

## Scope

- `scripts/visual/AnimatedAssetRegistry.gd`
- `scripts/world/ActiveSurfacePropOwnerBundle.gd`
- `scripts/testing/native_world/AnimatedAssetPresentationCaptureContract.gd`
- `scripts/testing/native_world/ActiveSurfacePropOwnerBundleContract.gd`
- `scripts/testing/native_world/N4BiomeCatalogAdapterContract.gd`
- `scripts/testing/native_world/N4VisualCatalogAdapterContract.gd`
- `scripts/testing/native_world/N4WildlifePresentationAdapterContract.gd`
- `scripts/testing/native_world/N4DirectSourceOrderProbe.gd`
- `scripts/testing/native_world/N4UndergroundPropSourceProbe.gd`

Pre-existing registry changes and descriptor contract belong to the surrounding owner-publication stage; do not stage unrelated working-tree files. No native C++, Main, Adapter, biome owner or visual owner changes are part of this consumer fix.

## Verification handoff

No Godot runs were launched by this implementation lead. HEAD owns independent review and sequential verification after the geometry gate. `git diff --check` passed for the edited scope. This is implementation-ready for review, not verified completion.

First rerun the existing descriptor contract:

```text
node tools/run-animated-asset-scene-state-descriptor-contract.mjs -OutputDirectory artifacts/citadel-runtime-integration/animated-owner-native-projection-20261006
```

Run each focused script below using the existing watchdog, with a fresh output prefix per run:

```text
node tools/run-godot-scene-watchdog.mjs --executable C:/Users/arkam/Desktop/Godot_v4.6.1-stable_win64.exe/Godot_v4.6.1-stable_win64_console.exe --timeout-seconds 120 --summary-path artifacts/citadel-runtime-integration/<unique>-watchdog.json --stdout-path artifacts/citadel-runtime-integration/<unique>-stdout.log --stderr-path artifacts/citadel-runtime-integration/<unique>-stderr.log -- --headless --path . --script res://scripts/testing/native_world/<fixture>.gd
```

Fixtures, in order: `AnimatedAssetPresentationCaptureContract`, `N4BiomeCatalogAdapterContract`, `N4VisualCatalogAdapterContract`, `N4WildlifePresentationAdapterContract`, `ActiveSurfacePropOwnerBundleContract`. Their result files remain under `artifacts/native-world-backend/` with their existing `n4-*-contract.json` names. Preserve each report with the run's source hashes and require functional success separately from authoritative zero-member cleanup.

The direct/underground probes require their existing seed/location configuration and source-evidence runners; their two-line admission migration is not evidence that ordered generation passed. These focused checks do not prove live gameplay, complete section rendering or performance.

## Detached-map consumer closure

The subsequent closure audit found three remaining fixture writes through detached compatibility maps. `N4DirectSourceOrderProbe` and `N4RockRuntimeBoundsOracle` now use `disable_asset_for_test` and `clear_test_disabled_assets`. The generated-rock member check in `CanopyAssetImportContractRunner` seeds private owner storage on a separate synthetic registry, rather than modifying the imported catalog; its evidence explicitly says synthetic render-policy contract.

The proposed owner-catalog commit also requires `TreeRuntimeRequestBuilder.gd`, new `TreeRequestAdmission.gd` and its `.uid`, and `CanopyRuntimeContractRunner.gd`: the registry now supplies the builder's new catalog-snapshot parameter. The admission helper uses the existing `EcologyProducerDomain.derive_tree_request_envelope` contract. Include any fixture helper and `.uid` referenced by the exact staged test set, such as `CertifiedTreeRequestFixture`, when applicable. `TreeSpawnService` is not needed merely to resolve the new builder call signature; its broader admission enforcement belongs to its own verified scope.

Additional verification commands (not run by the implementation lead):

```text
node tools/run-canopy-asset-import-contract-tests.mjs
node tools/run-canopy-runtime-contract-tests.mjs
node tools/run-n4-direct-source-order-differential.mjs
node tools/run-godot-scene-watchdog.mjs --executable C:/Users/arkam/Desktop/Godot_v4.6.1-stable_win64.exe/Godot_v4.6.1-stable_win64_console.exe --timeout-seconds 120 --summary-path artifacts/citadel-runtime-integration/n4-rock-owner-api-watchdog.json --stdout-path artifacts/citadel-runtime-integration/n4-rock-owner-api-stdout.log --stderr-path artifacts/citadel-runtime-integration/n4-rock-owner-api-stderr.log -- --headless --path . --script res://scripts/testing/native_world/N4RockRuntimeBoundsOracle.gd
```

These three additional fixture edits are frozen pending HEAD review and verification. They do not modify native code or production geometry.

## HEAD verification

HEAD reviewed the projection and bundle changes, then ran the descriptor runner
(19 checks passed) and all five listed watchdog fixtures sequentially. All exited
0 with clean cleanup and authoritative zero process membership. Presentation
capture passed 29 checks and nested owner bundles passed 59 checks; all three
native catalog adapter reports passed. Copies are retained under
`artifacts/citadel-runtime-integration/n4-owner-projection-<lowercase fixture>-20261006/report.json`.
Descriptor evidence is in `animated-asset-scene-state-descriptor-native-projection-20261006`.
Full project compilation also passed (`godot-0XFtLP`). The two ordered-generation
probes were not rerun; their API projection edits do not establish generation
parity.

The owner unit is committed as game `5ffac014` (`Publish owned catalogs and
preserve native consumer admission`), with 23 explicitly staged files. It includes
the required tree request builder/admission dependency and migrated consumer
fixtures. No generated import churn was staged and no game push was performed.

Final canopy import run passed 14 checks, and the biome catalog runner passed 18.
The rock constructor oracle requires a real renderer: its headless run produced
the intentional generated-asset proxies and failed its mesh-count assertion;
the subsequent headed service run passed all seven imported/fallback cases at
`n4-rock-owner-projection-headed-20261006`, with clean cleanup and zero membership.
This is not a gameplay acceptance claim. Canopy runtime still fails its literal
source-scan removal-gate assertion: the actual removedProps save/restore check
passes, but its expected code substring no longer matches the production path.
The fixture and failure were preserved; this unit does not claim the full canopy
suite or gameplay passed. Its standalone constructor cases also log missing
ecology-ledger metadata, an unresolved fixture setup limitation.
