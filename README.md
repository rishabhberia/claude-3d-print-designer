# 3D Print Designer: a Claude Code skill

Describe something you want to 3D print and Claude Code designs it end to end:

1. **Parametric CAD in Python** with [CadQuery](https://github.com/CadQuery/cadquery). Every
   dimension lives in one `params.py`, so a change is one edit and a rebuild.
2. **Automatic checks**: parts don't collide, each part is one valid solid, it fits the bed,
   screws can actually be fitted, nothing hangs over air. Every check is first shown failing on
   a deliberately broken case.
3. **True-to-print renders** to choose between options before printing anything.
4. **A ready Bambu Studio project (.3mf)** with every part laid out on plates, oriented face up
   with no supports, coloured per plate, with pauses for sealed-in magnets. Claude opens it in
   Bambu Studio for you.
5. **Real print time and grams** from the slicer, not guesses.
6. **An interactive 3D web page**: turn the model, open moving parts, cut it open, and follow
   a "Build it step by step" guide where each part flies into place with its ID, hardware and
   plate number.
7. **A knowledge base** that grows with every print, so the same mistake doesn't happen twice.

Built and used on real projects: a 28-plate rattan-front router cabinet, a coffee dispenser, a
Wi-Fi jar scale, a fold-out wall hook.

## What you need

| | What | Notes |
|---|---|---|
| 1 | **[Claude Code](https://claude.com/claude-code)** | desktop app or CLI |
| 2 | **A 3D printer + [Bambu Studio](https://bambulab.com/en/download/studio)** | built on a Bambu Lab A2L. Other Bambu printers should work; the 3MF details were verified on Bambu Studio 02.08 only |
| 3 | **Python 3.11 or 3.12** | [python.org](https://www.python.org/downloads/) or Homebrew (`brew install python@3.11`) |
| 4 | **CadQuery** (the CAD software) | free, open-source, code-based CAD on the OpenCascade kernel. Installed per project with pip (below); Claude does this for you |
| 5 | **Git** | versions every design |
| 6 | **Digital calipers** + a **10 mm-grid cutting mat** | Claude asks you to measure; photo measurements are often 3–5 % off |
| 7 | Chrome + the Claude in Chrome extension (optional) | lets Claude check the 3D page the way you see it |

### Installing the CAD software (CadQuery)

CadQuery is a Python library, not an app. Each project gets its own virtual environment:

```bash
python3 -m venv .venv
.venv/bin/python -m pip install --upgrade pip
.venv/bin/python -m pip install "cadquery>=2.8" trimesh manifold3d shapely scipy rtree rectpack fast_simplification matplotlib pillow
.venv/bin/python -c "import cadquery; print(cadquery.__version__)"
```

On Windows, use `.venv\Scripts\python` instead of `.venv/bin/python`.

> **If you use Anaconda, never install into its base environment.** Doing that once upgraded
> numpy and broke base. Always use the project's `.venv`.

You don't need a CAD app to look at the parts: Claude renders them and the 3D page shows the
assembly. If you'd like a live CAD view while editing, the optional
[OCP CAD Viewer](https://github.com/bernhard-42/vscode-ocp-cad-viewer) for VS Code or
[CQ-editor](https://github.com/CadQuery/CQ-editor) work with CadQuery.

### One saved project from your own printer

In Bambu Studio choose your printer, nozzle and filament, add any small part, slice it and
**save the project** (.3mf). Claude copies the complete printer settings from this file into
the projects it generates. Without it, Bambu Studio crashes on generated files. Save one per
nozzle size you use.

## Install the skill

**As a plugin (recommended).** In Claude Code:

```
/plugin marketplace add rishabhberia/claude-3d-print-designer
/plugin install 3d-print-designer@3d-print-designer
```

**Or copy the skill folder by hand:**

```bash
git clone https://github.com/rishabhberia/claude-3d-print-designer.git
mkdir -p ~/.claude/skills
cp -R claude-3d-print-designer/plugins/3d-print-designer/skills/3d-print-design ~/.claude/skills/
```

Restart Claude Code after installing.

## First run

Open Claude Code in the folder where you keep print projects and say:

> Set up the 3d-print-design skill for me.

Claude checks Python, CadQuery, Bambu Studio and your template project. It asks for your
printer, nozzles, filament and waking hours, creates your knowledge base, and adds a line to
your `~/.claude/CLAUDE.md` so every future session reads it.

## Then design something

> I want a wall bracket for my cordless drill. It mounts with two screws, black PLA,
> 0.4 mm nozzle. Tell me what to measure, then show me two or three options as renders.

The flow is: brief → measure → choose from renders → build and check → plates and real slice →
3D page → print the riskiest plate first → tell Claude how it came out → it fixes, locks the
printed plates and records the lesson.

## What's inside

```
plugins/3d-print-designer/skills/3d-print-design/
  SKILL.md                    the workflow and the non-negotiable rules
  references/setup.md         first-time setup and checks
  references/design-rules.md  face up, no supports, part IDs, sealed magnets, screws, fits
  references/project-layout.md  project folder, scripts and what each check must do
  references/bambu-3mf.md     writing multi-plate Bambu projects from code (+ pauses)
  references/bambu-cli.md     slicing from the command line, and its traps
  references/web-viewer.md    the 3D page and step-by-step guide
  templates/knowledge-base/   starting files for your own knowledge base
```

## Tested on, and not yet

- **Tested:** macOS, Bambu Lab A2L, Bambu Studio 02.08, PLA, 0.4 mm nozzle, CadQuery 2.8 on
  Python 3.11.
- **Not tested:** Windows and Linux paths for Bambu Studio, other Bambu printers, other
  slicers (PrusaSlicer / OrcaSlicer use a different project format). Corrections welcome.
  Please open an issue or a pull request.

## Safety

PLA is the safe default for a room you sit in. Don't print ABS/ASA/nylon in a room you work
in without ventilation or a filtered enclosure.

## License

MIT. See [LICENSE](LICENSE).
