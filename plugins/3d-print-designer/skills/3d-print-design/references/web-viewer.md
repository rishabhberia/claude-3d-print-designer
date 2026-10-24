# The interactive 3D page and assembly guide

Every design ships with a web page where the user (and anyone they share it with) can:

- turn and zoom the assembled model;
- open, lift or move the moving parts;
- cut it open with a clipping plane and see what goes inside (electronics, hardware stand-ins);
- follow **"Build it step by step"**: each printed part flies into place in build order with
  its ID, instruction, hardware, and the plate it prints on; loose parts wait on a tray.

## Export (`export_web.py`)

- Export the assembly as glTF/GLB with **one node per printed part at its assembled
  position** (trimesh scene), named by part ID. Decimate render meshes with
  `fast_simplification` to keep the file small.
- Write a step table (`meta.json`): step number, title, instruction, part IDs, hardware, plate.
- **Placing every part is a design check:** it has found floating parts and unplaced straps
  that the collision check missed. Any part without a step is a defect.
- Very dense geometry (woven/cane textures, millions of faces) won't fit: rasterise one
  periodic tile to an alpha texture (UVs in mm, `alphaTest`, plus a `customDepthMaterial` for
  holey shadows) and instance repeated features. Keep each instance a proper rotation
  (det > 0) or it mirrors.

## Page (`web/index.html`)

- three.js from `cdn.jsdelivr.net` via an importmap, pinned to an exact version (e.g.
  `three@0.160.0`), with OrbitControls and GLTFLoader.
- Panel UI: in a flex column with `max-height` + `overflow:auto`, set `flex-shrink:0` on every
  child, or sections with `overflow:hidden` collapse to zero height.
- Use HTML entities or `\u` escapes for non-ASCII text (a plain local server sends no charset),
  and add `[hidden]{display:none!important}` yourself.

## Publishing

**If the Artifact tool is available (Claude Code desktop / claude.ai):** publish as a private
artifact. The artifact host does **not serve `.glb`** and its CSP **blocks `fetch()` of `data:`
URLs** (so glTF with embedded base64 buffers fails too). Working method:

1. write the GLB as base64 into `model.txt`;
2. in the page: `fetch('model.txt')` → `.text()` → `atob` → `Uint8Array` →
   `new GLTFLoader().parseAsync(bytes.buffer, '')`.

Test locally first under a CSP meta tag that mimics the host:

```html
<meta http-equiv="Content-Security-Policy" content="default-src 'self'; script-src 'self' 'unsafe-inline' https://cdn.jsdelivr.net; img-src 'self' data: blob:; connect-src 'self'">
```

It reproduces the loading failure and proves the fix.

**Otherwise:** serve it locally (`python3 -m http.server -d web 8000`) or publish it with
GitHub Pages; the same base64 method works everywhere.

## Verify

- Look at the live page in the user's own browser (e.g. Claude in Chrome): a private artifact
  can't be seen by a signed-out browser.
- Verify controls by seeing them on screen, not only by clicking them with a script. Scripted
  clicks have "worked" on an invisible button.
- Walk the whole step guide once: every part appears, nothing floats, IDs match the debossed
  labels.
