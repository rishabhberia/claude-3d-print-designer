# Project layout and pipeline

```
<project>/
  README.md            what it is; decisions with dates; measured vs assumed sizes; plate table
  .venv/               see setup.md
  reference/           photos, traced outlines, measurements (third-party models stay local)
  cad/
    params.py          every dimension: the single source of truth
    lib.py             helpers: boxes, cylinders, insert/screw cutters, deboss text
    parts.py           each part as a CadQuery solid in ASSEMBLED coordinates
    build.py           Part(pid, name, solid, print_rotation, plate_hint, colour) list
    check.py           collisions, validity, bed fit, project rules
    overhang.py        overhang + bridge audit in print orientation
    export.py          dedupe identical parts, write STL/3MF meshes + MANIFEST.csv
    plates.py          pack into the Bambu multi-plate project
    slice_report.py    real time and grams per plate
    render.py          true-to-print images
    export_web.py      GLB + step table for the web page
  web/index.html       3D viewer + "Build it step by step"
  renders/
  out/plates/<NAME>.3mf   rebuilt from code (can be git-ignored)
  docs/changes/        one short note per meaningful change
```

Run order: `build.py` → `check.py` → `overhang.py` → `export.py` → `plates.py` →
`slice_report.py` → `render.py` → `export_web.py`.

## What each step must do

- **params.py:** drawings, solids and checks import the same values, so they can't disagree.
  Every value that came from a measurement says so in a comment (who, when).
- **parts.py:** parts are built where they sit in the assembly. The print rotation is applied
  only at export. That makes collision checks and the assembly animation free.
- **build.py:** every part has an ID (`A-1`, `JS-2`…), a print rotation and a colour.
- **check.py** (each rule with a negative control):
  1. pairwise interference between all parts (volume of intersection > a small tolerance);
  2. each part is one valid solid;
  3. bed fit in print orientation (bed − 5 mm border);
  4. hardware rules: clearances around magnets/inserts, screw backing, screw path traced from
     open air to the insert, head and tool room;
  5. moving parts: sweep the full motion range and check clearance at each step;
  6. project-specific rules (e.g. a weighing tray touches only the load cell).
- **overhang.py:** per part, down-facing area > 45° off the bed and bridge spans; compare all
  six axis-aligned orientations; print a table.
- **export.py:** dedupe by **face fingerprint in print orientation** (volume alone merges mirror
  images). Mesh-only geometry (e.g. a textured skin from a reference model) keeps a CadQuery
  proxy for the checks and is written only if it is watertight and its extents match the proxy
  within 0.5 mm.
- **plates.py:** see bambu-3mf.md. Order plates deliberately: structure first, precious visible
  parts last, the riskiest joint early. Keep a `PRINTED = {...}` set; printed plates are never
  repacked (`--keep-layout`), and the script proves their meshes are byte-identical.
- **render.py:** render what will print: real solids, seams at true width. An orthographic
  painter's algorithm with PIL or matplotlib is enough for exploded views and sections, but it
  can sort long thin faces wrongly; cross-check important views.

## Useful CadQuery facts

- `Workplane("XZ")`: local y = global Z and the normal is −Y; extrude negative to go +Y.
- Rotating (x, y, z) → (x, z, −y) is Rx(−90). Swapping y and z is a **reflection** (mirrors
  the part).
- A loft starting on a cylinder's end circle (coincident, tangent faces) can fail booleans;
  start it slightly inside.
- `trimesh` `section(...).to_2D()` returns points in its own rotated frame; transform back.
