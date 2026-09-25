# Code Architecture & Explanation

A precise walkthrough of how FolderSco works internally — function by function, section by section. References are accurate to the actual source code in `foldersco.py` (Proto 1) and `Foldersco2.0.py` (GUI).

---

## Overview

Both files share the same backend pipeline:

```
User input
    │
    ▼
scan_directory()          ← recursive filesystem walk → nested dict tree
    │
    ▼
build_graph()             ← dict tree → graphviz.Digraph object
    │
    ▼
render_graph()            ← Digraph → calls dot.exe → output file
    │
    ▼
Output file saved to Desktop\FileTreeGen\
```

The GUI (`Foldersco2.0.py`) wraps this same pipeline inside a tkinter window and runs it on a background thread.

---

## Section 1 — Configuration

Both files open with a flat configuration block of module-level constants. These are the only values a user would ever need to change to customise behaviour:

```python
OUTPUT_FOLDER_NAME = "FileTreeGen"   # sub-folder name created on the Desktop
MAX_NAME_LENGTH    = 50              # max chars before a label is truncated with "..."

NODE_FONT     = "Consolas"           # font used inside every node box
NODE_FONTSIZE = "11"                 # font size in points
GRAPH_DPI     = "150"                # render resolution
GRAPH_RANKDIR = "LR"                 # left-to-right tree layout

FILE_FILL = "#FFFFFF"                # file nodes use white fill

DEPTH_COLORS = [...]                 # list of 10 dicts, one per depth level
```

`DEPTH_COLORS` is a Python list of 10 dictionaries. Each dict has three keys:

```python
{"fill": "#DCEBFF", "font": "#1A3560", "border": "#5A90D0"}
```

- `fill` — background colour of folder nodes at this depth
- `font` — text colour inside the node
- `border` — outline colour, also used to colour the edges originating from this level

---

## Section 2 — Imports

`foldersco.py` uses: `os`, `sys`, `time`, `pathlib.Path`, `datetime`, and `graphviz`.

`Foldersco2.0.py` adds: `threading`, `tkinter`, `tkinter.ttk`, `tkinter.filedialog`, `tkinter.messagebox`, and optionally `tkinterdnd2`.

Both files guard the `import graphviz` with a `try/except ImportError` block that prints (terminal) or shows a dialog (GUI) with installation instructions, then calls `sys.exit(1)`.

`Foldersco2.0.py` additionally does a `try/except ImportError` for `tkinterdnd2` and sets `HAS_DND = True/False`, enabling or skipping drag-and-drop registration without crashing.

---

## Section 3 — Path and Output Setup

### `get_output_directory() → Path`

Locates the output folder on the user's Desktop without any hardcoded paths:

```python
home    = Path(os.environ.get("USERPROFILE", str(Path.home())))
desktop = home / "Desktop"
output_dir = desktop / OUTPUT_FOLDER_NAME
output_dir.mkdir(parents=True, exist_ok=True)
```

- Uses `USERPROFILE` environment variable (standard on all modern Windows versions)
- Falls back to `Path.home()` if `USERPROFILE` is missing
- Creates `Desktop\FileTreeGen\` if it does not already exist (`parents=True, exist_ok=True`)
- Returns a `Path` object pointing to the output directory

### `build_output_stem(root_name, output_dir, [fmt, custom_name]) → Path`

Generates the output file path stem (filename without extension):

- Auto mode: `FolderName_20260926_0012` (folder name + timestamp)
- Custom mode (GUI only): uses the name the user typed, stripping any extension suffix
- If the target file already exists (same minute), appends `_1`, `_2`, etc. to prevent overwriting
- Returns a `Path` object — the extension is appended later by `render_graph()`

In `foldersco.py` (Proto 1), the duplicate check uses `.svg`. In `Foldersco2.0.py`, it uses the chosen format extension.

---

## Section 4 — Directory Scanning

### `scan_directory(root_path, [on_progress]) → dict`

The core recursive scanner. Walks the entire filesystem tree from `root_path` and returns a single nested Python dictionary representing the full structure.

**Node schema:**

```python
{
    "name":     "src",          # filename or folder name
    "type":     "dir",          # "dir" or "file"
    "children": [...]           # list of child node dicts (dirs only)
}
```

**How it works — `build_node(path)` inner function:**

1. Creates a node dict with `name` and `type`
2. If the path is a directory:
   - Sets `children = []`
   - Calls `path.iterdir()` to list all entries
   - Sorts them: **subdirectories first, then files** — both groups alphabetically
   - Recursively calls `build_node(entry)` for each child and appends to `children`
   - Catches `PermissionError` per directory (terminal: prints a warning; GUI: silently skips)
3. If the path is a file: returns the node with no `children` key
4. Calls `on_progress("dir")` or `on_progress("file")` at each node if the callback is provided (GUI only — used to update the live file/folder counters)

**Key design choice:** Node IDs in the graph are assigned as plain incrementing integers (not filenames). This means filenames containing spaces, dots, brackets, quotes, or any other special characters cannot break the DOT syntax.

### `count_nodes(tree) → (folder_count, file_count)`

A simple post-scan walk of the tree dict that counts all folders and files. Subtracts 1 from the folder count because the root itself is not counted as a child. Used only in `foldersco.py` (terminal) to print the summary line.

---

## Section 5 — Graph Building

### `truncate(name) → str`

Returns the name unchanged if it is 50 characters or fewer. Otherwise returns the first 47 characters followed by `"..."`. This keeps node boxes compact in the output image.

### `build_graph(tree) → graphviz.Digraph`

Converts the nested dict tree from `scan_directory()` into a `graphviz.Digraph` object ready for rendering.

**Graph-level attributes set once:**

```python
dot.attr(rankdir="LR", dpi="150", bgcolor="#F0F2F5", pad="0.5",
         nodesep="0.3", ranksep="0.6")
dot.attr("node", shape="box", style="filled,rounded",
         fontname="Consolas", fontsize="11", margin="0.15,0.08")
dot.attr("edge", arrowsize="0.7", penwidth="1.0")
```

**`add_nodes(node, parent_id, depth)` — inner recursive function:**

For each node in the tree:

1. Assigns the next integer ID from `id_counter[0]` and increments the counter
2. Truncates the name with `truncate()`
3. Selects the colour palette for this depth: `DEPTH_COLORS[depth % 10]`
4. **Folder node:** `fillcolor = palette["fill"]`, `fontcolor = palette["font"]`, `color = palette["border"]`, `penwidth = "1.8"`, label prefixed with `"[D]  "`
5. **File node:** `fillcolor = "#FFFFFF"` (white), `fontcolor = palette["font"]`, `color = palette["border"]`, `penwidth = "1.0"`, label prefixed with `"[F]  "`
6. If `parent_id` is not `None`: draws a directed edge from `parent_id → node_id`. The edge colour comes from the **parent's** depth palette (`DEPTH_COLORS[(depth-1) % 10]["border"]`), not the current depth
7. Recurses into `node.get("children", [])` with `depth + 1`

The root node is called with `parent_id=None` and `depth=0`, so it gets no incoming edge and uses the depth-0 palette (Light Blue).

---

## Section 6 — Image Generation

### `check_graphviz_executable() → bool`

Calls `graphviz.version()`, which internally runs `dot -V`. If `dot.exe` is not on PATH, the Python graphviz package raises `graphviz.ExecutableNotFound`.

- **Terminal (`foldersco.py`):** Prints a step-by-step Windows installation guide to stdout and returns `False`. `main()` calls `sys.exit(1)` on `False`.
- **GUI (`Foldersco2.0.py`):** Called 200 ms after startup via `root.after(200, ...)`. Shows a `messagebox.showwarning()` dialog. Also called before generation if the chosen format is not DOT.

### `render_graph(dot, output_stem, [fmt]) → Path`

Invokes Graphviz to write the output file.

**`foldersco.py` (Proto 1 — SVG only):**

```python
dot.render(filename=str(output_stem), format="svg", engine="dot", cleanup=True)
return output_stem.parent / (output_stem.name + ".svg")
```

**`Foldersco2.0.py` (all formats):**

```python
if fmt.lower() == "dot":
    out_path = output_stem.parent / (output_stem.name + ".dot")
    out_path.write_text(dot.source, encoding="utf-8")
    return out_path

dot.render(filename=str(output_stem), format=fmt.lower(), engine="dot", cleanup=True)
return output_stem.parent / (output_stem.name + f".{fmt.lower()}")
```

- `cleanup=True` deletes the intermediate `.gv` source file after rendering, leaving only the final output file
- DOT format writes `dot.source` (the DOT language string) directly to disk via `Path.write_text()` — no subprocess call, no `dot.exe` required

---

## Section 7 — Processing and Timing

### Terminal (`foldersco.py`) — `scan_one(folder_path)`

Wraps the full pipeline for one scan run:

1. Validates the path (exists? is a directory?)
2. Records `start_time = time.time()`
3. Calls `scan_directory()` → `count_nodes()` → prints folder/file counts
4. Calls `build_graph()` → `render_graph()`
5. Prints `Done. (X.XXs)` and the output path
6. Returns the output `Path` on success, `None` on failure

Elapsed time is `time.time() - start_time` measured across the full scan + build + render sequence.

### GUI (`Foldersco2.0.py`) — `_worker()` + `_poll()`

The GUI uses Python's `threading.Thread` to keep the window responsive during generation.

**`_worker(folder_path, output_stem, fmt)` — runs on a daemon thread:**

```python
state["stage"] = "Scanning files…"
tree = scan_directory(folder_path, on_progress=on_progress)

state["stage"] = "Building graph…"
dot_graph = build_graph(tree)

state["stage"] = f"Rendering {fmt}…"
out_path = render_graph(dot_graph, output_stem, fmt)

state["done"] = True
```

The `on_progress` callback updates `state["files"]` and `state["folders"]` as each node is discovered during the scan.

**`_poll()` — runs on the main (UI) thread every 100 ms:**

```python
self.root.after(100, self._poll)
```

Reads `_job_state` (a plain Python dict) and updates:
- `self.stage_lbl` — current stage string
- `self.lbl_files` — formatted file count
- `self.lbl_folders` — formatted folder count
- `self.lbl_elapsed` — `MM:SS` elapsed time computed as `time.time() - state["start_time"]`

When `state["done"]` becomes `True`, `_poll()` calls `_on_complete()` and stops re-scheduling itself.

The progress bar (`ttk.Progressbar`) runs in `mode="indeterminate"` (bouncing animation) because the total number of files cannot be known before the scan completes — there is no fake percentage.

---

## Section 8 — User Interface (GUI only)

### Application class: `FolderScoApp`

All GUI logic lives in a single class. The `__init__` method sets up:

- `tk.Tk()` or `TkinterDnD.Tk()` root window (600×720 px, centred on screen)
- Tkinter variable objects: `folder_var`, `outdir_var`, `filename_var`, `autoname_var`, `format_var`
- Calls `_build_ui()` to construct all widgets
- Schedules the Graphviz preflight check 200 ms after startup

### `_build_ui()`

Constructs the entire widget tree using `pack` layout. Key sections:

| Widget section | Widgets used |
|----------------|--------------|
| Header bar | `tk.Frame` (blue `bg`), two `tk.Label` |
| Input folder | `tk.Entry` (path), `tk.Button` (Browse) |
| Output format | `ttk.Radiobutton` × 4 (SVG / PNG / PDF / DOT) |
| Output location | `tk.Entry` (dir), `tk.Button` (…), `ttk.Checkbutton` (auto), `tk.Entry` (filename) |
| Generate button | `tk.Button` with custom blue `bg`, hover bindings |
| Progress panel | `tk.Frame`, `tk.Label` (stage), `tk.Label` × 3 (stats), `ttk.Progressbar` |
| Results panel | `tk.Frame`, `tk.Label` (icon, title, detail), `ttk.Button` × 3 |

### Placeholder text

The path entry field shows `"Paste a folder path, or click Browse…"` in grey when empty. `<FocusIn>` clears it, `<FocusOut>` restores it if the field is still empty. This is implemented with `bind()` callbacks — not a built-in tkinter feature.

### State transitions

| State | `prog_frame` | `result_frame` | `gen_btn` |
|-------|-------------|----------------|-----------|
| Idle | hidden | hidden | enabled (blue) |
| Generating | visible | hidden | disabled (grey) |
| Complete (success) | hidden | visible (green) | enabled (blue) |
| Complete (error) | hidden | visible (red) | enabled (blue) |

### `_reset()`

Clears the folder path entry, resets the filename field, restores default output directory, sets format back to SVG, re-enables auto-filename, and hides the results panel. The generated output file on disk is **not** deleted.

---

## Important Functions — Quick Reference

| Function | File | Purpose |
|----------|------|---------|
| `get_output_directory()` | both | Locate `Desktop\FileTreeGen\` dynamically |
| `build_output_stem()` | both | Generate unique output filename with timestamp |
| `scan_directory()` | both | Recursive filesystem walk → nested dict |
| `count_nodes()` | proto 1 | Count folders and files from the tree dict |
| `truncate()` | both | Shorten long filenames to 50 chars |
| `build_graph()` | both | Convert tree dict → `graphviz.Digraph` |
| `check_graphviz_executable()` | both | Verify `dot.exe` is on PATH |
| `render_graph()` | both | Invoke `dot.exe` to produce the output file |
| `scan_one()` | proto 1 | Run full pipeline for one path (terminal) |
| `main()` | proto 1 | Banner + preflight + `while True` input loop |
| `FolderScoApp.__init__()` | 2.0 | GUI setup and variable initialisation |
| `FolderScoApp._build_ui()` | 2.0 | Construct all tkinter widgets |
| `FolderScoApp._start_generation()` | 2.0 | Validate inputs and launch worker thread |
| `FolderScoApp._worker()` | 2.0 | Background thread: scan → build → render |
| `FolderScoApp._poll()` | 2.0 | 100 ms main-thread UI updater |
| `FolderScoApp._on_complete()` | 2.0 | Show results panel after success |
| `FolderScoApp._on_error()` | 2.0 | Show error panel after failure |
| `FolderScoApp._reset()` | 2.0 | Clear form for next scan |
