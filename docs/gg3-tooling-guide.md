# GG3 Tooling Guide

Compiled from the GG3 Operator's Manual, all official .dd cut code files, the Ghost Gunner shop, and the Optic Cut Library documentation.

---

## Machine Constraints (What Fits)

| Parameter | Value |
|-----------|-------|
| Collet system | ER-11 (industry standard) |
| Accepted tool diameters | 1mm to 8mm (0.040" to 0.315") |
| Max tool length (OAL) | 82.5mm (3.25") -- longer tools crash during homing |
| Spindle RPM (guaranteed) | 3000-8000 |
| Spindle RPM (typical) | 1000-9000 |
| Max cutting power (peak) | 225W |
| Max cutting power (continuous) | 100W |
| Max linear velocity X | 2540 mm/min (~100 IPM) |
| Max linear velocity Y/Z | 3100 mm/min (~122 IPM) |
| Max acceleration | 500 mm/s^2 (~20 in/s^2) |
| Spindle runout | 0.0009" (measured 1mm from collet tip) |

---

## Every Tool Observed Across All Official .DD Files

### End Mills

| Tool | Diameter | Shank | Flutes | Material | Used In | RPM | Purpose |
|------|----------|-------|--------|----------|---------|-----|---------|
| 1/4" flat end mill | 6.35mm | 1/4" | 3-4 | Carbide | AR15, AR308, AK47, AR Zero, P80, FMDA, G0 | 7890-8000 | Primary roughing/finishing, aluminum |
| 5/32" flat end mill | 3.97mm | 3/16" | 4 | Carbide | Optic Cut, AR15, AR308 | 5000 | Steel slide milling, optic footprints |
| 1/8" flat end mill | 3.175mm | 1/8" | 3-4 | Carbide | AK47, FMDA, G0, AR Zero | 8000 | Detail work, smaller pockets, aluminum |
| 3/32" flat end mill | 2.38mm | 1/8" | -- | Carbide | G0 | 6500 | Corner cleanup, fine detail |
| 1/16" flat end mill | 1.59mm | 1/8" | 4 | Carbide | Optic Cut, G80 | 6500 | Bore milling, very fine detail, steel |
| 1/8" ball end mill | 3.175mm | 1/8" | 2 | Carbide | G0, 1911 (Al & Steel), G80 | 3000-8000 | 3D finishing/deburring |
| 1/4" ball end mill | 6.35mm | 1/4" | 2 | Carbide | G0, AR Zero (BTA) | 8000 | 3D surface finishing |
| 1/8" chamfer mill | 3.175mm | 1/8" | -- | Carbide | 1911 Steel, AR Zero | 6000 | Edge breaking, chamfers |

### Drills

| Tool | Diameter | Shank | Material | Used In | RPM | Purpose |
|------|----------|-------|----------|---------|-----|---------|
| 5/32" drill | 3.97mm | 4mm | HSS/Carbide | AR15, AR308 | 8000 | Trigger/hammer pin holes |
| #35 drill (0.110") | 2.79mm | 1/8" | -- | 1911 (Al & Steel) | 8000 | Screw holes |
| #22 drill | -- | -- | -- | 1911 (Al & Steel) | 8000 | Larger screw holes |
| 4mm spot drill | 4mm | 4mm | Carbide | P80 | 5500 | Hole spotting |
| 3mm drill | 3mm | 3mm | Carbide | P80, FMDA | 8000-10000 | Pin holes |
| 1/8" drill | 3.175mm | 1/8" | HSS | FMDA | 8000 | Drilling through aluminum |

### Thread Mills

| Tool | Diameter | Shank | Used In | RPM | Purpose |
|------|----------|-------|---------|-----|---------|
| 4-40 thread mill (0.080") | 2.03mm | 1/8" | Optic Cut | 6500 | Tapped holes in steel slides |
| 6-32 thread mill (0.098") | 2.49mm | 1/8" | Optic Cut | 6500 | Tapped holes in steel slides |
| M3 thread mill | ~2mm | 1/8" | Optic Cut | 6500 | Metric tapped holes |

---

## Required ER-11 Collets

Based on the tools above, you need these collet sizes:

| Collet Size | Holds Tools With Shank | Required For |
|-------------|----------------------|--------------|
| **1/4" (6.35mm)** | 1/4" end mills, 1/4" ball mills | Most operations |
| **1/8" (3.175mm)** | 1/8" end mills, 1/8" drills, 3/32" end mills, 1/16" end mills, thread mills | Detail work, bore milling, threading |
| **4mm** | 5/32" drills, 5/32" end mills, 4mm spot drills | Pin holes, optic cuts |
| **3/16" (4.76mm)** | 5/32" end mills (Optic Cut kit uses Rego-Fix 5/32" collet) | Optic cuts |
| **3mm** | 3mm drills | P80, FMDA pin holes |

**Minimum set for custom aluminum work:** 1/4" and 1/8" collets.

---

## Official DD Recommended Speeds (From the Operator's Manual)

The manual provides this table for aluminum at 8000 RPM, all values in inches:

### Chip Load (inches per tooth) by Tool Diameter

| Operation | 1/16" (0.0625") | 1/8" (0.125") | 1/4" (0.250") |
|-----------|-----------------|---------------|---------------|
| Slotting | 0.0005 IPT | 0.0007 IPT | 0.001 IPT |
| Roughing | 0.0005 IPT | 0.0007 IPT | 0.001 IPT |
| Finishing | 0.0005 IPT | 0.0007 IPT | 0.001 IPT |
| Adaptive | 0.0005 IPT | 0.0007 IPT | 0.001 IPT |

### Depth of Cut by Operation

| Operation | Radial (step-over) | Axial (step-down) |
|-----------|-------------------|-------------------|
| Slotting | 1.00x diameter | 0.10x diameter |
| Roughing | 0.60x diameter | 0.25x diameter |
| Finishing | 0.25x diameter | 1.00x diameter |
| Adaptive | 0.04x diameter | 2.50x diameter |

### Calculated Feed Rates (IPT x flutes x RPM)

For a **3-flute 1/4" end mill at 8000 RPM** (the most common tool):
- Feed = 0.001 IPT x 3 flutes x 8000 RPM = **24 IPM** (610 mm/min)

For a **4-flute 1/8" end mill at 8000 RPM**:
- Feed = 0.0007 IPT x 4 flutes x 8000 RPM = **22.4 IPM** (569 mm/min)

For a **4-flute 1/16" end mill at 6500 RPM** (steel):
- Feed = 0.0005 IPT x 4 flutes x 6500 RPM = **13 IPM** (330 mm/min)

### Helical Plunge / Ramp Entry

| Tool Diameter | Ramp Angle | Ramp Diameter |
|---------------|-----------|---------------|
| 1/16" (0.0625") | 1 degree | 0.0593" |
| 1/8" (0.125") | 1 degree | 0.1187" |
| 1/4" (0.250") | 1 degree | 0.2375" |

### Drilling

| Tool Diameter | RPM | Feed per Rev | Plunge Rate | Peck Depth |
|---------------|-----|-------------|-------------|------------|
| 1/16" | 8000 | 0.0593" | 7.2 IPM | 0.25x dia |
| 1/8" | 8000 | 0.1187" | 14.4 IPM | 0.25x dia |
| 1/4" | 8000 | 0.2375" | 24 IPM | 0.25x dia |

---

## Recommended Tool Set for Custom Work

### Tier 1: Essential (Aluminum General Purpose)

| Item | Spec | Approx Cost | Why |
|------|------|-------------|-----|
| 1/4" 3-flute flat end mill | Carbide, 1-1/4" LOC, 3" OAL | ~$40 | Primary roughing and finishing tool. Handles 90% of aluminum work. |
| 1/8" 4-flute flat end mill | Carbide, 1/2" LOC, 2" OAL | ~$30 | Detail work, smaller features, tighter corners |
| ER-11 collet, 1/4" | Industry standard | ~$10 | Holds 1/4" tools |
| ER-11 collet, 1/8" | Industry standard | ~$10 | Holds 1/8" tools |

**Total: ~$90.** This covers pockets, profiles, faces, and most 2.5D aluminum milling.

### Tier 2: Expanded (Holes + Finishing)

Add to Tier 1:

| Item | Spec | Approx Cost | Why |
|------|------|-------------|-----|
| 5/32" carbide drill | 2.5" OAL | ~$35 | Pin/screw holes |
| 1/8" ball end mill | 2-flute carbide, 2" OAL | ~$25 | 3D surface finishing, deburring, radius work |
| 1/16" flat end mill | 4-flute carbide, 1.5" OAL, 1/8" shank | ~$35 | Very fine detail, small bores |
| ER-11 collet, 4mm (or 3/16") | For 5/32" shank tools | ~$10 | Holds 5/32" drills/end mills |

**Added: ~$105. Running total: ~$195.**

### Tier 3: Steel Capable (Optic Cuts, Hardened Materials)

Add to Tier 1+2:

| Item | Spec | Approx Cost | Why |
|------|------|-------------|-----|
| 5/32" 4-flute flat end mill | Carbide, 3/16" shank, 1.5" OAL, 7/32" LOC+ | ~$30 | Steel slide milling (optic footprints) |
| 4-40 thread mill | 0.080" dia, 1/8" shank (e.g., Scientific Cutting Tools SPTM080LA) | ~$80 | Tapped holes for optic mounting screws |
| 6-32 thread mill | 0.098" dia, 1/8" shank (e.g., Scientific Cutting Tools SPTM098LA) | ~$86 | Alternative thread size |
| 1/8" chamfer mill | 45-degree, 1/8" shank | ~$25 | Edge breaking, countersinks |

**Added: ~$220. Running total: ~$415.**

### Tier 4: Full Professional Set

Add to all above:

| Item | Spec | Approx Cost | Why |
|------|------|-------------|-----|
| 3/32" flat end mill | 4-flute carbide, 1/8" shank | ~$25 | Corner cleanup in tight areas |
| 1/4" ball end mill | 2-flute carbide | ~$25 | Larger 3D finishing surfaces |
| 3mm drill | Carbide, 3mm shank | ~$15 | Metric pin holes |
| 4mm spot drill | 90-degree carbide | ~$20 | Hole spotting before drilling |
| ER-11 collet, 3mm | For 3mm shank tools | ~$10 | Holds metric drills |
| Chip fan | Clips to ER-11 nut | ~$15 | Chip evacuation, part cooling |
| Spare 1/4" end mill | Same as primary | ~$40 | Tools are consumable (8-10 AR lowers per end mill) |

**Full set total: ~$565.**

---

## Buying Tools: What to Look For

### End Mills

- **Material**: Solid carbide (not HSS). The GG3 runs at high RPM where carbide excels. HSS is acceptable for drills only.
- **Flutes**: 2-3 flutes for aluminum (better chip clearance), 4 flutes for steel (more rigidity)
- **Coating**: AlTiN or TiAlN for steel. Uncoated or ZrN for aluminum. Avoid TiN (outdated).
- **OAL**: Must be under 82.5mm (3.25"). Buy "stub" or "short" length tools when available.
- **Shank**: Must match an available ER-11 collet size. Common: 1/4", 1/8", 4mm, 3/16", 3mm.
- **LOC (length of cut)**: Must be longer than your deepest cut. Shorter LOC = more rigidity = less chatter.

### Collets

- **Standard ER-11** from any reputable supplier (Maritool, Techniks, Rego-Fix, etc.)
- Rego-Fix collets are used in the official Optic Cut kit and are high precision
- Budget collets from Amazon/AliExpress work fine for most hobby use
- Each ER-11 collet has a 0.5mm collapse range (e.g., a 6mm collet holds 5.5-6.0mm shanks)

### Where to Buy

| Source | Notes |
|--------|-------|
| [Ghost Gunner Store](https://ghostgunner.net/shop/) | Official tools, known compatible, higher price |
| Amazon / eBay | "Destiny Tool Viper" 1/4" 2-flute has been sold specifically for GG use |
| Harvey Tool / Helical | High quality micro end mills for detail work |
| Maritool | Good ER-11 collets at reasonable prices |
| McMaster-Carr | Everything including the spring (1986K74) DD recommends for probe development |
| Scientific Cutting Tools | Thread mills (SPTM080LA for 4-40, SPTM098LA for 6-32) |

---

## Manual Notes on Tool Care

- "Never use a dropped, visibly damaged, dull, or suspect cutting tool. Worn tools -- particularly carbide tools -- are brittle and could shatter while in motion."
- "End mills are considered a consumable." Expect 8-10 AR lowers per 1/4" carbide end mill.
- "Insufficient torque can allow the tool to walk out while milling, ruining your work piece and tool." Torque the collet nut to **30 foot-pounds**.
- "Metal chips reduce gripping force and increase runout." Always clean the collet and tool shank before installation.
- For probe development, DD recommends a **1/4" cylindrical spring** (McMaster 1986K74) instead of an end mill, so crashes deflect harmlessly.
- Use **CAM programs that cut along part geometry with uniform radial engagement** (adaptive clearing). Row-by-row raster programs work for wood/plastic but not metal.
- **Climb milling** is specified across all official code.
