# Project Structure

Understanding the organization of the PixelStudio project files and directories.

## Directory Structure

```
PixelStudio/
├── .github/              # GitHub-specific files
│   ├── workflows/        # CI/CD workflows
│   └── ISSUE_TEMPLATE/   # Issue templates
├── addons/               # Godot addons and plugins
├── assets/               # Game assets (sprites, fonts, etc.)
│   ├── icons/            # Application and tool icons
│   ├── fonts/            # UI fonts
│   └── themes/           # UI themes
├── docs/                 # Documentation (MkDocs)
│   ├── getting-started/  # Installation and setup guides
│   ├── user-guide/       # User documentation
│   ├── development/      # Developer documentation
│   ├── features/         # Feature-specific documentation
│   └── about/            # Project information
├── src/                  # Source code
│   ├── core/             # Core systems
│   │   ├── drawing/      # Drawing engine
│   │   ├── layers/       # Layer management
│   │   ├── animation/    # Animation system
│   │   └── palette/      # Color palette system
│   ├── tools/            # Drawing tools
│   │   ├── pencil.gd
│   │   ├── eraser.gd
│   │   ├── fill_bucket.gd
│   │   └── ...
│   ├── ui/               # User interface
│   │   ├── panels/       # UI panels (layers, tools, etc.)
│   │   ├── dialogs/      # Dialog windows
│   │   └── canvas/       # Canvas rendering
│   ├── utils/            # Utility functions and helpers
│   └── managers/         # Global managers (singletons)
├── scenes/               # Godot scene files (.tscn)
│   ├── main.tscn         # Main application scene
│   ├── canvas.tscn       # Canvas scene
│   └── ui/               # UI scenes
├── scripts/              # Build and utility scripts
├── tests/                # Test files
│   ├── unit/             # Unit tests
│   └── integration/      # Integration tests
├── export/               # Export presets and templates
├── .gitignore            # Git ignore file
├── CONTRIBUTING.md       # Contributing guidelines
├── LICENSE               # License file (MIT)
├── README.md             # Project readme
├── project.godot         # Godot project file
└── mkdocs.yml            # MkDocs configuration
```

## Source Code Organization

### Core Systems (`src/core/`)

Core functionality that powers the application.

#### Drawing System (`src/core/drawing/`)
```
drawing/
├── canvas.gd             # Main canvas class
├── drawing_engine.gd     # Core drawing algorithms
├── brush.gd              # Brush implementation
├── pixel_buffer.gd       # Pixel data management
└── compositing.gd        # Layer composition
```

#### Layer System (`src/core/layers/`)
```
layers/
├── layer.gd              # Layer class
├── layer_stack.gd        # Layer collection management
├── blend_modes.gd        # Blending algorithms
└── layer_group.gd        # Layer grouping
```

#### Animation System (`src/core/animation/`)
```
animation/
├── timeline.gd           # Timeline management
├── frame.gd              # Animation frame
├── animation_player.gd   # Playback controller
└── onion_skin.gd         # Onion skinning
```

#### Palette System (`src/core/palette/`)
```
palette/
├── palette.gd            # Palette class
├── palette_manager.gd    # Palette operations
├── color_picker.gd       # Color selection UI
└── palette_io.gd         # Import/export handlers
```

### Tools (`src/tools/`)

Individual drawing tool implementations.

```
tools/
├── tool_base.gd          # Base tool class
├── pencil.gd             # Pencil tool
├── eraser.gd             # Eraser tool
├── fill_bucket.gd        # Fill bucket tool
├── line.gd               # Line tool
├── rectangle.gd          # Rectangle tool
├── circle.gd             # Circle/ellipse tool
├── selection/            # Selection tools
│   ├── marquee.gd
│   ├── lasso.gd
│   └── magic_wand.gd
├── eyedropper.gd         # Color picker tool
└── move.gd               # Move tool
```

### User Interface (`src/ui/`)

UI components and panels.

```
ui/
├── panels/
│   ├── layers_panel.gd   # Layers panel
│   ├── tools_panel.gd    # Tools panel
│   ├── color_panel.gd    # Color selection panel
│   ├── palette_panel.gd  # Palette panel
│   └── timeline_panel.gd # Animation timeline
├── dialogs/
│   ├── new_project.gd    # New project dialog
│   ├── export_dialog.gd  # Export options
│   └── preferences.gd    # Preferences window
├── canvas_view.gd        # Canvas viewport
├── menu_bar.gd           # Main menu
└── toolbar.gd            # Tool toolbar
```

### Managers (`src/managers/`)

Singleton managers for global state.

```
managers/
├── project_manager.gd    # Project state
├── tool_manager.gd       # Active tool management
├── history_manager.gd    # Undo/redo
├── preferences_manager.gd # User preferences
├── theme_manager.gd      # UI theming
└── input_manager.gd      # Input handling
```

### Utilities (`src/utils/`)

Helper functions and utilities.

```
utils/
├── file_utils.gd         # File I/O helpers
├── image_utils.gd        # Image processing
├── math_utils.gd         # Math helpers
├── color_utils.gd        # Color operations
└── string_utils.gd       # String utilities
```

## Scene Organization

Godot scenes are organized in the `scenes/` directory:

```
scenes/
├── main.tscn             # Entry point
├── workspace/
│   ├── drawing_workspace.tscn
│   ├── animation_workspace.tscn
│   └── pixel_art_workspace.tscn
├── ui/
│   ├── panels/           # Panel scenes
│   ├── dialogs/          # Dialog scenes
│   └── components/       # Reusable UI components
└── canvas/
    └── canvas.tscn       # Canvas scene
```

## Asset Organization

Assets are organized by type in the `assets/` directory:

```
assets/
├── icons/
│   ├── tools/            # Tool icons (32x32)
│   ├── ui/               # UI element icons (16x16)
│   └── app/              # Application icons (various sizes)
├── fonts/
│   ├── ui_font.ttf       # Main UI font
│   └── mono_font.ttf     # Monospace font
├── themes/
│   ├── dark_theme.tres   # Dark theme
│   ├── light_theme.tres  # Light theme
│   └── high_contrast.tres # Accessibility theme
└── cursors/              # Custom cursor images
    ├── pencil_cursor.png
    ├── fill_cursor.png
    └── ...
```

## Configuration Files

### `project.godot`

Main Godot project configuration:
- Application settings
- Display settings
- Input mappings
- Autoload singletons
- Export presets

### `mkdocs.yml`

Documentation site configuration:
- Site structure
- Theme settings
- Navigation menu
- Plugins

### `.gitignore`

Excludes:
- Godot build artifacts (`.godot/`, `.import/`)
- Build output (`build/`, `dist/`)
- OS-specific files (`.DS_Store`, `Thumbs.db`)
- Documentation build (`site/`)

## Build and Export

Build-related files in `export/` and `scripts/`:

```
export/
├── templates/            # Export templates
├── icons/                # Platform-specific icons
└── metadata/             # App metadata (version, etc.)

scripts/
├── build.sh              # Build script
├── export_all.sh         # Export for all platforms
└── package.sh            # Package for distribution
```

## Testing Structure

Tests are organized by type:

```
tests/
├── unit/
│   ├── test_drawing_engine.gd
│   ├── test_layer_stack.gd
│   ├── test_color_utils.gd
│   └── ...
├── integration/
│   ├── test_tool_canvas.gd
│   ├── test_file_io.gd
│   └── ...
└── test_runner.gd        # Test execution
```

## Documentation Structure

```
docs/
├── index.md              # Documentation home
├── getting-started/
│   ├── installation.md
│   ├── quick-start.md
│   └── building.md
├── user-guide/
│   ├── interface.md
│   ├── drawing-tools.md
│   ├── layers.md
│   ├── animation.md
│   ├── palettes.md
│   ├── exporting.md
│   └── shortcuts.md
├── development/
│   ├── architecture.md
│   ├── project-structure.md  # This file
│   ├── coding-standards.md
│   └── contributing.md
├── features/
│   ├── drawing-system.md
│   ├── layer-management.md
│   ├── animation-timeline.md
│   └── file-formats.md
└── about/
    ├── roadmap.md
    ├── license.md
    └── changelog.md
```

## File Naming Conventions

### GDScript Files
- **Snake case**: `my_script.gd`
- **Class names**: PascalCase in file
- **Autoloads**: Suffix with `_manager` or `_system`

### Scene Files
- **Snake case**: `layer_panel.tscn`
- **Descriptive names**: Reflect purpose
- **Organized by type**: Group related scenes

### Asset Files
- **Lowercase with underscores**: `pencil_icon.png`
- **Size suffix** (if applicable): `icon_16x16.png`
- **Descriptive names**: Clear purpose

## Adding New Features

When adding new features, follow this organization:

1. **Core Logic**: Add to appropriate `src/core/` subdirectory
2. **UI Components**: Add scenes to `scenes/ui/`
3. **Tools**: Add tool scripts to `src/tools/`
4. **Documentation**: Add docs to appropriate `docs/` section
5. **Tests**: Add tests to `tests/` with matching structure

## Version Control

### Git Workflow
- **Main branch**: Stable releases
- **Develop branch**: Active development
- **Feature branches**: `feature/feature-name`
- **Bugfix branches**: `bugfix/bug-name`

### Commit Organization
- Atomic commits for each feature
- Clear commit messages
- Reference issues in commits

## Build Artifacts

Generated files (not in version control):
- `.godot/` - Godot engine cache
- `.import/` - Imported asset cache
- `site/` - Built documentation
- `build/` - Build output
- `dist/` - Distribution packages

## Maintenance

### Code Cleanup
- Remove unused files regularly
- Refactor as complexity grows
- Update documentation with changes

### Dependencies
- Minimize external dependencies
- Document addon requirements
- Keep dependencies updated

## Next Steps

- Review [Architecture](architecture.md) for design patterns
- Read [Coding Standards](coding-standards.md) for code style
- Check [Contributing](contributing.md) to start developing
