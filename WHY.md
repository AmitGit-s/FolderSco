# Why FolderSco?

> **FolderSco turns a filesystem structure into a visual map that is easier to understand, communicate, and share.**

## The Problem

Navigating a complex project structure using a standard file explorer or a terminal tree command can be tedious. You often have to open folders one by one to see what's inside, or scroll through hundreds of lines of text in a console. This makes it difficult to quickly grasp the architecture of an unfamiliar project, communicate that structure to others, or provide context to an AI assistant.

## The Solution

FolderSco solves this by converting any local directory into a directed graph. Using Graphviz's hierarchical layout and a distinct 10-level color-coded depth system, it instantly visualizes parent-to-child relationships. The output is a clear, self-contained SVG, PNG, or PDF that you can easily read, zoom into, and share. 

## Practical Use Cases

* **Quickly understanding an unfamiliar project structure:** Get an immediate bird's-eye view of a new codebase or asset directory without clicking through endless nested folders.
* **Visualizing large or complex folder hierarchies:** See the exact depth, scale, and layout of your directories at a glance.
* **Sharing project structure with teammates:** Provide a clear, visual map of where files belong when onboarding new developers, handing off deliverables, or proposing architectural changes.
* **Sharing folder architecture with AI agents:** Provide AI coding assistants with a complete structural map of your project without manually explaining every path or formatting text trees.
* **Creating a visual reference/documentation of a project:** Embed the generated SVGs or PNGs directly into your `README.md`, wikis, or internal documentation.
* **Understanding parent → child relationships:** The left-to-right directed graph and depth-colored edges make the hierarchy between files and folders instantly obvious.
* **Quickly inspecting a project:** Audit a directory's contents completely offline, without needing to open folders one by one in Windows Explorer.
* **Creating shareable representations of a local filesystem:** Generate portable SVG, PNG, PDF, or DOT files that can be viewed by anyone on any device.
* **Communicating project structure more clearly:** Help developers and stakeholders see the exact layout of a project, eliminating ambiguity and saving time.
