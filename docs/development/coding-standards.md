# Coding Standards

Guidelines for writing clean, consistent, and maintainable code in PixelStudio.

## GDScript Style Guide

We follow the [official Godot GDScript style guide](https://docs.godotengine.org/en/stable/tutorials/scripting/gdscript/gdscript_styleguide.html) with some project-specific additions.

## Code Formatting

### Indentation
- Use **tabs** for indentation (Godot default)
- One tab = 4 spaces equivalent
- Align continuation lines with opening delimiter

### Line Length
- Maximum **100 characters** per line
- Break long lines at logical points
- Use line continuation for long expressions

### Blank Lines
- Two blank lines between top-level classes and functions
- One blank line between methods
- One blank line between logical sections in functions

## Naming Conventions

### Variables
```gdscript
# Local variables: snake_case
var player_health: int = 100
var canvas_size: Vector2i

# Constants: SCREAMING_SNAKE_CASE
const MAX_LAYERS: int = 64
const DEFAULT_BRUSH_SIZE: int = 1

# Private variables: prefix with underscore
var _internal_state: bool = false
var _cached_data: Array = []
```

### Functions
```gdscript
# Functions: snake_case
func draw_pixel(position: Vector2i, color: Color) -> void:
    pass

func get_layer_count() -> int:
    return layers.size()

# Private functions: prefix with underscore
func _update_internal_state() -> void:
    pass

# Signal handlers: prefix with _on_
func _on_button_pressed() -> void:
    pass
```

### Classes
```gdscript
# Class names: PascalCase
class_name DrawingTool
extends RefCounted

class_name LayerManager
extends Node

# Enums: PascalCase for enum, SCREAMING_SNAKE_CASE for values
enum BlendMode {
    NORMAL,
    MULTIPLY,
    SCREEN,
    OVERLAY,
}

enum ToolType {
    PENCIL,
    ERASER,
    FILL_BUCKET,
}
```

### Files
- Script files: `snake_case.gd`
- Scene files: `snake_case.tscn`
- Resource files: `snake_case.tres`

## Type Hints

**Always use type hints** for better code clarity and performance.

```gdscript
# Good: Type hints everywhere
func calculate_distance(point_a: Vector2, point_b: Vector2) -> float:
    var distance: float = point_a.distance_to(point_b)
    return distance

# Bad: No type hints
func calculate_distance(point_a, point_b):
    var distance = point_a.distance_to(point_b)
    return distance
```

### Type Hint Guidelines
- All function parameters
- All function return types
- All variable declarations (when not obvious)
- Use `-> void` for functions that don't return

## Code Organization

### Script Structure

Order elements in scripts as follows:

```gdscript
class_name MyClass
extends BaseClass

# 1. Signals
signal data_changed(new_value: int)
signal action_completed()

# 2. Enums
enum State {
    IDLE,
    ACTIVE,
    DISABLED,
}

# 3. Constants
const MAX_VALUE: int = 100
const DEFAULT_COLOR: Color = Color.WHITE

# 4. Exported variables
@export var visible_property: bool = true
@export var speed: float = 100.0

# 5. Public variables
var current_state: State = State.IDLE
var position: Vector2 = Vector2.ZERO

# 6. Private variables
var _internal_cache: Dictionary = {}
var _is_initialized: bool = false

# 7. Onready variables
@onready var sprite: Sprite2D = $Sprite2D
@onready var timer: Timer = $Timer

# 8. Lifecycle methods (in order they're called)
func _init() -> void:
    pass

func _ready() -> void:
    pass

func _process(delta: float) -> void:
    pass

func _physics_process(delta: float) -> void:
    pass

# 9. Public methods
func public_method() -> void:
    pass

# 10. Private methods
func _private_method() -> void:
    pass

# 11. Signal handlers
func _on_button_pressed() -> void:
    pass
```

## Comments

### When to Comment

**Do comment:**
- Complex algorithms and logic
- Non-obvious optimizations
- Workarounds for known issues
- Public API documentation
- TODOs and FIXMEs

**Don't comment:**
- Obvious code
- What code does (code should be self-explanatory)
- Redundant information

### Comment Style

```gdscript
# Single-line comments for brief explanations
var speed: float = 100.0  # Pixels per second

## Documentation comments for public APIs
## Use double-hash for documentation that should appear in editor
## 
## @param position: The world position to check
## @return: True if position is valid
func is_valid_position(position: Vector2i) -> bool:
    return position.x >= 0 and position.y >= 0

# Multi-line explanations for complex logic
# This algorithm uses a flood-fill approach with a queue
# to efficiently fill connected regions with the same color.
# Time complexity: O(n) where n is the number of pixels filled.
func flood_fill(start_pos: Vector2i, fill_color: Color) -> void:
    pass

# TODOs should include context and ideally an issue number
# TODO(username): Optimize this for large images (#123)
# FIXME: Handle edge case when position is negative
```

## Best Practices

### Error Handling

```gdscript
# Use assert for development-time checks
func set_layer_opacity(index: int, opacity: float) -> void:
    assert(index >= 0 and index < layers.size(), "Layer index out of bounds")
    assert(opacity >= 0.0 and opacity <= 1.0, "Opacity must be 0-1")
    layers[index].opacity = opacity

# Use push_error for runtime errors
func load_file(path: String) -> bool:
    if not FileAccess.file_exists(path):
        push_error("File not found: " + path)
        return false
    # Load file...
    return true

# Use push_warning for non-critical issues
func update_canvas() -> void:
    if not is_initialized:
        push_warning("Canvas not initialized, skipping update")
        return
    # Update canvas...
```

### Resource Management

```gdscript
# Free resources when done
var image: Image = Image.create(128, 128, false, Image.FORMAT_RGBA8)
# Use image...
image = null  # Let GC handle it, or use queue_free() for nodes

# Use signals for cleanup
func _ready() -> void:
    tree_exiting.connect(_on_tree_exiting)

func _on_tree_exiting() -> void:
    # Cleanup resources
    _cleanup_resources()
```

### Performance

```gdscript
# Cache frequently accessed values
var _cached_canvas_rect: Rect2i
var _cache_dirty: bool = true

func get_canvas_rect() -> Rect2i:
    if _cache_dirty:
        _cached_canvas_rect = _calculate_canvas_rect()
        _cache_dirty = false
    return _cached_canvas_rect

# Use typed arrays when possible
var positions: Array[Vector2i] = []

# Avoid creating objects in loops
# Bad:
for i in range(1000):
    var temp: Vector2 = Vector2(i, i)  # Creates 1000 objects

# Good:
var temp: Vector2 = Vector2.ZERO
for i in range(1000):
    temp.x = i
    temp.y = i
```

### Signals

```gdscript
# Declare signals at the top of the class
signal layer_added(layer: Layer)
signal layer_removed(index: int)
signal property_changed(property_name: String, value: Variant)

# Connect signals in _ready or _init
func _ready() -> void:
    layer_added.connect(_on_layer_added)
    
# Name signal handlers with _on_ prefix
func _on_layer_added(layer: Layer) -> void:
    _update_layer_list()

# Always disconnect signals when appropriate
func _exit_tree() -> void:
    if some_object:
        some_object.signal_name.disconnect(_on_signal)
```

## Code Review Checklist

Before submitting code, verify:

- [ ] All code follows naming conventions
- [ ] Type hints are used everywhere
- [ ] Comments explain "why", not "what"
- [ ] No magic numbers (use constants)
- [ ] Error cases are handled
- [ ] Resources are properly managed
- [ ] No unnecessary dependencies
- [ ] Code is DRY (Don't Repeat Yourself)
- [ ] Functions are small and focused
- [ ] Variables have meaningful names
- [ ] Tests are included for new features

## Common Patterns

### Singleton Pattern (Autoloads)

```gdscript
# project_manager.gd (Autoload)
extends Node

var current_project: Project = null
var project_path: String = ""

func load_project(path: String) -> bool:
    # Load project logic
    return true

func save_project() -> bool:
    # Save project logic
    return true
```

### Command Pattern (Undo/Redo)

```gdscript
class_name DrawCommand
extends Command

var _canvas: Canvas
var _position: Vector2i
var _color: Color
var _old_color: Color

func _init(canvas: Canvas, position: Vector2i, color: Color) -> void:
    _canvas = canvas
    _position = position
    _color = color
    _old_color = canvas.get_pixel(position)

func execute() -> void:
    _canvas.set_pixel(_position, _color)

func undo() -> void:
    _canvas.set_pixel(_position, _old_color)
```

### Observer Pattern (Signals)

```gdscript
class_name Layer
extends RefCounted

signal opacity_changed(new_opacity: float)
signal visibility_changed(visible: bool)

var opacity: float = 1.0:
    set(value):
        opacity = clamp(value, 0.0, 1.0)
        opacity_changed.emit(opacity)
```

## Testing Standards

### Unit Test Structure

```gdscript
# test_layer.gd
extends GutTest

func test_layer_creation() -> void:
    var layer: Layer = Layer.new()
    assert_not_null(layer, "Layer should be created")
    assert_eq(layer.opacity, 1.0, "Default opacity should be 1.0")

func test_layer_opacity_clamping() -> void:
    var layer: Layer = Layer.new()
    layer.opacity = 1.5
    assert_eq(layer.opacity, 1.0, "Opacity should be clamped to 1.0")
    
    layer.opacity = -0.5
    assert_eq(layer.opacity, 0.0, "Opacity should be clamped to 0.0")
```

### Test Naming
- Test files: `test_*.gd`
- Test functions: `test_*`
- Setup: `before_each()` or `before_all()`
- Teardown: `after_each()` or `after_all()`

## Git Commit Messages

### Format
```
<type>(<scope>): <subject>

<body>

<footer>
```

### Types
- `feat`: New feature
- `fix`: Bug fix
- `docs`: Documentation changes
- `style`: Code style changes (formatting)
- `refactor`: Code refactoring
- `test`: Adding or updating tests
- `chore`: Maintenance tasks

### Examples
```
feat(tools): Add circle drawing tool

Implement circle/ellipse drawing tool with shift-constraint
for perfect circles. Supports filled and outline modes.

Closes #42

---

fix(layers): Fix opacity not applying to child layers

Layer groups now correctly apply opacity to all child layers
in the rendering pipeline.

Fixes #67

---

docs(api): Add documentation for Canvas class

Added comprehensive documentation for Canvas class methods
including usage examples and parameter descriptions.
```

## Documentation Standards

### Inline Documentation

```gdscript
## Base class for all drawing tools.
##
## Drawing tools implement the tool interface to handle user input
## and modify the canvas. Tools can be selected from the toolbar
## and have customizable settings.
##
## @tutorial: https://docs.pixelstudio.com/tools
class_name Tool
extends RefCounted

## Emitted when tool settings change.
signal settings_changed()

## The display name of the tool.
var name: String = "Tool"

## The keyboard shortcut for this tool.
var shortcut: String = ""

## Called when the tool is activated.
## @param canvas: The target canvas for the tool.
func activate(canvas: Canvas) -> void:
    pass

## Called when mouse button is pressed.
## @param position: The pixel position in canvas coordinates.
## @param button: The mouse button (MOUSE_BUTTON_LEFT, etc.)
func on_mouse_down(position: Vector2i, button: int) -> void:
    pass
```

## Tools and Utilities

### Recommended Editor Settings

**Godot Editor Settings:**
- Text Editor > Behavior > Files > Trim Trailing Whitespace On Save: **On**
- Text Editor > Behavior > Files > Auto Indent: **On**
- Text Editor > Indent > Type: **Tabs**
- Text Editor > Indent > Size: **4**

### Linting

Consider using [GDScript Toolkit](https://github.com/Scony/godot-gdscript-toolkit) for linting:

```bash
# Install
pip install gdtoolkit

# Format code
gdformat src/

# Lint code
gdlint src/
```

## Further Reading

- [Godot Best Practices](https://docs.godotengine.org/en/stable/tutorials/best_practices/index.html)
- [GDScript Style Guide](https://docs.godotengine.org/en/stable/tutorials/scripting/gdscript/gdscript_styleguide.html)
- [Clean Code](https://github.com/ryanmcdermott/clean-code-javascript) (Principles apply to GDScript)

## Next Steps

- Review [Architecture](architecture.md) for design patterns
- Check [Project Structure](project-structure.md) for file organization
- Read [Contributing](contributing.md) to start developing
