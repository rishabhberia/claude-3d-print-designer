# Generating Bambu Studio multi-plate projects from code

Verified with Bambu Studio 02.08 on macOS (Bambu A2L, many-plate projects). Other versions
may differ: if the app crashes or ignores plates, compare your file with a project the user
saved in that version.

## Zip layout (minimum that works)

```
[Content_Types].xml                  rels, model, png defaults
_rels/.rels                          -> /3D/3dmodel.model
3D/3dmodel.model                     objects = <components> pointing at mesh files; <build> items
3D/_rels/3dmodel.model.rels          one Relationship per mesh file
3D/Objects/object_<n>.model          one mesh object per distinct part
Metadata/model_settings.config       object names/extruders + <plate> blocks
Metadata/project_settings.config     FULL printer + process + filament config (JSON)
Metadata/custom_gcode_per_layer.xml  only if there are pauses
```

- `3dmodel.model` must carry `<metadata name="Application">BambuStudio-<version></metadata>`.
  Without it the file is treated as a foreign 3MF and plates are ignored.
- **With Application = BambuStudio and a missing or partial `project_settings.config`, Bambu
  Studio crashes (exit 139).** A file with only preset names also crashed.
- **Fix:** copy the complete `project_settings.config` from the user's template project (same
  printer, nozzle, app version), then:
  - resize every per-filament list (the filament preset keys and any `filament_*` key) to n
    filaments;
  - `flush_volumes_matrix` → n×n, `flush_volumes_vector` → 2n;
  - `different_settings_to_system` → [process, *filaments, printer] (n + 2 entries);
  - apply overrides (walls, top/bottom layers, infill).
- Building the config from presets instead needs `print_compatible_printers`,
  `default_print_profile` etc., or the CLI fails with "process not compatible with printer"
  (exit 239). The template route avoids this.

## Objects and plates

- One distinct part = one object. Several `<item objectid=…>` entries for the same object
  become instances 0, 1, 2… in order.
- Centre each mesh in xy at 0 with z-min at 0; the item translation is then the part centre on
  the bed.
- `<plate>` in `model_settings.config` holds `plater_id`, `plater_name`, `locked`,
  `filament_map_mode` "Auto For Flush", `filament_maps` "1 1", `filament_volume_maps` "0 0",
  and `<model_instance>` entries (object_id, instance_id, a unique identify_id).
- **Plate origins:** plate i (0-based) at `x = (i % cols)·bed_x·1.2`,
  `y = −(i // cols)·bed_y·1.2`, `cols = ceil(sqrt(n_plates))`. Item positions must fall inside
  their plate.
- 3MF transform: 12 numbers, row-vector convention (`p' = [x y z 1]·M`):
  `m00 m01 m02 m10 m11 m12 m20 m21 m22 tx ty tz`. 90° about z is `0 1 0 -1 0 0 0 0 1 tx ty tz`.
- Per-object extruder (`<metadata key="extruder" value="1|2">`) gives each plate one colour.
- The nozzle can't change between plates: one project per nozzle size.

## Packing and verification

- `rectpack` (MaxRectsBssf, bin BFF, rotation allowed) on footprint bounding boxes + gap, in a
  bin of (bed − 2·border + gap).
- **Verify independently after writing:** re-read the 3MF, apply the transforms, convex-hull
  each footprint, check it lies inside the bed and nothing overlaps.

## Pause at a layer (embedded magnets)

`Metadata/custom_gcode_per_layer.xml`:

```xml
<custom_gcodes_per_layer>
  <plate>
    <plate_info id="1"/>
    <layer top_z="4.00" type="1" extruder="1" color="" extra="" gcode="M400 U1"/>
    <mode value="SingleExtruder"/>
  </plate>
</custom_gcodes_per_layer>
```

- `type` 1 = pause (0 = colour change, 2 = tool change). `top_z` = the top of the **first layer
  that must not print before the pause** (cavity top + one layer).
- **Check it in the sliced G-code every time:** `M400 U1` must sit right after
  `; Z_HEIGHT: <top_z>` and before that layer's first extrusion. Report layer number, Z and time
  into the print.

## Modifier volumes (e.g. denser infill around screw columns)

One sub-model file holds the part mesh and the modifier meshes; the main object's
`<components>` lists all of them; in `model_settings.config` each modifier gets
`<part id="<component objectid>" subtype="modifier_part">` with settings such as
`<metadata key="sparse_infill_density" value="60%"/>`. Control test: a huge 100 % modifier
must visibly add grams and time.
