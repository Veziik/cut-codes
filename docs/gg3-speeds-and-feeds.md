# GG3 Speeds & Feeds Reference

Extracted from all official Defense Distributed and GGD .dd cut code files. All values are what the production code actually uses -- not theoretical calculations.

---

## Speed/Feed Summary Table

### Aluminum Milling

| Tool | Diameter | RPM | Feed (cutting) | Plunge | Units | Source Projects |
|------|----------|-----|----------------|--------|-------|-----------------|
| Flat end mill | 1/4" (6.35mm) | **7890-8000** | 762 mm/min (30 in/min) | 1270 mm/min (50 in/min) | mm | FMDA Front Rails |
| Flat end mill | 1/4" (6.35mm) | **8000** | 200 mm/min | 200 mm/min (F200) | mm | P80, AK47 |
| Flat end mill | 1/4" (6.35mm) | **6000-8000** | 24 in/min | 6-12 in/min | in | AR15 (well mill) |
| Flat end mill | 1/4" (6.35mm) | **8000** | 28.8 in/min | 7.2 in/min | in | AR15/AR308 (selector) |
| Flat end mill | 1/4" (6.35mm) | **5000** | 10-50 in/min | 50 in/min | in | G0 (face operation) |
| Flat end mill | 1/4" (6.35mm) | **8000** | 100 mm/min | 40-80 mm/min | mm | AR Zero Buffer Tower |
| Flat end mill | 1/8" (3.175mm) | **8000** | 200 mm/min | 200 mm/min (F200) | mm | AK47 |
| Flat end mill | 1/8" (3.175mm) | **8000** | 254-508 mm/min | 254 mm/min | mm | FMDA (1/8" operations) |
| Flat end mill | 3/32" (2.38mm) | **6500** | 5 in/min | 5 in/min | in | G0 (corner cleanup) |
| Ball end mill | 1/8" (3.175mm) | **8000** | 10 in/min | 10 in/min | in | G0 (finishing) |
| 5/32" drill | 5/32" (3.97mm) | **8000** | 4 in/min (plunge) | 4 in/min | in | AR15/AR308 (trigger pins) |
| 4mm spot drill | 4mm | **5500** | 150-400 mm/min | 100 mm/min | mm | P80 |
| 3mm drill | 3mm | **10000** | 200 mm/min | -- | mm | P80 |

### Steel Milling (Optic Cuts, 1911 Steel)

| Tool | Diameter | RPM | Feed (cutting) | Plunge | Units | Source Projects |
|------|----------|-----|----------------|--------|-------|-----------------|
| Flat end mill | 5/32" (3.97mm) | **5000** | 10 in/min | 13.3 in/min | in | Optic Cut (footprint milling) |
| Flat end mill | 1/16" (1.59mm) | **6500** | 5 in/min | 5 in/min | in | Optic Cut (bore milling) |
| Thread mill | 0.080" | **6500** | 3 in/min | 3 in/min | in | Optic Cut (4-40 threads) |
| Flat end mill (slot) | -- | **5000** | 25 in/min (rough) / 8 in/min (finish) | 15-18 in/min | in | 1911 Steel |
| Ball end mill | 1/8" | **3000** | 55 mm/min | 40 mm/min | mm | 1911 Steel |
| Chamfer mill | 1/8" | **6000** | 5-6 in/min | -- | in | 1911 Steel |

### 1911 Aluminum vs Steel Comparison (Same Slotting Operation)

| Parameter | Aluminum | Steel | Reduction |
|-----------|----------|-------|-----------|
| Spindle RPM | **8000** | **5000** | 37.5% slower |
| Rough slot feed | 30 in/min | 25 in/min | 17% slower |
| Finish slot feed | 15 in/min | 8 in/min | 47% slower |
| Plunge feed | 20 in/min | 15 in/min | 25% slower |
| Finish plunge | 60 in/min | 40 in/min | 33% slower |

### Probing Speeds (All Materials)

| Parameter | Value | Notes |
|-----------|-------|-------|
| Spindle RPM during probe | **5000** (typical), **1500** (FMDA) | Slow spindle to minimize vibration |
| Probe feed rate | **30 mm/min** (F30) | Standard across all projects |
| Post-probe retract | **2 mm** incremental | G91 G0 Z2.0 or similar |

---

## Spindle Speed Patterns

### Warmup Sequence (Used Across All Projects)

The GG3 VFD spindle requires a graduated warmup before machining:

```gcode
M3 S5000     ; Start at 5000 RPM
G4 P0.5      ; Dwell 0.5 sec (or P1, P2)
S6000        ; Step up
G4 P0.5
S7000        ; Step up
G4 P0.5
S8000        ; Full machining speed
G4 P0.5      ; Brief dwell before cutting
```

Some projects use `G4 P10` (10 sec dwell) for a longer warmup.

### RPM by Operation Type

| Operation | Aluminum RPM | Steel RPM |
|-----------|-------------|-----------|
| Roughing (1/4" endmill) | 7890-8000 | 3000-5000 |
| Finishing (1/4" endmill) | 8000 | 5000 |
| Roughing (1/8" endmill) | 8000 | 6000-6500 |
| Finishing (1/8" ball) | 8000 | 3000 |
| Corner cleanup (3/32") | 6500 | -- |
| Drilling | 8000-10000 | 6500 |
| Thread milling | -- | 6500 |
| Probing | 5000 | 5000 |
| Idle/positioning | S0 or S1 | S0 or S1 |

---

## Feed Rate Patterns

### Common Feed Rates by Tool & Material

#### Aluminum - 1/4" Flat End Mill (most common tool)

| Operation | Feed (mm/min) | Feed (in/min) | Notes |
|-----------|--------------|---------------|-------|
| Facing | 254-508 | 10-20 | Light surface skim |
| Roughing (contour) | 635-762 | 25-30 | Main material removal |
| Roughing (adaptive/pocket) | 200 | 8 | Deeper engagement |
| Finishing | 762-1270 | 30-50 | Final passes |
| Plunge/ramp-in | 635-1270 | 25-50 | Entry moves |
| Slot cutting | 381-762 | 15-30 | Full-width slot |

#### Aluminum - 1/8" Flat End Mill

| Operation | Feed (mm/min) | Feed (in/min) | Notes |
|-----------|--------------|---------------|-------|
| Roughing | 200-508 | 8-20 | |
| Finishing | 254 | 10 | |
| Plunge | 254 | 10 | |

#### Steel - 5/32" Flat End Mill

| Operation | Feed (mm/min) | Feed (in/min) | Notes |
|-----------|--------------|---------------|-------|
| Footprint roughing | 254 | 10 | Optic cuts |
| Plunge/ramp | 338 | 13.3 | |
| Fine finishing | 127-203 | 5-8 | |

#### Steel - 1/16" Flat End Mill

| Operation | Feed (mm/min) | Feed (in/min) | Notes |
|-----------|--------------|---------------|-------|
| Bore milling | 127 | 5 | Very light cuts |
| Plunge | 127 | 5 | |

#### Steel - Thread Mill (0.080")

| Operation | Feed (mm/min) | Feed (in/min) | Notes |
|-----------|--------------|---------------|-------|
| Thread milling | 76 | 3 | Helical interpolation |

---

## Depth Per Pass (Step-Down)

Extracted from Z-axis increments in actual toolpaths:

| Material | Tool | Step-Down | Notes |
|----------|------|-----------|-------|
| Aluminum | 1/4" endmill | 1.0 mm (~0.040") | FMDA, contour operations |
| Aluminum | 1/4" endmill | 0.035" per helix | AK47, circular pocket milling |
| Aluminum | 1/4" endmill | 0.025" ramp entry | AR15 well milling |
| Aluminum | 1/8" endmill | 0.5-2.5 mm | FMDA, pocket/contour |
| Aluminum | 1/4" endmill (slot) | Full depth single pass | 1911 slot (0.219" deep) |
| Steel | 5/32" endmill | 0.015-0.020" ramp | Optic Cut footprints |
| Steel | 1/16" endmill | 0.005" per helix | Optic Cut bore milling |
| Steel | Thread mill | Full thread depth in spiral | Single helical pass |

---

## Recommended Starting Points for Custom Work

Based on the patterns above, conservative starting values for a custom design:

### Aluminum (6061-T6)

| Parameter | 1/4" Endmill | 1/8" Endmill |
|-----------|-------------|-------------|
| RPM | 8000 | 8000 |
| Cutting Feed | 500 mm/min (20 in/min) | 250 mm/min (10 in/min) |
| Plunge Feed | 250 mm/min (10 in/min) | 125 mm/min (5 in/min) |
| Step-Down | 1.0 mm (0.040") | 0.5 mm (0.020") |
| Step-Over | 50% tool diameter | 50% tool diameter |

### Steel (mild/stainless slides)

| Parameter | 5/32" Endmill | 1/8" Endmill |
|-----------|--------------|-------------|
| RPM | 5000 | 6500 |
| Cutting Feed | 250 mm/min (10 in/min) | 125 mm/min (5 in/min) |
| Plunge Feed | 125 mm/min (5 in/min) | 75 mm/min (3 in/min) |
| Step-Down | 0.5 mm (0.020") | 0.25 mm (0.010") |
| Step-Over | 40% tool diameter | 40% tool diameter |

### Probing (All Materials)

| Parameter | Value |
|-----------|-------|
| RPM | 5000 |
| Probe Feed | 30 mm/min |
| Probe Retract | 1-2 mm |

---

## Key Observations

1. **8000 RPM is the workhorse speed** for aluminum milling across all projects. The GG3 tops out at 9000 RPM, but official code rarely exceeds 8000.

2. **5000 RPM is standard for steel** and for all probing operations regardless of material.

3. **Feed rates are conservative** compared to what larger CNC mills use. The GG3 is a light desktop machine -- pushing it harder risks chatter, deflection, or stalling.

4. **Steel feeds are roughly 50% of aluminum feeds** for the same operation type and tool.

5. **The spindle warmup sequence is mandatory** -- jumping straight to 8000 RPM from cold can damage the VFD or cause inconsistent spindle speed. Always ramp through 5000 > 6000 > 7000 > 8000 with dwells.

6. **S1 and S0 are used for positioning** -- setting spindle to near-zero while repositioning prevents accidental cutting and reduces noise. `S1 M5` is common for "spindle off after probing."

7. **G0 project uses the widest range of tools** (0.25", 0.09375", 0.125" ball) and demonstrates that smaller tools run at higher RPM but much lower feeds.

8. **AK47 files show the highest feed rates** (F800, F1200 mm/min) for the 1/8" endmill -- these are likely adaptive/trochoidal clearing passes where the tool engagement is low despite the high feed.
