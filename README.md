# FolderSco

**FolderSco** is a local, offline Windows utility that scans any folder on your computer and generates a visual diagram of its complete structure — every subfolder and file, arranged in a clear, colour-coded tree graph.

No internet. No cloud. No accounts. Point it at a folder, click Generate.

---

## ✨ Features

- **Full recursive scan** — walks every subfolder and file from the chosen root
- **Graph-based visualization** — uses Graphviz `dot` layout for a clean directed-graph tree
- **10-level depth colour system** — each depth level gets a distinct pastel colour that cycles; folders carry solid fills, files carry white fills so they are visually distinct
- **Depth-coloured edges** — arrows between nodes are tinted to match their parent's depth colour
- **4 output formats** — SVG, PNG, PDF, and DOT (raw Graphviz source)
- **Auto filename generation** — output named `FolderName_YYYYMMDD_HHMM.svg` by default
- **Two interfaces** — a terminal prototype (`foldersco.py`) and a full GUI (`Foldersco2.0.py` / `FolderSco2.0.exe`)
- **Live progress monitor** (GUI) — real-time files scanned, folders scanned, elapsed time, and stage indicator while generating
- **Threaded generation** (GUI) — the window stays fully responsive during large scans
- **Permission-safe** — locked system folders are skipped with a warning; they do not abort the scan
- **No hardcoded paths** — output directory is detected dynamically from `USERPROFILE`

---

## 🖥️ Interfaces

### Terminal — `foldersco.py`
A script-based prototype. Type or paste a folder path, get an SVG. After each scan it offers `[1] Scan another folder` or `[2] Exit`.

### GUI — `Foldersco2.0.py` / `FolderSco2.0.exe`
A full tkinter dashboard. Browse or drag-and-drop a folder, choose your output format and location, hit **GENERATE VISUALIZATION**, and watch the live progress monitor.

---

## 🚀 Quick Start

### Using the EXE (recommended)
1. Install Graphviz from [graphviz.org/download](https://graphviz.org/download/) — tick *"Add to PATH"* during setup
2. Double-click `FolderSco2.0.exe`
3. Browse to a folder → choose format → click **GENERATE VISUALIZATION**
4. Click **Open File** or **Open Folder** when done

### Using the source
```bash
pip install graphviz
python Foldersco2.0.py
```

---

## 📂 Basic Workflow

```
Select folder  →  Choose format  →  Set output location  →  Generate  →  Open result
```

During generation the GUI shows:

```
Scanning files...
Files scanned:    8,391
Folders scanned:  327
Elapsed time:     00:02:14
[████████████░░░░░░] Processing...
```

After completion:

```
✓  Visualization generated successfully
Files: 8,391  |  Folders: 327  |  Time: 00:02:31
Output: MyProject_20260926_0012.svg
[Open File]  [Open Folder]  [↺ New Scan]
```

---

## 📄 Output Formats

| Format | Description | Requires dot.exe |
|--------|-------------|-----------------|
| **SVG** | Vector — infinitely scalable, opens in any browser | ✅ Yes |
| **PNG** | Raster image | ✅ Yes |
| **PDF** | Document format | ✅ Yes |
| **DOT** | Raw Graphviz source (`.dot` text file) | ❌ No |

> **SVG is the recommended format.** It is vector-based, so it stays crisp at any zoom level regardless of how large the tree is.

---

## 💡 Example Use Cases

- **Document a project** before sharing it on GitHub or handing it off
- **Audit a folder** to understand what's inside before archiving or deleting
- **Visualize a drive** to see top-level organisation at a glance
- **Generate a report** of a client deliverable or asset folder
- **Explore an unfamiliar codebase** by scanning its root directory

---

## 📁 Output Location

By default, all generated files are saved to:

```
C:\Users\<YourName>\Desktop\FileTreeGen\
```

This folder is created automatically if it does not exist. The location can be changed in the GUI.

---

## 🎨 Colour System

Each depth level in the tree gets a distinct pastel background colour. The palette has 10 levels and then repeats:

| Depth | Colour |
|-------|--------|
| 0 | Light Blue (`#DCEBFF`) |
| 1 | Mint (`#DDF5E3`) |
| 2 | Soft Yellow (`#FFF1CC`) |
| 3 | Soft Pink (`#FCE1E4`) |
| 4 | Lavender (`#E8DEFF`) |
| 5 | Aqua (`#DDF4F4`) |
| 6 | Peach (`#FFE4CC`) |
| 7 | Periwinkle (`#E5E5F5`) |
| 8 | Light Green (`#E6F4D7`) |
| 9 | Soft Purple (`#F5E1F5`) |

Folder nodes carry the full pastel fill. File nodes use white (`#FFFFFF`) with the depth's border and font colour — keeping leaves visually distinct while still showing their hierarchy level.

---

## 📸 Screenshots

See the [`Screenshots/`](Screenshots/) folder.

---

## 📋 Requirements

- Windows 10 / 11
- Python 3.8+ *(source only — not needed for the EXE)*
- `graphviz` Python package — `pip install graphviz`
- **Graphviz application** (dot.exe) — [graphviz.org/download](https://graphviz.org/download/)
- `tkinterdnd2` *(optional, enables drag-and-drop)* — `pip install tkinterdnd2`

See [SETUP.md](SETUP.md) for full instructions.

---

## 📜 License

MIT — see [LICENSE](LICENSE).
