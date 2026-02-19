# FreeCAD 1.0+ CAM Workbench Guide for Ghost Gunner G-Code

## Prerequisites

- FreeCAD 1.0 or newer (the "Path" workbench was renamed to "CAM" in 1.0)
- Your part modeled or imported as a STEP file
- Know your tool dimensions (the GG3 uses ER-11 collets, max ~8mm/5/16" tool diameter)

## Quick Reference: GG3 Machine Parameters

Use these when setting up your CAM Job:

| Parameter | Value |
|-----------|-------|
| Machine type | 3-axis mill |
| Work envelope | 241.8 x 88.9 x 79.0 mm (9.52" x 3.50" x 3.11") |
| Max spindle RPM | 9000 |
| Max tool diameter | ~8mm (5/16") |
| Collet | ER-11 |
| Post processor | grbl |
| Units | Millimeters preferred (G21) |

---

## Step-by-Step Workflow

### 1. Open/Import Your Model

If your part is already a FreeCAD file, open it directly.

To import a STEP file:
- **File > Open** or **File > Import**
- Select your `.step` / `.stp` file
- The solid body appears in the model tree

If you have an STL instead, you need to convert it first:
- **Part > Shape from Mesh** (set tolerance ~0.1mm)
- **Part > Convert to Solid**
- **Part > Refine Shape**
- Use the refined solid as your CAM base (STEP is always preferred over STL)

### 2. Switch to CAM Workbench

- Use the **workbench selector dropdown** (top of window) and select **CAM**
- The toolbar changes to show CAM-specific tools

### 3. Create a Job

This is the container for your entire machining project.

- **CAM > Job** from the menu (or press `P` then `J`, or click the Job toolbar button)
- In the dialog, select your Body/Part as the **Base Model**
- Leave Template as "None" for a fresh setup
- Click **OK**

A wireframe box (the stock) appears around your model.

### 4. Configure the Job

The Job Edit panel opens with several tabs:

#### Setup Tab
- **Stock**: Defines your raw material size
  - Set the bounding box offsets to match your actual stock
  - Example: if your stock is 25.4mm x 25.4mm x 19.05mm (1"x1"x3/4"), set the Create Box dimensions or add offsets to the model's bounding box
  - Set the **origin corner** to match where you'll zero the machine

#### Output Tab
- **Postprocessor**: Select **`grbl`** from the dropdown
  - This is FreeCAD's built-in Grbl post processor, compatible with the GG3
  - If `grbl` doesn't appear, check Edit > Preferences > CAM for post processor paths
- **Output file**: Set where the .gcode file will be saved
- **Post Processor Arguments**: Leave blank for defaults (metric mm output)
  - Add `--inches` if you need G20 imperial output instead

#### Tools Tab (covered in next step)

### 5. Set Up Your Cutting Tools

You need to define the tools that match what you physically have for the GG3.

#### Create a Tool in the Library

1. In the Job Edit dialog, go to the **Tools** tab
2. Click the **Default** tool entry, then click **Edit** to open the Tool Controller
3. Alternatively, use **CAM > ToolBit Library Editor** to manage a reusable library

#### Tool Controller Settings

For each tool, configure:

| Parameter | Example (1/4" end mill) | Example (1/8" end mill) |
|-----------|------------------------|------------------------|
| Tool Number | 1 | 2 |
| Diameter | 6.35 mm (0.250") | 3.175 mm (0.125") |
| Flutes | 3 or 4 | 2 or 4 |
| Shape | EndMill | EndMill |
| Horizontal Feed | 100-300 mm/min | 50-200 mm/min |
| Vertical Feed | 50-100 mm/min | 25-75 mm/min |
| Spindle RPM | 5000-8000 | 5000-8000 |

**Feeds and speeds depend on material.** For 6061 aluminum on the GG3, conservative starting points:
- 1/4" end mill: ~200 mm/min feed, ~50 mm/min plunge, 6000 RPM, 0.5mm depth per pass
- 1/8" end mill: ~100 mm/min feed, ~30 mm/min plunge, 7000 RPM, 0.3mm depth per pass

These are conservative. Adjust based on results -- listen for chatter, check chip formation.

### 6. Create Machining Operations

Select faces/edges on your model, then apply operations. The main ones you'll use:

#### Profile (Contour Cutting)

Cuts along an edge or around a face outline.

1. Select the edge(s) or face(s) to profile
2. **CAM > Profile** (toolbar button)
3. Configure in the panel:
   - **Base Geometry**: Shows selected features
   - **Depths**:
     - Start Depth: Top of stock (usually 0)
     - Final Depth: How deep to cut
     - Step Down: Depth per pass (e.g., 0.5 mm for aluminum)
   - **Heights**:
     - Clearance: Safe height above stock for rapids
     - Safe: Height for moves between cuts
   - **Operation**:
     - Cut Side: Inside, Outside, or On the line
     - Direction: Conventional or Climb milling
4. Click **OK**

A green toolpath line appears on your model.

#### Pocket (Area Clearing)

Clears material from an enclosed area (pocket, cavity).

1. Select the bottom face of the pocket
2. **CAM > Pocket Shape** (toolbar button)
3. Configure:
   - **Depths**: Same as Profile
   - **Operation**:
     - Step Over: Percentage of tool diameter per pass (50% is typical)
     - Pattern: **Offset** (spiral inward -- good general choice) or **ZigZag**
     - Cut Mode: Conventional or Climb
4. Click **OK**

#### Face (Surface Flattening)

Flattens the top of your stock.

1. Select the top face
2. **CAM > Face**
3. Set depths (usually just skim the surface)

#### Drilling

For making holes.

1. Select circular edges or hole features
2. **CAM > Drilling**
3. Set peck depth if deep drilling
4. The Grbl post processor converts G81/G82/G83 drill cycles to basic G0/G1 moves (Grbl doesn't support canned cycles)

#### Adaptive Clearing

More efficient roughing than Pocket -- varies engagement to maintain consistent tool load.

1. Select features
2. **CAM > Adaptive**
3. Good for removing large volumes of material quickly

### 7. Operation Order

Operations execute top-to-bottom in the model tree. Drag to reorder. Typical sequence:

1. **Face** (flatten top if needed)
2. **Adaptive** or **Pocket** (rough out cavities)
3. **Profile** (cut outlines/contours)
4. **Drilling** (holes last, so material is stable)

### 8. Add Holding Tabs (if cutting parts free)

If your profile operation cuts a part free from the stock, add tabs to hold it:

1. Select your Profile operation
2. **CAM > Path Dressup > Tag**
3. Auto-generates 4 tabs -- adjust size and position
4. You'll file/sand these off after milling

### 9. Simulate

Before exporting, verify your toolpaths:

1. Select the Job in the model tree
2. **CAM > CAM Simulator** (or **SimulatorGL** in FreeCAD 1.0+)
3. Adjust speed slider, click **Play**
4. Watch the virtual stock get cut -- check for:
   - Gouging (cutting where it shouldn't)
   - Missed areas
   - Collisions
5. Click **Cancel** to discard simulation, or **OK** to keep the preview solid

### 10. Post-Process to G-Code

1. Select the **Job** object in the model tree
2. **CAM > Post Process** (toolbar button)
3. A G-code preview window opens -- review the output
4. Confirm and save the `.gcode` file

The output will contain standard Grbl-compatible G-code ready for the GG3.

### 11. Use the G-Code

You have two options to run it on the Ghost Gunner:

**Option A: DDCut Manual Mode**
- Open DDCut
- Use Manual Mode to load and run your .gcode file directly
- Good for testing / one-off jobs

**Option B: Package as .DD File**
- Follow the cheat sheet in `dd-file-cheat-sheet.md`
- Create a manifest.yml with step-by-step instructions
- Add your .gcode files and instructional images
- ZIP and rename to .dd
- Load in DDCut as a guided job

---

## Common Pitfalls

**Wrong post processor**: If you use LinuxCNC post instead of Grbl, the output may contain commands the GG3 doesn't understand (canned cycles, tool change macros). Always use `grbl`.

**Tool too large**: The GG3 maxes out at ~8mm tool diameter with ER-11 collets. CAM won't stop you from defining a 12mm tool.

**Exceeding work envelope**: The GG3 has a small work area (241.8 x 88.9 x 79.0 mm). Verify your part + stock fits within these bounds.

**Aggressive feeds/speeds**: The GG3 is a desktop mill, not an industrial machine. Start conservative and increase gradually. Broken tools and crashed spindles are expensive mistakes.

**Forgetting to set work zero**: Your CAM Job origin must match where you'll zero the machine. The GG3 uses probing to set work coordinates -- your CAM origin should correspond to the probe reference point.

**STL instead of STEP**: STL is a mesh approximation. Toolpaths generated from STL geometry may have faceting artifacts. Always prefer STEP for CAM input.

---

## FreeCAD CAM Quick Cheat Sheet

| Action | Menu / Shortcut |
|--------|----------------|
| Switch to CAM workbench | Workbench dropdown > CAM |
| Create Job | CAM > Job (or `P`, `J`) |
| Add Profile operation | CAM > Profile |
| Add Pocket operation | CAM > Pocket Shape |
| Add Face operation | CAM > Face |
| Add Drilling | CAM > Drilling |
| Add Adaptive clearing | CAM > Adaptive |
| Add holding tabs | CAM > Path Dressup > Tag |
| Inspect G-code | CAM > Inspect |
| Simulate | CAM > CAM Simulator |
| Export G-code | CAM > Post Process |
| Tool library | CAM > ToolBit Library Editor |
| Preferences | Edit > Preferences > CAM |
