# Design-for-print rules

Learned on real prints (Bambu A2L, PLA, 0.4 mm nozzle). Numbers are starting points; replace
them with the user's own results as they come in and note the date.

## Orientation

- **Finished outside face prints UP**, never on the bed: the top surface comes out better.
  Hidden faces go on the bed. If flipping a part would need supports under its show face,
  redesign it (e.g. split a decorative skin from its frame). If "face up" and "no overhang"
  conflict, ask the user.
- **No supports.** Turn each part so nothing prints over air. Bosses are full-height columns
  from the bed, recesses open upward, chamfers replace flat ceilings, bridges stay ≤ 35–40 mm.
- **Audit overhangs** for every part in its print orientation: down-facing area steeper than
  45° that isn't on the bed, plus bridges. Compare all six orientations and report what's left.
  A hidden face may go on the bed if that removes a big bridge.
- **Mirrored parts need their own rotation.** A turn that works for a part can put its mirror
  image's bed face on top; a bounding-box check can't see it.

## Labels and hardware

- **Part IDs:** every part in a multi-part design gets its ID **debossed 0.6 mm** (cut in, never
  raised, so mating faces stay flat) on a hidden face that is UP in print orientation. Bold
  font, 7 mm high (5 mm minimum on a 0.4 nozzle), 1.5 mm clear of edges and holes, solid
  plastic underneath. Identical copies share one label so they stay one print geometry. Never
  add labels to parts already printed. Show the same IDs on the web page.
- **Magnets are sealed inside the print**, no glue, not visible:
  - closed cavity = magnet thickness + 0.2 mm, floor and roof on the layer grid (0.2 mm layers:
    0.6 mm skin, 3.2 mm cavity for a 3 mm magnet);
  - a pause written into the project at the layer above the cavity top (see bambu-3mf.md);
    one pause height per plate where possible;
  - tell the user the layer number, Z, time into the print, and a polarity method that can't
    go wrong: stack all magnets in one column, dot every top face, then state "dot up" or "dot
    down" per part **in its print orientation**; test each magnet against the mating part
    before resuming;
  - use a stainless nozzle (hardened steel is magnetic).
- **Every screw must be fittable,** not just aligned: it enters from open air, its path is
  clear, the counterbore is on the head side, there's room for the head and the hex key with
  the parts present at that assembly step, and there's no air gap at the insert.
- **Screw length from measured grip:** thread engagement 1.25–1.5 × diameter (5–7 mm into a
  heat-set insert), never past the bottom of the bore. Count hardware from the finished
  geometry, then cross-check against the screw layout.
- Hardware positions come from one function (a `screw_sites()`); every part cuts its holes
  from it.
- Keep magnets, inserts and keepers ≥ 8 mm from other hardware on the same post; enforce it in
  `check.py` (cut pockets are invisible to a collision check).
- **3 walls minimum on anything with heat-set inserts or screw bosses.**
- Keep fixings out of cable channels. Screw into solid features, never into lattice crossings.
- Removable panels on locating tabs plus magnets beat printed hinges: PLA creeps and sags.

## Moving parts

- **A pivot far from a curved wall magnifies interference.** A part that swings about a pin
  near its outer edge moves almost tangent to an inner curved wall. A fraction of a mm of
  radial interference then holds it degrees open, and the free end stands many mm off.
  A closed-pose collision check can't see this. Compute the **closing stop angle** (2D: rotate
  the outline towards closed until it first touches the obstacle) against an obstacle a
  little bigger than measured, and require 0°.
- Leave ≥ 1.5 mm where the obstacle's size came from a ruler or a photo, not calipers.
- **Don't widen a part to close a gap seen on a part that isn't seated.** Find what holds it
  open first.
- Check a downloaded mechanism over its full motion before changing it (see Measuring).

## Fits and bed

- Clearances: sliding 0.40 mm, general 0.30 mm, panel-to-panel 0.35 mm, rebate 0.30 mm.
- Keep a 5 mm border from the bed edge and 6 mm between parts. Check it in code and fail if a
  plate doesn't fit.
- Long parts: split into corner Ls plus infills with tongue/mortise joints and screwed straps,
  keeping joints out of load paths.

## Filament and time (from real slices)

| Change | Filament | Time |
|---|---|---|
| 6 walls, 8/6 top/bottom, 20 % → 3 walls, 5/4, 12 % gyroid | −27 % | −15 % |
| Cutting windows (netting) into hidden parts | −2.5 % | ±0 |
| 3 → 2 walls | −11 % | +4 % |

- **The slicer profile is the big lever.** A light profile (3 walls, 5 top / 4 bottom, 12 %
  gyroid) is a good default for hidden structure.
- Netting thin plates saves almost nothing (every window gets its own perimeters). If a hidden
  face carries nothing, leave it open.
- Tall thin walls print solid: thin walls and ribs (1.6 mm) and thicken only at fixings.
- Fine lattices cost time (thousands of short moves).
- Weigh a redesign with a one-part slice before and after; CAD volume misleads.
- **Small vertical ribs are slow.** 20 rib islands (1.2 × 1.2 mm, full height) on a box's
  walls added 45 % to its print time: every layer gets many tiny perimeters. Half-height ribs
  cost about half that. Slice the variants and let the user choose with real numbers. Rib
  pairs of that size print cleanly with a 0.4 mm nozzle, no supports.
- **Partitions in a wedge or round box:** put the slots on radial lines. Every radial line is
  the same length, so one insert fits every slot; concentric dividers need a part per radius.

## Measuring

- Calipers for every size that matters. Photos only for outlines, on a 10 mm-grid cutting mat,
  checked against at least one caliper value.
- Anything asymmetric measured from a photo gets a handedness check: overlay the outline on
  the photo in the same frame. State each coordinate frame's handedness in its docstring.
- **Ask how thick the material is at the hole**, not "the wall thickness": a recess round a
  hole can leave far less than the nominal wall.
- "Distance between the two round ends" can be centre to centre or edge to edge. Ask for
  centre to centre and cross-check against a photo scaled by a known size.
- If the user didn't understand a question ("recess depth"), it is still open. Ask again as a
  measurement they can take, and keep the risky dimension short of the nominal value.
- **Use the host object's own features before adding parts** (existing holes as hinges, a
  dip that lets a cap sit flush). Asking first removed four parts and about three hours of
  print time on one project.
- **Modular means per unit.** For a stack of identical units, design the alignment piece
  first, as one identical interlocking part per unit, and aim for one plate per unit that is
  simply reprinted.
- A downloaded mechanism (print-in-place hinge etc.): check it for built-in interference over
  its **full** motion before changing it, and check the new full range after.
