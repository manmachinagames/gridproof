# Gridproof — agent guide

Logic puzzle compendium for Steam, Godot 4.7 (GDScript), fully playable offline. Part of the Proof series.

Not started yet. Design: `docs/design.md` (read it first). Until this guide is written out:

- Tileproof's `CLAUDE.md` (`manmachinagames/tileproof`) is the template: same module boundaries, determinism rules, localisation rules and code style. Shared code comes from `manmachinagames/proof_core`; CI checks it out as in Mineproof's workflow.
- The series plan is in `manmachinagames/proof_core`, `docs/proof_series_structure.md`.
- The defining rule of this game: every puzzle a player sees has exactly one solution and is solvable by named techniques without guessing. Both are proven by tests for every generator, technique or pool change.
