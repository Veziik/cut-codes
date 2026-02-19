# Ghost Gunner .DD File Format Cheat Sheet

## What is a .DD File?

A `.dd` file is a **ZIP archive** (renamed from `.zip` to `.dd`) that DDCut (the Ghost Gunner control software) reads to present guided, step-by-step CNC machining workflows to the operator. It bundles together G-code toolpaths, instructional images, and a YAML manifest that orchestrates the user experience.

---

## Archive Structure

Every `.dd` file, when unzipped, contains this basic layout:

```
my-project.dd (ZIP archive)
|
+-- manifest.yml              # REQUIRED - Main job definition
+-- Code/                     # G-code files (.gcode, .txt, .nc)
|   +-- operation1.gcode
|   +-- probe_sequence.nc
|   +-- setup_step.txt
+-- Image/                    # Instructional images (.jpg, .png, .gif)
|   +-- step1.jpg
|   +-- warning.png
+-- Additional Files/         # OPTIONAL - STL files, extras
|   +-- fixture_part.stl
+-- sub_manifest.yml          # OPTIONAL - Reusable step groups
+-- Subfolder/                # OPTIONAL - Organized sub-manifests
    +-- manifest_part.yml
    +-- Code/
    +-- Image/
```

### File Types Observed

| Extension | Purpose | Location |
|-----------|---------|----------|
| `.yml` | Manifest / step definitions | Root or subfolders |
| `.gcode` | Machining G-code (typically numbered: 1000.gcode, 2000.gcode) | `Code/` |
| `.txt` | G-code (alternative extension, same content) | `Code/` |
| `.nc` | G-code, typically for probing/setup routines | `Code/` |
| `.jpg` / `.png` / `.gif` | Step instruction images/animations | `Image/` |
| `.stl` | 3D-printable fixture/jig files (optional extras) | `Additional Files/` |

---

## manifest.yml - Complete Reference

The manifest is a YAML list of **jobs**. Each job is a guided workflow that appears as a selectable option in DDCut.

### Job-Level Keys

```yaml
- job_name: "My Custom Job"                    # REQUIRED - Display name in DDCut job selector
  job_text: "Description of what this job does" # REQUIRED - Description shown to user
  disable_wcs_clear_prompt: true               # OPTIONAL - Skip the "reset WCS?" prompt
  min_fw_version: 20220800                     # OPTIONAL - Minimum firmware version (YYYYMMDD)
  min_ddcut_version: 5.3.3                     # OPTIONAL - Minimum DDCut version

  job_steps:                                   # REQUIRED - Ordered list of steps
    - step_name: "Step 1"
      ...
```

### Step-Level Keys

```yaml
- step_name: "Install End Mill"          # REQUIRED - Step title displayed to user

  # --- Content (at least one required) ---
  step_text: "Plain text instructions"   # Plain text description
  step_markdown: |                       # Markdown-formatted instructions (preferred by newer DDCut)
    **Bold** and [links](url) supported
  step_image: Image/photo.jpg            # Path to instructional image (relative to archive root)

  # --- G-Code Execution ---
  step_gcode: Code/operation.gcode       # Path to G-code file to run when user clicks "Next"
  timeout: 360                           # Timeout in seconds for G-code execution

  # --- Popups / Branching (for decision trees) ---
  popup_title: "Confirm Selection"       # Title of confirmation popup
  popup_text: "Are you sure?"            # Body text of confirmation popup
  popup_yes_step: +10                    # Relative step jump if user clicks "Yes"
  popup_no_step: +1                      # Relative step jump if user clicks "No" (default: next step)

  # --- Sub-manifest Inclusion ---
  # Instead of defining step_name etc., reference another .yml file:
  - manifest_file: subfolder/steps.yml   # Include steps from another YAML file

  # --- Step Navigation ---
  step_goto: +5                          # Jump forward/backward N steps after this step
```

### Minimal Working Example

```yaml
- job_name: My Custom Mill Job
  job_text: Mill a custom part on the GG3

  job_steps:
    - step_name: Verify GG Empty
      step_text: Remove chip cover and verify nothing is in the Ghost Gunner.
      step_image: Image/empty_gg.jpg
      step_gcode: Code/home.txt
      timeout: 30

    - step_name: Install Tool
      step_text: Install the 1/4" end mill into the ER-11 collet.
      step_image: Image/install_tool.jpg

    - step_name: Mill Part
      step_text: Machine will now mill the part. Stay nearby.
      step_image: Image/milling.jpg
      step_gcode: Code/mill_operation.gcode
      timeout: 600

    - step_name: Complete
      step_text: Milling complete. Remove part and clean machine.
      step_image: Image/done.jpg
```

### Sub-Manifest Pattern (for reusable step groups)

**manifest.yml:**
```yaml
- job_name: Full Operation
  job_steps:
    - manifest_file: install_tool.yml
    - manifest_file: probe_and_mill.yml
    - manifest_file: cleanup.yml

- job_name: Resume from Milling
  job_steps:
    - manifest_file: probe_and_mill.yml
    - manifest_file: cleanup.yml
```

**install_tool.yml:**
```yaml
- step_name: Install End Mill
  step_text: Insert the 1/4" end mill...
  step_image: Image/tool.jpg
  step_gcode: Code/home_and_level.txt
  timeout: 30
```

### Decision Tree Pattern (Optic Cut style)

```yaml
- step_name: Choose Option A?
  step_markdown: |
    Do you want Option A? Click next to decide.
  step_image: Image/option_a.png
  popup_title: Option A?
  popup_text: Proceed with Option A?
  popup_yes_step: +5    # Skip ahead to Option A steps
  # If "No", falls through to next step (Option B question)

- step_name: Choose Option B?
  step_markdown: |
    Do you want Option B?
  popup_title: Option B?
  popup_text: Proceed with Option B?
  popup_yes_step: +3    # Skip to Option B steps
```

---

## G-Code Dialect: Grbl

The Ghost Gunner 3 runs **Grbl** firmware. Key characteristics:

### Grbl-Specific Commands

| Command | Meaning |
|---------|---------|
| `$H` | Home all axes |
| `$L` | Unlock (after alarm) |
| `$20=1` | Enable soft limits |
| `M100` | GG3-specific: coordinate averaging (e.g., `M100 G56XG57XG54X`) |
| `G38.2` | Probe toward workpiece (touch-off) |
| `G10 L20 P1 Z0` | Set coordinate system offset (zero current position) |

### Standard G-Code Used

| Code | Meaning |
|------|---------|
| `G0` | Rapid move |
| `G1` | Linear feed move |
| `G20` | Inch mode |
| `G21` | Millimeter mode (more common in GG3 files) |
| `G90` | Absolute positioning |
| `G91` | Incremental positioning |
| `G53` | Machine coordinate system (absolute machine position) |
| `G54`-`G59` | Work coordinate systems |
| `G94` | Feed per minute mode |
| `G17` | XY plane selection |
| `M3` / `M4` | Spindle on (CW / CCW) |
| `M5` | Spindle off |
| `M8` | Coolant on (used as chip fan signal) |
| `M9` | Coolant off |
| `M2` | End program |
| `S5000` | Set spindle speed (RPM) |
| `F10` | Set feed rate |
| `G4P10` | Dwell (pause) for 10 seconds |

### Units

- **Both inches (G20) and millimeters (G21)** are used across official files
- Millimeter mode is more common (~73% of files)
- Spindle speeds observed: 1500-10000 RPM (max 9000 RPM on GG3 spec)

### G-Code File Naming Conventions

Official files use these patterns:
- **Numbered operations**: `1000.gcode`, `2000.gcode`, `3000.gcode` (major operations)
- **Sub-operations**: `4000.gcode`, `4001.gcode`, `4002.gcode` (multi-pass within one op)
- **Setup/probe**: `01_Home_Tool_Install.txt`, `03_Base_Fixture_Probe.txt`
- **Probe routines**: `*.nc` extension common for probing sequences

### Typical G-Code File Structure

```gcode
(Job Name / Comment)
(Tool description: T1 D=0.25 CR=0. - FLAT END MILL)

$H                          ; Home all axes
$L                          ; Unlock

G21 G90                     ; Metric, absolute mode
G0 G90 G53 Z-1.0            ; Retract Z to safe height (machine coords)

; --- Probing sequence ---
S5000 M4                    ; Spindle on
G91 G38.2 Z-20. F30.        ; Probe Z downward
G10 L20 P1 Z0.              ; Set Z zero at touch point
G00 Z1.                     ; Retract

; --- Machining ---
M3 S5000                    ; Spindle on CW at 5000 RPM
G54                         ; Select work coordinate system
G0 X1.0 Y3.8                ; Rapid to start position
G1 Z-0.05 F10.              ; Plunge cut
; ... toolpath ...

M5                          ; Spindle off
M2                          ; End program
```

---

## How to Build a Custom .DD File

### Step-by-Step Process

1. **Create directory structure:**
   ```
   my-project/
   +-- manifest.yml
   +-- Code/
   |   +-- home.txt
   |   +-- probe.nc
   |   +-- mill_op1.gcode
   +-- Image/
       +-- step1.jpg
       +-- step2.jpg
   ```

2. **Write your G-code** (see CAM Software section below)

3. **Take/create instructional photos** for each step

4. **Write manifest.yml** following the schema above

5. **Package as ZIP then rename:**
   ```bash
   cd my-project/
   zip -r ../my-project.dd manifest.yml Code/ Image/
   # The .dd extension is all that's needed - it IS a zip file
   ```

6. **Load into DDCut** via USB drive or direct file selection

### Important Notes

- All file paths in manifest.yml are **relative to the archive root**
- DDCut reads the manifest top-down; steps execute in order unless popups/gotos redirect
- The `timeout` value should be generous enough for the G-code to complete
- Multiple jobs can exist in one .dd file (each is a top-level list item in manifest.yml)
- Sub-manifests (`manifest_file:`) are inlined at that position in the step list
- `step_markdown` is preferred over `step_text` for newer DDCut versions (5.1.4+)
- Comments in YAML use `#` prefix

---

## CAM Software for Generating G-Code

### Recommended: Autodesk Fusion 360 (with GG3 Post Processor)

**Best option for GG3 users.** A dedicated Ghost Gunner 3 post processor exists.

- **CAD + CAM** in one package
- Import STL and STEP files directly
- Generate 2D/3D milling toolpaths
- **GG3-specific post processor** available on [DEFCAD](https://defcad.com/library/ggd-ghost-gunner-3-postprocessor-for-fusion360/)
- Also works with the generic [Grbl post processor](https://cam.autodesk.com/hsmposts)
- Free for personal/hobby use
- Workflow: Import STEP/STL -> CAM workspace -> Create toolpaths -> Post process with GG3/Grbl post -> Get .gcode files

**Setup for GG3:**
- Machine type: 3-axis mill
- Work envelope: 9.52" x 3.50" x 3.11" (241.8 x 88.9 x 79.0 mm)
- Max spindle speed: 9000 RPM
- Collet: ER-11 (max tool diameter ~8mm / 5/16")
- Post processor: GGD Ghost Gunner 3 (preferred) or generic Grbl

### Alternative: FreeCAD Path Workbench

Since you're already using FreeCAD for modeling, this keeps everything in one tool.

- **Free and open source**
- Built-in Path (CAM) workbench
- Import STEP natively, STL with mesh-to-solid conversion
- Supports Grbl post processor output
- Less polished CAM than Fusion 360, but capable
- You will likely need to use the **LinuxCNC/Grbl post processor** and may need minor manual edits to the output

**FreeCAD Path setup:**
1. Open your model in FreeCAD
2. Switch to Path workbench
3. Create a Job (define stock, set machine limits)
4. Add operations (Profile, Pocket, Drilling, etc.)
5. Post-process with `grbl` or `linuxcnc` post processor
6. Review/edit output G-code for GG3 compatibility

**Caveats with FreeCAD:**
- STL files need conversion to solid (Part > Shape from Mesh > Refine)
- STEP files work better as CAM input (prefer STEP for your designs)
- The Path workbench has a learning curve
- Test G-code in DDCut's Manual Mode first with no workpiece

### Other Options

| Software | License | Notes |
|----------|---------|-------|
| **Fusion 360** | Free (personal) | Best GG3 support, dedicated post processor |
| **FreeCAD** | Free/OSS | You already use it; Path workbench works |
| **CamBam** | ~$150 | Lightweight 2.5D CAM, Grbl-compatible output |
| **Carbide Create** | Free | Simple 2D/2.5D CAM, Grbl post available |
| **Estlcam** | ~$60 | Good for simple 2.5D work, Grbl output |
| **OpenCAM** | Free/OSS | Basic CAM, requires manual post-processing |
| **HSMWorks** | Paid | Professional, Grbl post available |

### Workflow Summary: STL/STEP to .DD File

```
+----------------+     +------------------+     +------------------+
| FreeCAD        | --> | Fusion 360 or    | --> | G-code files     |
| (Design in     |     | FreeCAD Path     |     | (.gcode/.nc)     |
|  STEP format)  |     | (Generate CAM    |     |                  |
+----------------+     |  toolpaths, post |     +--------+---------+
                        |  with Grbl/GG3)  |              |
                        +------------------+              v
                                               +------------------+
                                               | Write manifest   |
                                               | .yml + add images|
                                               +--------+---------+
                                                        |
                                                        v
                                               +------------------+
                                               | ZIP everything   |
                                               | rename to .dd    |
                                               +------------------+
                                                        |
                                                        v
                                               +------------------+
                                               | Load in DDCut    |
                                               | (or Manual Mode) |
                                               +------------------+
```

### FreeCAD-Specific Recommendation

Since you model in FreeCAD:
1. **Export as STEP** (not STL) for CAM input -- STEP preserves exact geometry
2. If staying in FreeCAD: use the Path workbench with Grbl post
3. If using Fusion 360 for CAM: import the STEP file, do CAM there, post with GG3 post processor
4. STL is a mesh format and less ideal for CNC toolpath generation; always prefer STEP/BREP

---

## GG3 Machine Constraints (for CAM Setup)

| Parameter | Value |
|-----------|-------|
| Axes | 3-axis (X, Y, Z) |
| Work envelope | 9.52" x 3.50" x 3.11" (241.8 x 88.9 x 79.0 mm) |
| Spindle | ER-11 collet, 9000 RPM max |
| Max tool diameter | ~8mm (5/16") |
| Spindle orientation | Horizontal |
| Firmware | Grbl-based (custom motion control board) |
| Control software | DDCut (runs .dd files or Manual Mode for raw G-code) |
| Probe | Electronic touch probe (G38.2 commands) |
| Materials | Aluminum (primary), steel (with appropriate tooling/speeds) |

---

## Quick Reference: Building Your First Custom .DD

```bash
# 1. Create project structure
mkdir -p my-custom-job/{Code,Image}

# 2. Put your G-code files in Code/
# 3. Put instructional images in Image/
# 4. Write manifest.yml (see Minimal Working Example above)

# 5. Package
cd my-custom-job
zip -r ../my-custom-job.dd manifest.yml Code/ Image/

# 6. Copy my-custom-job.dd to USB and load in DDCut
```
