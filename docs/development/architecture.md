# Architecture

!!! info "Status"
    Placeholder page. This will be expanded as implementation progresses.

## Overview
High-level architecture of PixelStudio, including core systems and how they interact.

## Core Components
- Drawing engine (Image/ImageTexture, brush pipeline)
- Tool system (pencil, eraser, fill; tool state, cursors)
- Layer system (stack, blend, compositing)
- Command/History (undo/redo)
- Persistence (project format, export pipeline)
- UI/UX (Godot Control layout, theming)

## Data Flow
Input → Tool → Operation → Image → Texture → Canvas

---