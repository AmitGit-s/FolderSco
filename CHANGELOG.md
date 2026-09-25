# Changelog

All notable changes to FolderSco are documented in this file.

The format follows [Keep a Changelog](https://keepachangelog.com/en/1.0.0/).  
Versions follow [Semantic Versioning](https://semver.org/).

---

## [2.0.0] — 2026-09-26

### Added — GUI Edition (`Foldersco2.0.py`)

- **Full tkinter GUI dashboard** — replaces terminal input with a graphical window
- **Folder input via Browse dialog** — native Windows folder picker using `filedialog.askdirectory()`
- **Drag-and-drop folder input** — drop a folder onto the path field (requires `tkinterdnd2`; gracefully disabled if not installed)
- **Paste path input** — text entry field with placeholder text that clears on focus
- **Output format selection** — radio buttons for SVG, PNG, PDF, and DOT formats
  - SVG, PNG, PDF rendered via Graphviz `dot.exe`
  - DOT format writes the raw `.dot` source file directly (no `dot.exe` required)
- **Output location control** — directory field with Browse button; defaults to `Desktop\FileTreeGen\`
- **Custom filename input** — text field for a user-defined output filename
- **Auto-generate filename** — checkbox (on by default); generates `FolderName_YYYYMMDD_HHMM.ext`
- **Live progress monitor** — during generation, displays in real time:
  - Current stage (Scanning / Building graph / Rendering)
  - Files scanned (formatted with thousands separator)
  - Folders scanned (formatted with thousands separator)
  - Elapsed time in `MM:SS` format
  - Indeterminate progress bar (bouncing animation — no fake percentage)
- **Threaded generation** — scan, graph build, and render run on a background `threading.Thread`; the UI thread polls shared state every 100 ms via `root.after()` so the window never shows "Not Responding"
- **Completion panel** — shown after successful generation:
  - ✓ success icon with green text
  - Summary: total files, total folders, elapsed time, output filename
  - **Open File** button — opens the generated file with the system default application
  - **Open Folder** button — opens the output directory in Windows Explorer
  - **↺ New Scan** button — clears the form for the next generation
- **Error panel** — shown if generation fails; displays the error message with a red ✗ icon
- **Graphviz preflight check** — 200 ms after startup, verifies `dot.exe` is on PATH; shows a warning dialog with installation instructions if not found
- **Drive root name fix** — paths like `C:\` (empty `.name` component) automatically use `C_Drive` as the filename prefix
- **Compiled EXE** — `FolderSco2.0.exe` (~11 MB) built with PyInstaller 6.22.3 (`--onefile --noconsole`); self-contained, no Python installation required on the target machine

### Unchanged from Proto 1

- Recursive directory scanning logic (`scan_directory`)
- Graphviz graph construction (`build_graph`)
- 10-level depth colour palette and cycling logic
- White file node fill distinguishing files from folders
- Depth-coloured edges (tinted to match parent depth's border colour)
- Output path detection via `USERPROFILE` environment variable
- `Desktop\FileTreeGen\` default output location
- Unique filename generation with counter suffix to prevent overwrites
- `PermissionError` handling per directory (skipped silently in GUI)
- Node label truncation at 50 characters

---

## [1.0.0] — 2026-09-25

### Added — Terminal Prototype (`foldersco.py`)

- **Recursive folder scanning** — walks any folder path and builds a complete nested dict tree of all subfolders and files
- **Graphviz Digraph generation** — converts the tree dict into a `graphviz.Digraph` object using the `dot` layout engine
- **SVG output** — renders to scalable vector graphics via `dot.render(format="svg")`
- **10-level depth colour palette** — each tree depth level assigned a distinct pastel colour from a 10-entry list that cycles for deeper trees:
  - Level 0: Light Blue (`#DCEBFF`)
  - Level 1: Mint (`#DDF5E3`)
  - Level 2: Soft Yellow (`#FFF1CC`)
  - Level 3: Soft Pink (`#FCE1E4`)
  - Level 4: Lavender (`#E8DEFF`)
  - Level 5: Aqua (`#DDF4F4`)
  - Level 6: Peach (`#FFE4CC`)
  - Level 7: Periwinkle (`#E5E5F5`)
  - Level 8: Light Green (`#E6F4D7`)
  - Level 9: Soft Purple (`#F5E1F5`)
- **Folder vs. file visual distinction** — folder nodes carry the full pastel fill; file nodes use white (`#FFFFFF`) with the depth's border and font colour
- **Depth-coloured edges** — arrows coloured to match the parent node's depth border colour
- **Left-to-right layout** (`rankdir=LR`) — tree extends horizontally from root
- **Light canvas background** (`#F0F2F5`) — neutral background that lets pastel nodes stand out
- **Sorted directory listing** — subdirectories listed before files within each folder, both groups alphabetically
- **PermissionError handling** — inaccessible folders are skipped with a printed warning; the scan continues
- **Auto output location** — output saved to `Desktop\FileTreeGen\<FolderName>_<YYYYMMDD_HHMM>.svg`; directory created automatically
- **Unique filename generation** — counter suffix (`_1`, `_2`, …) prevents overwriting files generated within the same minute
- **Node label truncation** — names longer than 50 characters are shortened with `...`
- **Integer node IDs** — nodes are identified by incrementing integers, not filenames, so special characters in paths cannot break DOT syntax
- **Elapsed time reporting** — prints total time for scan + build + render in seconds
- **`while True` scan loop** — after each scan, offers `[1] Scan another folder` / `[2] Exit`
- **`exit`/`quit`/`q` shortcut** — accepted at the folder path prompt to quit without scanning
- **Graphviz preflight check** — verifies `dot.exe` is on PATH before entering the loop; prints a step-by-step Windows installation guide and exits if missing
- **Dynamic path detection** — uses `USERPROFILE` environment variable; no hardcoded usernames or drive letters anywhere in the code
- **Graphviz import guard** — `try/except ImportError` prints installation instructions and exits cleanly if the Python package is missing
