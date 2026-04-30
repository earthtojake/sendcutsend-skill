<div align="center">

<img src="assets/sendcutsend-skill-demo.gif" alt="Demo of the SendCutSend skill producing a preflight report from a CAD upload file" width="100%">

<br>

</div>

# SendCutSend Skill

SendCutSend preflight review for DXF and STEP/STP files.

The SendCutSend skill reviews exact upload files against current SendCutSend ordering, catalog, and engineering-spec sources. It is built for conservative manufacturing-readiness reports: direct file inspection, cited source fields, explicit order-context assumptions, and clear pass/warning/fail/need-more-info findings.

## What It Can Do

- Refresh current SendCutSend ordering guide, catalog, and engineering specs.
- Inspect DXF files for laser sheet-cutting upload risks.
- Inspect STEP/STP files for CNC routing or 3D model upload risks.
- Check material, thickness, service, finish, hardware, bending, tapping, countersinking, and order-context compatibility.
- Compare measured geometry facts only against cited current source facts.
- Produce structured preflight reports with next edits and source bibliography.

## Commands

Run commands from this skill directory when refreshing official source files:

```bash
python scripts/download_sources.py
python scripts/download_sources.py --skip-cache
```

Install the inspection dependency when the active Python environment does not already provide it:

```bash
python -m pip install -r requirements.txt
```

The source downloader writes generated references under `references/generated/`. Those refreshed files are local cache data and are ignored by git.

For agent-facing workflow rules, use [SKILL.md](SKILL.md). Report structure and source-selection details live in [references/](references/).
