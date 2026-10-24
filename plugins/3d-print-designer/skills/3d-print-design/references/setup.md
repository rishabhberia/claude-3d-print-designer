# First-time setup

Run this once per computer. Check each item yourself where you can, and tell the user exactly
what is missing. Don't install system software without the user's go-ahead.

## 1. Tools the user installs

| Tool | Why | How to check |
|---|---|---|
| **Python 3.11 or 3.12** | runs CadQuery | `python3 --version` (tested with 3.11) |
| **Bambu Studio** | slicer; also sliced from the command line | macOS: `/Applications/BambuStudio.app/Contents/MacOS/BambuStudio --help`. Windows/Linux: find the installed executable (often `bambu-studio`) and confirm `--help` runs. Note the exact version (Help → About) |
| **Git** | versions every design | `git --version` |
| **Calipers** (0.01 mm) and a **10 mm-grid cutting mat** | measuring | ask the user |
| Chrome + the **Claude in Chrome** extension (optional) | checking the published web page as the user sees it | ask the user |

Bambu Studio downloads: https://bambulab.com/en/download/studio

## 2. The CAD environment (CadQuery)

CadQuery is a pip package, not an app. Each project gets its own virtual environment:

```bash
cd <project>
python3 -m venv .venv
.venv/bin/python -m pip install --upgrade pip
.venv/bin/python -m pip install "cadquery>=2.8" trimesh manifold3d shapely scipy rtree rectpack fast_simplification matplotlib pillow
.venv/bin/python -c "import cadquery, trimesh, manifold3d; print('cadquery', cadquery.__version__)"
```

Rules:
- **Never install into a base Anaconda or system Python.** Installing CadQuery into Anaconda's
  base once upgraded numpy and broke that environment. Check the interpreter path before every
  `pip install`.
- `rtree` is needed for trimesh ray casting, `rectpack` for plate packing,
  `fast_simplification` for lighter render meshes.
- Several projects may share one `.venv` if the user prefers; record which one in the project
  README.
- Optional: the user can view CAD live with the OCP CAD Viewer for VS Code (`pip install
  ocp_vscode`) or CQ-editor. The workflow doesn't need either: renders and Bambu Studio do the
  showing.

## 3. A template project from the user's own printer

Generated Bambu projects need a **complete** `project_settings.config` copied from a project
saved on the same printer, nozzle and Bambu Studio version (see bambu-3mf.md). Ask the user to:

1. open Bambu Studio, choose their printer, nozzle and filament;
2. add any small part, slice it, and **save the project** (.3mf);
3. tell you its path. One template per nozzle size they use.

Check it: unzip it and confirm `Metadata/project_settings.config` exists and parses as JSON.

## 4. The knowledge base

Ask the user where to keep it (suggest `<their projects folder>/3d-printing-knowledge/`, a
private git repo). Copy `templates/knowledge-base/` there and fill in from the user's answers:

- printer model and bed size (plan a 5 mm border inside it);
- nozzles they own (size and material: stainless or hardened steel); default nozzle;
- filaments, brands and colours, and which colour is for visible vs hidden parts;
- Bambu Studio version and executable path; template project path(s);
- the `.venv` path;
- their waking hours (for planning print queues);
- where the printer sits (if it's in a room they work in, stick to PLA/PETG and suggest
  ventilation).

Then `git init` it and commit.

## 5. Point Claude at it

Add to the user's `~/.claude/CLAUDE.md` (show them the text first):

```markdown
# 3D printing
- Before any 3D-printing work, read <path>/3d-printing-knowledge/memory.md and use the
  3d-print-design skill. When a new fact is established, update the right file with the date
  and commit.
```

## 6. Setup check (report this table)

| Item | Status |
|---|---|
| Python version | |
| `.venv` with cadquery imports | |
| Bambu Studio executable runs `--help` | |
| Template project(s) found, settings parse | |
| Knowledge base created and committed | |
| CLAUDE.md pointer added | |

Anything not ✓ is listed with the fix. Don't hide a failed item.
