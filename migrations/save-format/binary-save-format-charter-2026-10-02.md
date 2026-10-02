# Binary save format migration charter — 2026-10-02

## Outcome and scope

Write gameplay save slots as Godot binary Variant data instead of JSON. Preserve the v2 save envelope and gameplay snapshot semantics. Remove old JSON save files as explicitly authorized by the user; do not migrate or load them. Keep human-readable diagnostics and runner reports in JSON.

## Authority and affected paths

`SaveSystem.gd` owns serialization, slot paths, active-seed selection, deletion, and save-file cleanup. `MainSaveState.gd` remains the snapshot authority. Continue/menu startup and tooling that pairs, validates, copies, or deletes save slots must use the binary slot contract. Terrain remains generated from the world seed plus durable terrain edits; render objects are not part of this migration.

## Baseline and workspace

- Game repository: branch `master`, revision `e6838c4e`.
- The game worktree already contains unrelated dirty world-streaming/loading changes; preserve them.
- Documentation repository has pre-existing edits in the two world-streaming migration documents; preserve them.
- Existing save encoding is JSON in `SaveSystem.gd`; slots use `*_slot_<seed>.json`; the active-seed marker is separate text.
- No save-format test baseline was run before implementation.

## Design decisions

- Keep `SAVE_VERSION = 2`; serialization changes while the save data schema stays the same.
- Store each slot as `*_slot_<seed>.bin`, using Godot's built-in Variant binary encoding without object deserialization.
- Keep the active-seed marker as small text metadata.
- On startup, delete the old JSON monolithic save and JSON slot files. Reject malformed or wrong-version binary slots.
- Do not persist live Godot mesh/scene objects in the gameplay save.

## Risks and dependencies

- Binary Variant values must round-trip every value present in the full gameplay snapshot, including packed arrays and terrain-volume data.
- Runtime/tooling that inspects saves must stop parsing slots as JSON; integrity receipts can hash/copy opaque binary slots, while save semantics stay verified through `SaveSystem`.
- Existing saves are intentionally lost. This is user-authorized.
- Autosave remains a full snapshot write; binary encoding removes JSON text overhead but does not make snapshot capture or disk I/O free.

## Acceptance evidence

- Save and load preserve the v2 envelope and nested terrain/gameplay values through binary encoding.
- Slot creation, active-seed selection, async save, deletion, and invalid/wrong-version rejection operate on `.bin` slots.
- Startup removes legacy `.json` save files and does not treat them as valid saves.
- Continue tooling pairs and copies the opaque binary slot and active seed byte-for-byte.
- Update source-local save-format references and inspect the final diff. Report unrun verification separately from passed evidence.

## Stages

1. Inventory callers and establish the save ownership/path contract.
2. Implement binary slots and JSON cleanup in `SaveSystem`.
3. Migrate production tooling and current save contracts to opaque binary slots.
4. Review final changes and record exact verification status and limitations.

## Implementation status

- Stages 1–4 have been implemented in the game worktree at `master` revision `e6838c4e` plus the current local changes.
- Production save paths now use `.bin` slots with `VBW2` framing and Godot Variant binary encoding. The active-seed sidecar stays text. Existing v2 JSON monolithic and slot files are deleted at startup.
- Continue and loading-matrix tools verify the binary header and file integrity, then rely on the headed Continue flow for gameplay restoration evidence.
- Current save contract and tooling fixtures were updated for binary slots.
- No Godot or Node test/playtest runner was launched in this implementation pass. The change is not runtime-verified and must not be treated as migration acceptance until the focused save contract and a headed save/Continue flow pass.
- `git diff --check` reported no whitespace errors; it printed the repository's existing LF-to-CRLF working-copy notices.
