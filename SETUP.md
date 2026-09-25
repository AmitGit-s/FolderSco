# Setup Guide

Complete instructions for installing and running FolderSco on Windows.

---

## System Requirements

| Requirement | Minimum |
|-------------|---------|
| **Operating System** | Windows 10 or Windows 11 (64-bit) |
| **Python** | 3.8 or later *(source files only — not needed for the EXE)* |
| **Graphviz application** | Any recent stable release |
| **Disk space** | ~50 MB (includes Graphviz) |
| **RAM** | No hard limit — large folder trees use more memory during scan |

---

## Step 1 — Install the Graphviz Application

FolderSco calls `dot.exe` (part of the Graphviz application) to render graphs.  
**This is separate from the Python `graphviz` package** — both are required.

1. Go to **[https://graphviz.org/download/](https://graphviz.org/download/)**
2. Download the Windows `.exe` installer (choose the 64-bit stable release)
3. Run the installer
4. On the *Select Additional Tasks* screen, tick:
   > ✅ **Add Graphviz to the system PATH for all users**
   > *(or "for current user" if you prefer)*
5. Complete the installation and **close any open terminals**

### Verify Graphviz is working

Open a new Command Prompt or PowerShell and run:

```
dot -V
```

Expected output (version number will vary):

```
dot - graphviz version 12.x.x
```

If you see `'dot' is not recognized`, either Graphviz was not added to PATH or you need to restart the terminal.

> **DOT format exception** — Generating raw `.dot` source files does *not* require `dot.exe`. Only SVG, PNG, and PDF rendering uses it.

---

## Step 2 — Install Python (source only)

Skip this step if you are using `FolderSco2.0.exe`.

Download Python 3.8 or later from **[python.org/downloads](https://python.org/downloads/)**.  
During installation, tick **"Add Python to PATH"**.

Verify:

```
python --version
```

---

## Step 3 — Install the Python `graphviz` Package (source only)

```bash
pip install graphviz
```

This installs the Python wrapper that lets the scripts communicate with `dot.exe`.

Verify:

```
pip show graphviz
```

---

## Step 4 (Optional) — Install Drag-and-Drop Support

The GUI (`Foldersco2.0.py`) can accept folders dragged directly onto the path field if `tkinterdnd2` is installed:

```bash
pip install tkinterdnd2
```

If `tkinterdnd2` is not installed, the GUI still works fully — Browse and paste-path input both function without it.

---

## Running the Source Files

### Terminal — `foldersco.py`

```bash
python foldersco.py
```

The program prints a banner, then prompts:

```
Enter folder path  (or type 'exit' to quit):
>
```

Type or paste a folder path and press Enter. After each scan:

```
--------------------------------------------
  [1]  Scan another folder
  [2]  Exit
--------------------------------------------
>
```

Press `1` to scan again or `2` to exit. You can also type `exit`, `quit`, or `q` at the path prompt to quit immediately.

**Output** is saved automatically to:

```
C:\Users\<YourName>\Desktop\FileTreeGen\<FolderName>_<YYYYMMDD_HHMM>.svg
```

### GUI — `Foldersco2.0.py`

```bash
python Foldersco2.0.py
```

The GUI window opens. Follow the on-screen workflow:

1. Paste or browse to a folder path
2. Choose an output format (SVG / PNG / PDF / DOT)
3. Optionally change the output directory or set a custom filename
4. Click **⚡ GENERATE VISUALIZATION**
5. Watch the live progress monitor
6. Click **Open File** or **Open Folder** when complete
7. Click **↺ New Scan** to start again

---

## Using the EXE — `FolderSco2.0.exe`

No Python installation is required to run the EXE.

1. Ensure Graphviz is installed and `dot.exe` is on PATH (Step 1 above)
2. Double-click `dist\FolderSco2.0.exe`
3. The FolderSco 2.0 GUI opens — follow the same steps as above

> The EXE has Python 3.14 bundled inside it. It is a self-contained Windows application (~11 MB).

---

## Default Output Location

All generated files are saved to:

```
C:\Users\<YourName>\Desktop\FileTreeGen\
```

This folder is created automatically on first run. The GUI lets you change this to any directory.

---

## Troubleshooting

| Problem | Solution |
|---------|----------|
| `'dot' is not recognized` | Graphviz not on PATH — reinstall with PATH option ticked, then restart terminal |
| `ModuleNotFoundError: graphviz` | Run `pip install graphviz` |
| `ExecutableNotFound` error at runtime | `dot.exe` was removed or PATH changed — reinstall Graphviz |
| GUI shows "Not Responding" | Should not happen — generation runs on a background thread. If it does, the folder is very large; allow time to complete |
| Output file not found | Check `Desktop\FileTreeGen\` — the folder and file are created automatically |
| Blank filename for drive roots (`C:\`) | The EXE/GUI handles this automatically and uses `C_Drive` as the name prefix |
