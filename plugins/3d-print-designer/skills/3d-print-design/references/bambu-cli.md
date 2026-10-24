# Slicing with the Bambu Studio command line

Verified on macOS with Bambu Studio 02.08. On Windows/Linux use the installed executable; the
flags are the same app's, but the paths and the GUI hand-off behaviour below were not tested.

```bash
BS=/Applications/BambuStudio.app/Contents/MacOS/BambuStudio
"$BS" --slice 0 --outputdir "$TMPDIR/slice" out/plates/PROJECT.3mf     # 0 = all plates
```

Each `plate_<n>.gcode` header carries the real numbers:

```
; total filament weight [g] : 614.85
; model printing time: ... ; total estimated time: 12h 23m 13s
```

`slice_report.py` should: slice into a temp dir, parse those headers, **delete the G-code**
(20–30 MB per plate), and write `out/plates/slice_report.json`. A plate with no G-code or no
header is reported as **missing**, never as zero. 28 plates took about 4 minutes.

The G-code layer marker is `; Z_HEIGHT: <z>` (not `;Z:`). Use it to verify pauses and to check
per-layer coverage.

## Traps

- **If the Bambu Studio GUI is running, a CLI call can hand the file to the GUI and exit 0
  without doing the work.** Always check the expected output files exist. Ask the user to close
  the GUI before slicing if outputs are missing.
- `--export-3mf` together with `--outputdir` failed ("Unable to open the file
  sliced.3mf.tmp") while the G-code was still written. Don't rely on `--export-3mf`.
- Exit 139 = segfault: usually an incomplete `project_settings.config` (see bambu-3mf.md).
  A 0.6 mm nozzle project sliced from the CLI also segfaulted once; switching the nozzle in
  the GUI worked.
- Exit 239 "process not compatible with printer": compatibility keys missing.
- macOS: `open -a BambuStudio PROJECT.3mf` opens the project for the user; it may open a second
  window, so tell them to use the newest one.
- The Bambu log is encrypted. Crash reports land in `~/Library/Logs/DiagnosticReports/`
  (`BambuStudio-*.ips`); no new report after an open is the only headless sign it loaded.
- Screenshots of the GUI need the screen-recording permission; otherwise ask the user what the
  GUI shows.

## Useful flags (`--help`)

`--arrange 1`, `--orient`, `--allow-rotations`, `--load-settings "machine.json;process.json"`,
`--load-filaments`, `--export-png <plate>`, `--slice <plate|0>`.
