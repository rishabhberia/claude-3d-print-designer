---
name: 3d-print-design
description: Design physical objects for FDM 3D printing, from idea to a sliced, ready-to-print Bambu Studio project. Parametric CadQuery CAD, automatic checks (collisions, overhangs, bed fit, screw fit), true-to-print renders, multi-plate .3mf generation with real slicer time and grams, embedded-magnet pauses, and an interactive 3D web page with a step-by-step assembly guide. Use when the user wants to design, model, modify or fix something to 3D print, make an STL/3MF, lay out print plates, slice from code, or set up this workflow for the first time.
---

# 3D print design

Turn an idea into printed parts: measured inputs → parametric CAD → checks → renders the user
chooses from → a Bambu Studio project laid out on plates and sliced for real numbers → a 3D
web page showing how it goes together → print → learn.

The CAD software is **CadQuery** (free, open source, Python, built on the OpenCascade kernel),
with **trimesh** and **manifold3d** for mesh work. The slicer is **Bambu Studio**. Everything
is code, so a design change is one edit in `params.py` and a rebuild.

## Before anything else

1. **Find the knowledge base.** Look for a `3d-printing-knowledge/memory.md` path in the
   user's CLAUDE.md. If there is one, read `memory.md` now, then the files it points to that
   matter for this task.
2. **No knowledge base yet → run first-time setup** from [references/setup.md](references/setup.md):
   check the tools, ask the user for their printer, nozzles, filament and slicer version,
   create the knowledge base from `templates/knowledge-base/`, and add the pointer to the
   user's CLAUDE.md. Don't design anything until the setup check passes or the user has
   accepted what is missing.
3. **Existing project?** Read its `README.md` and `projects/<name>.md` in the knowledge base.
   Ask what has been printed since last time and how it came out before changing anything.

## Non-negotiable rules

- **Never guess a dimension.** List every size the design depends on, ask the user to measure
  it with calipers, and keep a "measured vs assumed" table in the project README. An assumed
  value is labelled `assumed - CHECK` and stays on the list until measured.
- **Real numbers only.** Print time and grams come from the slicer. Any estimate is labelled
  as an estimate. Missing data is reported as missing, never as zero.
- **Every check gets a negative control.** Before reporting "all clear", show that the check
  fails on a deliberately broken case (oversized part, screw over air, part moved into
  another). Several checks have "passed" while blind.
- **Check that outputs exist.** A CLI that exits 0 may have done nothing.
- **Python packages go into the project `.venv` only.** Never into a system or base Anaconda
  environment.
- **The user decides from renders of what will really print** (seams, gaps, joins at true
  size), with options side by side. No idealised pictures, no test kits unless they ask:
  order the real plates so an early one proves the riskiest joint.
- **Once a plate is printed, lock it.** Printed parts must stay byte-identical after every
  rebuild; new parts go on new or unprinted plates.
- Record user decisions with the date in the project README. Don't re-ask decided questions.

## The workflow

1. **Brief.** What it does, where it sits, what it holds or fixes to, colours, hardware the
   user already owns, which nozzle(s) they have.
2. **Measure.** Send the measurement list. Photos on a 10 mm cutting mat are fine for
   outlines, not for sizes (paper sheets read ~5 % off). Anything asymmetric gets a
   handedness check against a photo: mirror-image parts are an easy, costly mistake.
3. **Options.** Render 2–3 options side by side, including downsides. Wait for the choice.
4. **Build.** Project layout and scripts: [references/project-layout.md](references/project-layout.md).
   Apply every rule in [references/design-rules.md](references/design-rules.md):
   finished face up, no supports, debossed part IDs, sealed magnets, fittable screws.
5. **Check.** `check.py` (collisions, one valid solid per part, bed fit, hardware and screw
   rules) and `overhang.py`, each with its negative control. Report what is left honestly.
6. **Plates.** Write the Bambu multi-plate project from code:
   [references/bambu-3mf.md](references/bambu-3mf.md). Slice it from the command line for
   real time and grams: [references/bambu-cli.md](references/bambu-cli.md). Open it in
   Bambu Studio for the user.
7. **3D web page.** Build and publish the interactive model + "Build it step by step" guide:
   [references/web-viewer.md](references/web-viewer.md).
8. **Hand over.** Tell the user, per plate: nozzle, filament colour, time, grams, any pause
   (layer, Z, time into the print, what to do at it), and which plate to print first.
   Plan the order around their waking hours (one night-length plate at bedtime).
9. **Learn.** When they report back, fix, rebuild, lock printed plates, and write the lesson
   into the knowledge base (`lessons-learned.md`, `design-for-print.md`, `projects/<name>.md`)
   with the date.

## Completion report

Keep it short: what changed, files, checks run (with their negative controls), plate table
(time, grams, nozzle, colour, pauses), what is still assumed, and risks.
