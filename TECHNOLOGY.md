# Technology Reference

A precise description of every technology, library, and tool used in the FolderSco project.

---

## Language

### Python 3
Both source files (`foldersco.py` and `Foldersco2.0.py`) are written in pure Python 3.  
The project was developed and tested on **Python 3.14.5** and is compatible with Python 3.8+.

No compiled extensions or C extensions are written directly — all native Python.

---

## External Dependencies

### graphviz (Python package)
**Install:** `pip install graphviz`  
**Version tested:** 0.21  
**Purpose:** Python wrapper around the Graphviz command-line tools.

Used for:
- Creating a `graphviz.Digraph` object that represents the folder tree
- Configuring global graph, node, and edge attributes (layout direction, DPI, font, colours, etc.)
- Calling `dot.render()` to invoke the `dot.exe` layout engine and produce output files
- Calling `graphviz.version()` to verify that `dot.exe` is on PATH before scanning

The Python package does *not* include the Graphviz executables — it is a thin wrapper that calls the separately-installed Graphviz application via subprocess.

### Graphviz application (`dot.exe`)
**Install:** [graphviz.org/download](https://graphviz.org/download/)  
**Version tested:** 16.1.0  
**Purpose:** The actual graph rendering engine.

`dot` is the specific layout engine used. It performs hierarchical (tree-style) layout, which is ideal for directory structures. The `dot` engine is invoked automatically by the Python `graphviz` package when `dot.render()` is called.

`dot.exe` is required for SVG, PNG, and PDF output. It is **not** required to export DOT source (`.dot`) files.

### tkinterdnd2 (optional)
**Install:** `pip install tkinterdnd2`  
**Purpose:** Enables drag-and-drop of folders onto the path field in the GUI.

Optional — detected at runtime with a `try/except` import. If not installed, `HAS_DND = False` and the drag-and-drop registration is simply skipped. All other GUI features continue to work without it.

---

## Standard Library Modules

These are part of Python's standard library — no installation required.

| Module | Used for |
|--------|----------|
| `os` | Reading the `USERPROFILE` environment variable to locate the Desktop |
| `sys` | Exiting cleanly (`sys.exit`) when a critical dependency is missing |
| `time` | Measuring elapsed time for the scan + render pipeline (`time.time()`) |
| `threading` | Running the scan + graph build + render on a background daemon thread in the GUI, keeping the window responsive |
| `pathlib.Path` | All filesystem operations — path existence, directory creation, name extraction, iteration |
| `datetime.datetime` | Generating the `YYYYMMDD_HHMM` timestamp used in auto-generated filenames |
| `tkinter` | The GUI framework — windows, buttons, labels, entry fields, layout managers (`pack`, `grid`) |
| `tkinter.ttk` | Themed widget set — `ttk.Radiobutton`, `ttk.Checkbutton`, `ttk.Progressbar`, `ttk.Entry`, `ttk.Button` |
| `tkinter.filedialog` | Native Windows folder browser dialog (`filedialog.askdirectory`) |
| `tkinter.messagebox` | Native Windows message and error dialogs |

---

## GUI Framework

### tkinter + ttk
FolderSco 2.0's GUI is built entirely with Python's built-in `tkinter` library and its `ttk` (themed) extension. No third-party GUI framework is used.

Key characteristics:
- **Layout:** `pack` geometry manager is used for the main vertical layout; `grid` is used for the stats table inside the progress panel
- **Custom styling:** The Generate button uses a `tk.Button` with explicit `bg`, `fg`, and `activebackground` colours. Other buttons use `ttk.Button` for the native Windows appearance
- **Progress bar:** `ttk.Progressbar(mode='indeterminate')` — bouncing animation used because total file count cannot be predicted before the scan completes
- **Threading model:** The GUI never calls tkinter widgets from the worker thread. Instead, `root.after(100, self._poll)` polls shared state (a plain Python dict) every 100 ms from the main thread, then updates labels and the progress bar safely
- **Font:** `Segoe UI` — the native Windows system font

---

## Graph Engine

### Graphviz `dot` layout
The `dot` layout engine places nodes in a ranked hierarchy (top-to-bottom or left-to-right). FolderSco uses **left-to-right** (`rankdir=LR`) so the tree extends horizontally from the root folder outward.

Graph properties configured by FolderSco:
- `rankdir = "LR"` — horizontal tree
- `dpi = "150"` — render resolution
- `bgcolor = "#F0F2F5"` — light neutral canvas
- `nodesep = "0.3"` — vertical gap between sibling nodes
- `ranksep = "0.6"` — horizontal gap between parent and child ranks
- Node `shape = "box"`, `style = "filled,rounded"`, `fontname = "Consolas"`, `fontsize = "11"`
- Directed graph (`graphviz.Digraph`) — arrows flow from parent to child

---

## Build Tool

### PyInstaller 6.22.3
**Install:** `pip install pyinstaller`  
**Purpose:** Packages `Foldersco2.0.py` and all its Python dependencies into a single self-contained Windows executable.

Build flags used:
- `--onefile` — bundle everything into one `.exe` file
- `--noconsole` — suppress the terminal window (GUI-only app)
- `--name "FolderSco2.0"` — output filename
- `--clean` — clear previous build cache

The resulting `dist\FolderSco2.0.exe` (~11 MB) bundles Python 3.14.5 and all imported standard library modules. The Graphviz application (`dot.exe`) is **not** bundled — it must be installed separately on any machine running the EXE.

---

## File Formats Produced

| Format | Produced by | Notes |
|--------|-------------|-------|
| `.svg` | `dot.render(format="svg")` | Vector; default for Proto 1 |
| `.png` | `dot.render(format="png")` | Raster; available in GUI only |
| `.pdf` | `dot.render(format="pdf")` | PDF; available in GUI only |
| `.dot` | `Path.write_text(dot.source)` | Raw DOT source; no rendering required |

---

## Development Environment (tested on)

| Component | Version |
|-----------|---------|
| OS | Windows 11 |
| Python | 3.14.5 |
| graphviz (Python) | 0.21 |
| Graphviz application | 16.1.0 |
| PyInstaller | 6.22.3 |
