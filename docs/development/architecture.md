# Architecture

Learn about the technical architecture and design patterns used in PixelStudio.

## Overview

PixelStudio is built on **Godot Engine 4.x**, leveraging its powerful 2D rendering, node-based scene system, and GDScript for rapid development. The architecture follows a modular design with clear separation of concerns.

## High-Level Architecture

```
┌─────────────────────────────────────────────┐
│              User Interface                  │
│  (Canvas, Panels, Toolbars, Dialogs)        │
└──────────────────┬──────────────────────────┘
                   │
┌──────────────────▼──────────────────────────┐
│           Application Layer                  │
│  (Tools, Commands, State Management)        │
└──────────────────┬──────────────────────────┘
                   │
┌──────────────────▼──────────────────────────┐
│            Core Systems                      │
│  (Drawing, Layers, Animation, Palette)      │
└──────────────────┬──────────────────────────┘
                   │
┌──────────────────▼──────────────────────────┐
│         Data & File Management               │
│  (Projects, Import/Export, Persistence)     │
└─────────────────────────────────────────────┘
```

## Core Components

### 1. Drawing System

The drawing system handles all rendering and pixel manipulation operations.

**Components:**
- **Canvas**: Main drawing surface (Godot Viewport + Sprite2D)
- **Drawing Engine**: Manages pixel operations and brush strokes
- **Tool System**: Implements drawing tools (pencil, eraser, fill, etc.)
- **Brush System**: Configurable brushes with size, opacity, and shape

**Design Patterns:**
- **Command Pattern**: All drawing operations are commands for undo/redo
- **Strategy Pattern**: Different tools implement a common tool interface
- **Observer Pattern**: Canvas updates notify observers (layers panel, history, etc.)

### 2. Layer Management

Layers are implemented as separate Image resources with additional metadata.

**Structure:**
- **Layer**: Contains Image data, name, visibility, opacity, blend mode
- **LayerStack**: Manages layer order and composition
- **LayerRenderer**: Composites layers for display

**Features:**
- Multiple layers with independent properties
- Blend modes (Normal, Multiply, Screen, Overlay, etc.)
- Layer groups for organization
- Opacity and visibility controls

### 3. Animation Timeline

Frame-by-frame animation using a cel-based system.

**Components:**
- **Timeline**: UI component for frame management
- **Frame**: Container for layer states at specific time
- **AnimationPlayer**: Handles playback and preview
- **Onion Skin**: Shows previous/next frames as overlays

**Data Structure:**
```
Animation
├── Frame 1
│   ├── Layer 1 Image
│   ├── Layer 2 Image
│   └── ...
├── Frame 2
│   ├── Layer 1 Image
│   ├── Layer 2 Image
│   └── ...
└── ...
```

### 4. Color Palette System

Manages color palettes for efficient color selection and management.

**Components:**
- **Palette**: Array of colors with metadata
- **PaletteManager**: Handles palette loading, saving, and editing
- **ColorPicker**: Custom color selection UI
- **History**: Recently used colors

**Features:**
- Multiple palettes per project
- Import/export palette formats (GPL, ASE, PNG)
- Palette generation from images
- Color ramps and gradients

### 5. Tool System

Extensible tool architecture for drawing and editing operations.

**Base Tool Class:**
```gdscript
class_name Tool
extends RefCounted

var name: String
var icon: Texture2D
var shortcut: String

func on_mouse_down(position: Vector2i, button: int) -> void:
    pass

func on_mouse_move(position: Vector2i) -> void:
    pass

func on_mouse_up(position: Vector2i, button: int) -> void:
    pass

func get_cursor() -> int:
    return CURSOR_ARROW
```

**Tool Types:**
- **Painting Tools**: Pencil, Brush, Eraser
- **Shape Tools**: Line, Rectangle, Circle, Polygon
- **Selection Tools**: Marquee, Lasso, Magic Wand
- **Utility Tools**: Eyedropper, Fill Bucket, Move

### 6. Command System

All user actions are encapsulated as commands for undo/redo functionality.

**Command Pattern Implementation:**
```gdscript
class_name Command
extends RefCounted

func execute() -> void:
    pass

func undo() -> void:
    pass

func redo() -> void:
    execute()
```

**Command History:**
- Stack-based history manager
- Configurable history depth
- Memory management for large operations
- Command merging for continuous operations

## Data Flow

### Drawing Operation Flow

```
User Input → Tool → Command → Drawing Engine → Canvas Update → UI Refresh
                                     ↓
                              Command History
```

1. **User Input**: Mouse/keyboard events captured by input manager
2. **Tool Processing**: Active tool processes input and creates command
3. **Command Execution**: Command applied to canvas data
4. **History Recording**: Command stored for undo/redo
5. **Canvas Update**: Visual representation updated
6. **UI Refresh**: Related UI elements updated (layers, navigator, etc.)

### File Operations Flow

```
Load: File → Parser → Project Data → Scene Setup → UI Update
Save: Project Data → Serializer → File
Export: Canvas → Renderer → Format Encoder → File
```

## State Management

### Application State

Global application state managed through autoload singletons:

- **ProjectManager**: Current project, file paths
- **ToolManager**: Active tool and tool settings
- **HistoryManager**: Undo/redo stack
- **PreferencesManager**: User preferences
- **ThemeManager**: UI theme and colors

### Project State

Project-specific state stored in project file:

```json
{
  "version": "1.0",
  "canvas_size": {"width": 128, "height": 128},
  "layers": [...],
  "frames": [...],
  "palettes": [...],
  "settings": {...}
}
```

## Performance Considerations

### Optimization Strategies

1. **Dirty Region Tracking**: Only redraw changed areas
2. **Layer Caching**: Cache composite results when layers don't change
3. **Chunked Processing**: Process large operations in chunks
4. **Multithreading**: Use threads for export and file I/O
5. **Memory Pooling**: Reuse image buffers for operations

### Rendering Pipeline

1. **Layer Composition**: Combine visible layers with blend modes
2. **Canvas Rendering**: Draw composited result to viewport
3. **UI Overlay**: Add grid, guides, selection, etc.
4. **Viewport Output**: Present final result to screen

## Plugin System

Future extensibility through plugin architecture:

- **Tool Plugins**: Custom drawing tools
- **Export Plugins**: Additional file format support
- **Filter Plugins**: Image processing filters
- **Brush Plugins**: Custom brush behaviors

## Testing Architecture

### Unit Tests
- Core logic testing (drawing algorithms, color operations)
- Command execution and undo/redo
- File format parsing and serialization

### Integration Tests
- Tool interactions with canvas
- Layer operations and composition
- Animation timeline functionality

### UI Tests
- User interaction workflows
- Panel and dialog functionality
- Keyboard shortcut handling

## Error Handling

Robust error handling throughout the application:

1. **File Operations**: Graceful handling of corrupt or invalid files
2. **Memory Management**: Protection against out-of-memory conditions
3. **User Input**: Validation of all user inputs
4. **Rendering**: Fallback rendering for unsupported features

## Security Considerations

- **File Validation**: Sanitize all imported files
- **Resource Limits**: Prevent excessive resource usage
- **Safe Scripting**: (If plugins added) Sandbox plugin execution

## Accessibility

- **Keyboard Navigation**: Full keyboard access to all features
- **High Contrast**: Theme support for visibility
- **Screen Reader**: ARIA labels for UI elements
- **Customizable UI**: Scalable interface and fonts

## Future Architecture Goals

- [ ] Plugin API for extensibility
- [ ] Cloud sync and collaboration
- [ ] Real-time multiplayer editing
- [ ] WebAssembly export for browser support
- [ ] Mobile version architecture
- [ ] GPU-accelerated rendering
- [ ] Advanced filter and effects system

## Technical Stack

- **Engine**: Godot Engine 4.x
- **Language**: GDScript (Primary), C# (Optional)
- **Rendering**: Godot 2D Renderer (OpenGL/Vulkan)
- **UI**: Godot Control nodes + custom themes
- **Data**: JSON for project files, custom binary for images
- **Build**: Godot export templates

## References

- [Godot Engine Documentation](https://docs.godotengine.org/)
- [GDScript Style Guide](https://docs.godotengine.org/en/stable/tutorials/scripting/gdscript/gdscript_styleguide.html)
- [Design Patterns in Game Development](https://gameprogrammingpatterns.com/)

## Next Steps

- Explore [Project Structure](project-structure.md) for file organization
- Read [Coding Standards](coding-standards.md) for code conventions
- Check [Contributing](contributing.md) to start developing
