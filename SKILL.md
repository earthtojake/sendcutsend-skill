---
name: sendcutsend
description: Review DXF and STEP/STP files against SendCutSend ordering specs. Use when producing SendCutSend preflight reports for upload readiness, material/service compatibility, manufacturability risks, SKU-specific tolerances, bending, tapping, countersinking, hardware insertion, finishing, or order-context gaps.
---

# SendCutSend

Use this skill to produce conservative, evidence-backed SendCutSend preflight reports for DXF and STEP/STP files.

Treat SendCutSend's ordering guide, catalog JSON, and specs JSON as evidence feeds, not stable APIs. Field names, types, and coverage may vary. Do not turn missing, unparsable, `N/A`, or conflicting source data into a pass or fail. Use scripts only to fetch source files or measure specific file facts; write the final report from explicit comparisons.

## Setup

From this skill directory, install the inspection dependency before geometry inspection if `build123d` is missing:

```bash
python -m pip install -r requirements.txt
```

Use `build123d.import_step` for STEP/STP inspection and `build123d.ezdxf` for DXF inspection. Do not use raw text parsing or alternate geometry backends for geometry facts.

## Source Refresh

Before each review, download or refresh the three SendCutSend source files:

```bash
python scripts/download_sources.py
```

The downloader writes `references/generated/sendcutsend-ordering-guide.md`, `references/generated/sendcutsend-catalog.json`, `references/generated/sendcutsend-specs.json`, and `references/generated/sources-manifest.json`. The cache is fresh for 24 hours. If the user explicitly asks to skip cache, force a fresh fetch:

```bash
python scripts/download_sources.py --skip-cache
```

Use the manifest's `fetched_at`, `cache_expires_at`, `sha256`, and JSON `_meta` values in the source bibliography when helpful. If a source cannot be fetched and only stale files are available, report that limitation and avoid ready verdicts for dependent checks.

## Workflow

1. Collect order intent.
   - Prefer DXF for laser sheet cutting and 2D sheet profiles.
   - Prefer STEP/STP for CNC routing and 3D model upload workflows.
   - Record file type, intended process, material/SKU, thickness, quantity, services, finish, and hardware.
   - If order context is missing or ambiguous, inspect enough source data to present concrete options, then ask the user to confirm before writing a readiness verdict. Include candidate SKUs/materials/thicknesses/services with relevant source links; include `photo_url` images and `learn_more_url` links from the specs JSON when available.
2. Read `references/official-sources.md`, run the source downloader, then inspect the generated source files directly. Normalize source facts defensively: parse numeric strings, size strings, `N/A`, missing fields, mixed types, and absent service arrays into explicit notes.
3. Inspect the exact upload file with targeted Python and `build123d`. Do not inspect only the source generator, CAD model, or generator console summary.
   - DXF: measure units, bounds, layers, entity types, open/duplicate geometry, unsupported annotations, candidate holes/circles, linework stats, bend-line candidates, and bend-to-cut distances as needed for the selected service.
   - STEP/STP: measure parseability, units hints, solid/surface signals, bounding box, shell/body signals, and limitations as needed for the selected service.
   - Keep each inspection script fact-only. It may report measurements, parse errors, and limitations, but it must not emit pass/fail/readiness statuses.
4. Select source records by evidence quality.
   - Use exact SKU as the only authoritative catalog/spec join.
   - If only material and thickness are provided, use a selected material only when the candidate match is unique and exact enough; otherwise list candidates with links/images from the source records and ask the user to choose.
   - Use the catalog JSON for orderability: stock, cutting process, available services, size limits, hardware, and finishes.
   - Use the specs JSON for engineering values: tolerances, holes, bridges, bending, tapping, countersinking, hardware insertion, finishing, and material properties.
   - Use the ordering guide for plain-language workflow and general file-format rules.

## Comparison

Compare only trustworthy pairs of evidence.

- Determine whether a check applies.
- Cite the source field path or guide section.
- Cite the measured file fact.
- Compare only when both the source requirement and measured file fact are available and trustworthy.
- If a needed measurement is missing or risky, write a small targeted `build123d`/`ezdxf` inspector for that specific geometry fact.
- For 2D files with bend lines, check flange length locally along every bend line. Measure the nearest cut/free edge on both sides of each bend at each span or sample point, including notches, slots, gaps, split tabs, and cutouts that interrupt the bend span or create a local free edge. Compare the minimum local flange depth to the selected SKU's `bending_specs.min_flange_length_before_bend` and `bending_specs.min_flange_length_after_bend`. Do not apply flange-length limits to ordinary enclosed holes or interior cutouts unless a cited source gives a hole-to-bend or feature-to-bend rule for that service; report those separately with centerline-to-bend or edge-to-bend measurements as the cited rule requires. Do not treat nearby bend-adjacent cut geometry as only corner relief unless the remaining local flange still passes the flange-length minimum. If any local flange depth is below the SKU minimum, report `❌ fail`. Do not rely on aggregate source-level values when exported geometry has local cutouts, interrupted bends, split bend segments, reliefs, tabs, or unsupported regions.

Report with restrained status labels:

- `✅ pass`: the measured file fact satisfies the cited current requirement.
- `⚠️ warning`: likely manufacturability or upload risk, but not necessarily a blocker.
- `❌ fail`: a direct measured violation of a cited current requirement.
- `❓ need more info`: missing context, missing source evidence, unmeasured geometry, source conflicts, or tool limitations.

## Diagnostic Images

When findings would be easier to understand visually, produce a concise diagnostic diagram proactively if image-generation or image-editing capabilities are available. Do this without waiting for the user to ask whenever there is a `❌ fail`, a spatially ambiguous `⚠️ warning`, or a geometry edit that needs a before/after explanation. If image-generation tools are unavailable, state that limitation and describe the intended diagram in the report.

Before generating an image, run a layout preflight:

- Choose the smallest set of callouts needed to explain the fix.
- Estimate whether labels will crowd the geometry, overlap each other, or run outside the canvas. If crowding is likely, flag it before generation and switch to numbered markers plus a side legend, a larger canvas, or separate detail views.
- Keep long measured values and rule text in the legend, not directly over dense geometry.
- Include the measured failing distance, the cited minimum, and the proposed movement or clearance target.

After generating an image, inspect the rendered image before delivery. If labels overlap, are clipped, are hard to read, or obscure the geometry, regenerate or revise the diagram before reporting it.

## DXF Review

For laser sheet cutting, start from the refreshed sources and measured DXF geometry facts.

Check for:

- single, uploadable DXF file with model geometry at 1:1 scale
- units and overall part size
- closed cut profiles where the service requires closed contours
- duplicate or overlapping cut geometry
- unsupported annotation, text, dimensions, images, construction lines, or hidden instruction layers
- layer/color/linework conventions from the ordering guide and upload workflow
- bend-line entities, bend segment lengths, split/common-axis bends, local flange depth on both sides of each bend line, nearest non-bend cut edge or cutout distances, and cut geometry touching or crossing bend lines when bending is in scope
- minimum holes, slots, web widths, interior geometry, part density, nesting, and spacing only when both source facts and measured file facts support a comparison
- secondary-service requirements for bending, tapping, countersinking, hardware, finishing, or deburring when requested

## STEP Review

For CNC routing or 3D model upload, start from the refreshed sources and measured STEP geometry facts.

Check for:

- STEP/STP file readability and a solid body rather than loose curves or surfaces
- units, scale, bounding box, thickness, and feature dimensions
- sharp inside corners, small holes/slots, thin walls, islands, deep pockets, tool access, and tolerances only when the file inspection can measure the fact
- whether geometry represents a sheet profile better served as DXF for laser cutting
- material, thickness, finish, and secondary-service compatibility

## Reporting

Include the file path, assumed service, material/order context, source files checked with access date, inspected geometry facts, findings ordered by severity, and specific next edits. In the findings table, include a `Rule source` column with Markdown links to the source URL plus the specific JSON field path or guide section used for that row. If a row is based only on direct file inspection and has no external rule, say `Direct file inspection`; do not leave the source blank. Do not call a file "SendCutSend ready" unless every required cited check either passes or is explicitly outside the selected service.

Use `references/report-template.md` when a structured report would help.

## References

- Official source selection: `references/official-sources.md`
- Report shape: `references/report-template.md`
