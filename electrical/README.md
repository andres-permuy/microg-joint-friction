# Microgravity electrical design

Open **[Microgravity.kicad_pro](Microgravity.kicad_pro)** for the current design.

**Start with [AI_HANDOFF_2026-11-18.md](AI_HANDOFF_2026-11-18.md)** before modifying these files. It contains the design rationale, source traceability, file map, generation internals, validation limits, change procedures, and a continuation prompt for another AI.

- [DESIGN_NOTES.md](DESIGN_NOTES.md): circuit explanation, calculations, assumptions, and hardware acceptance work.
- [Microgravity_diagram.pdf](Microgravity_diagram.pdf): seven-page document. Page 1 is the complete native A0 circuit; pages 2–7 are reference snapshots.
- [Microgravity_interconnect.pdf](Microgravity_interconnect.pdf): page-1-only export.
- [BOM.csv](BOM.csv) and [WIRING.csv](WIRING.csv): component and connection tables.
- [handoff_state_2026-11-18.json](handoff_state_2026-11-18.json): file hashes, counts and fresh verification evidence at handoff.

The circuit targets **Pittman 9236S009-SP + RoboClaw 2x7A / IMC404**. It has 95 counted component instances, 59 connected networks and 15 intentional NC networks. Fresh ERC at handoff: 0 errors, 0 warnings. The PCB is empty; hardware qualification and firmware commissioning remain pending.

Page 1 is editable. The six linked detail pages contain images, not duplicate electrical circuits. The editable detailed source project is separately preserved under [reference/detailed_revision_B/](reference/detailed_revision_B/).

**Do not execute/import the generation scripts merely to inspect the project.** They overwrite files. `build_interconnect.py` also depends on a fixed archived XML baseline; editing only the detailed generator can leave the full-page circuit stale. Read handoff sections 8 and 12 before regenerating or changing electrical behavior.
