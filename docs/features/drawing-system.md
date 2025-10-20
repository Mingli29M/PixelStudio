# Drawing System

An in-depth look at PixelStudio's drawing system architecture and capabilities.

## Overview

The drawing system is the core of PixelStudio, providing pixel-perfect rendering and manipulation with a variety of tools and algorithms optimized for pixel art creation.

## Architecture

### Core Components

```
Drawing System
├── Canvas          # Main drawing surface
├── Drawing Engine  # Core pixel operations
├── Tool System     # Drawing tools
├── Brush System    # Brush configuration
└── Pixel Buffer    # Pixel data storage
```

## Canvas

The canvas is the main drawing surface where all artwork is created.

### Canvas Properties

- **Size**: Resolution in pixels (width × height)
- **Format**: RGBA8 (8-bit per channel with alpha)
- **Background**: Transparent or solid color
- **Grid**: Optional pixel grid overlay
- **Guides**: Ruler guides for alignment

### Canvas Rendering

The canvas uses Godot's `Image` class for pixel data storage and `Sprite2D` for display:

```gdscript
class_name Canvas
extends Node2D

var image: Image
var texture: ImageTexture
var sprite: Sprite2D

func _ready() -> void:
    image = Image.create(width, height, false, Image.FORMAT_RGBA8)
    texture = ImageTexture.create_from_image(image)
    sprite.texture = texture

func update_texture() -> void:
    texture.update(image)
```

### Viewport System

Multiple viewports for different zoom levels and preview modes:
- **Main Viewport**: Primary editing view
- **Navigator**: Thumbnail view for navigation
- **Preview**: Full-resolution preview

## Drawing Engine

The core drawing engine handles all pixel manipulation operations.

### Pixel Operations

#### Set Pixel
```gdscript
func set_pixel(position: Vector2i, color: Color) -> void:
    if not is_valid_position(position):
        return
    image.set_pixelv(position, color)
    mark_region_dirty(Rect2i(position, Vector2i(1, 1)))
```

#### Get Pixel
```gdscript
func get_pixel(position: Vector2i) -> Color:
    if not is_valid_position(position):
        return Color.TRANSPARENT
    return image.get_pixelv(position)
```

#### Blend Pixel
```gdscript
func blend_pixel(position: Vector2i, color: Color, opacity: float) -> void:
    var existing: Color = get_pixel(position)
    var blended: Color = existing.lerp(color, opacity)
    set_pixel(position, blended)
```

### Drawing Algorithms

#### Line Drawing (Bresenham's Algorithm)
```gdscript
func draw_line(start: Vector2i, end: Vector2i, color: Color) -> void:
    var dx: int = abs(end.x - start.x)
    var dy: int = abs(end.y - start.y)
    var sx: int = 1 if start.x < end.x else -1
    var sy: int = 1 if start.y < end.y else -1
    var err: int = dx - dy
    var pos: Vector2i = start
    
    while true:
        set_pixel(pos, color)
        if pos == end:
            break
        var e2: int = 2 * err
        if e2 > -dy:
            err -= dy
            pos.x += sx
        if e2 < dx:
            err += dx
            pos.y += sy
```

#### Circle Drawing (Midpoint Circle Algorithm)
```gdscript
func draw_circle(center: Vector2i, radius: int, color: Color, filled: bool) -> void:
    if filled:
        _draw_filled_circle(center, radius, color)
    else:
        _draw_circle_outline(center, radius, color)

func _draw_circle_outline(center: Vector2i, radius: int, color: Color) -> void:
    var x: int = 0
    var y: int = radius
    var d: int = 1 - radius
    
    while x <= y:
        _plot_circle_points(center, x, y, color)
        x += 1
        if d < 0:
            d += 2 * x + 1
        else:
            y -= 1
            d += 2 * (x - y) + 1

func _plot_circle_points(center: Vector2i, x: int, y: int, color: Color) -> void:
    # Plot 8 symmetric points
    set_pixel(Vector2i(center.x + x, center.y + y), color)
    set_pixel(Vector2i(center.x - x, center.y + y), color)
    set_pixel(Vector2i(center.x + x, center.y - y), color)
    set_pixel(Vector2i(center.x - x, center.y - y), color)
    set_pixel(Vector2i(center.x + y, center.y + x), color)
    set_pixel(Vector2i(center.x - y, center.y + x), color)
    set_pixel(Vector2i(center.x + y, center.y - x), color)
    set_pixel(Vector2i(center.x - y, center.y - x), color)
```

#### Flood Fill
```gdscript
func flood_fill(start: Vector2i, fill_color: Color) -> void:
    var target_color: Color = get_pixel(start)
    if target_color == fill_color:
        return
    
    var queue: Array[Vector2i] = [start]
    var visited: Dictionary = {}
    
    while not queue.is_empty():
        var pos: Vector2i = queue.pop_front()
        
        if visited.has(pos) or not is_valid_position(pos):
            continue
        
        if get_pixel(pos) != target_color:
            continue
        
        set_pixel(pos, fill_color)
        visited[pos] = true
        
        # Add neighbors
        queue.append(Vector2i(pos.x + 1, pos.y))
        queue.append(Vector2i(pos.x - 1, pos.y))
        queue.append(Vector2i(pos.x, pos.y + 1))
        queue.append(Vector2i(pos.x, pos.y - 1))
```

## Tool System

### Tool Interface

All tools implement a common interface:

```gdscript
class_name Tool
extends RefCounted

signal settings_changed()

var name: String
var icon: Texture2D
var shortcut: String
var cursor: int = CURSOR_ARROW

func activate(canvas: Canvas) -> void:
    pass

func deactivate() -> void:
    pass

func on_mouse_down(position: Vector2i, button: int) -> void:
    pass

func on_mouse_move(position: Vector2i) -> void:
    pass

func on_mouse_up(position: Vector2i, button: int) -> void:
    pass

func on_key_pressed(event: InputEventKey) -> void:
    pass

func draw_overlay(canvas_node: CanvasItem) -> void:
    pass
```

### Drawing Tools

#### Pencil Tool
- Single pixel drawing
- Pressure sensitivity support
- Configurable size (1-64 pixels)

#### Brush Tool
- Soft or hard edges
- Custom brush shapes
- Opacity and flow control
- Texture brushes

#### Eraser Tool
- Removes pixels (sets to transparent)
- Same brush options as brush tool
- Can erase to background color

#### Fill Bucket Tool
- Flood fill algorithm
- Tolerance setting for color matching
- Fill selection mode
- Contiguous or all matching pixels

### Shape Tools

#### Line Tool
- Bresenham's line algorithm
- Hold Shift for 45° angle constraint
- Width control

#### Rectangle Tool
- Filled or outline mode
- Square constraint with Shift
- Rounded corners option

#### Circle Tool
- Midpoint circle algorithm
- Filled or outline mode
- Perfect circle with Shift

### Selection Tools

#### Rectangular Selection
- Click and drag to create rectangle
- Hold Shift to add to selection
- Hold Alt to subtract from selection

#### Lasso Selection
- Free-form selection
- Polygon mode for straight edges

#### Magic Wand
- Color-based selection
- Tolerance setting
- Contiguous or global

## Brush System

### Brush Properties

```gdscript
class_name Brush
extends Resource

@export var size: int = 1          # 1-64 pixels
@export var opacity: float = 1.0   # 0.0-1.0
@export var hardness: float = 1.0  # 0.0-1.0 (soft to hard)
@export var shape: BrushShape = BrushShape.CIRCLE
@export var texture: Texture2D = null
@export var spacing: float = 0.1   # Distance between dabs

enum BrushShape {
    CIRCLE,
    SQUARE,
    DIAMOND,
    CUSTOM,
}
```

### Brush Rendering

```gdscript
func draw_brush_dab(position: Vector2i, brush: Brush, color: Color) -> void:
    var half_size: int = brush.size / 2
    
    for y in range(-half_size, half_size + 1):
        for x in range(-half_size, half_size + 1):
            var pixel_pos: Vector2i = position + Vector2i(x, y)
            
            # Check if pixel is within brush shape
            if _is_in_brush_shape(Vector2i(x, y), brush):
                var alpha: float = _calculate_brush_alpha(
                    Vector2i(x, y), brush
                )
                var pixel_color: Color = color
                pixel_color.a *= alpha * brush.opacity
                blend_pixel(pixel_pos, pixel_color, pixel_color.a)

func _is_in_brush_shape(offset: Vector2i, brush: Brush) -> bool:
    match brush.shape:
        Brush.BrushShape.CIRCLE:
            return offset.length() <= brush.size / 2.0
        Brush.BrushShape.SQUARE:
            return abs(offset.x) <= brush.size / 2 and abs(offset.y) <= brush.size / 2
        Brush.BrushShape.DIAMOND:
            return abs(offset.x) + abs(offset.y) <= brush.size / 2
    return false

func _calculate_brush_alpha(offset: Vector2i, brush: Brush) -> float:
    var distance: float = offset.length()
    var radius: float = brush.size / 2.0
    
    if distance >= radius:
        return 0.0
    
    # Apply hardness
    var hardness_falloff: float = 1.0 - brush.hardness
    var normalized_distance: float = distance / radius
    
    if normalized_distance < 1.0 - hardness_falloff:
        return 1.0
    else:
        var falloff_range: float = 1.0 - (1.0 - hardness_falloff)
        var alpha: float = 1.0 - ((normalized_distance - (1.0 - hardness_falloff)) / hardness_falloff)
        return max(0.0, alpha)
```

## Performance Optimization

### Dirty Region Tracking

Only redraw changed areas:

```gdscript
var _dirty_regions: Array[Rect2i] = []

func mark_region_dirty(region: Rect2i) -> void:
    _dirty_regions.append(region)

func update_texture() -> void:
    if _dirty_regions.is_empty():
        return
    
    # Merge overlapping regions
    var merged: Rect2i = _dirty_regions[0]
    for region in _dirty_regions:
        merged = merged.merge(region)
    
    # Update only dirty region
    texture.update(image)
    _dirty_regions.clear()
```

### Chunk-based Processing

Process large operations in chunks to avoid freezing:

```gdscript
func process_large_operation(pixels: Array[Vector2i], color: Color) -> void:
    const CHUNK_SIZE: int = 1000
    var chunks: int = ceili(pixels.size() / float(CHUNK_SIZE))
    
    for i in range(chunks):
        var start: int = i * CHUNK_SIZE
        var end: int = min(start + CHUNK_SIZE, pixels.size())
        
        for j in range(start, end):
            set_pixel(pixels[j], color)
        
        # Yield to prevent freezing
        await get_tree().process_frame
```

## Future Enhancements

- [ ] GPU-accelerated rendering
- [ ] Custom brush engine
- [ ] Pressure curve customization
- [ ] Symmetry drawing modes
- [ ] Perspective grid support
- [ ] Texture wrapping preview
- [ ] Animation onion skin in drawing
- [ ] Brush stabilization

## See Also

- [Drawing Tools User Guide](../user-guide/drawing-tools.md)
- [Layer Management](layer-management.md)
- [Architecture Overview](../development/architecture.md)
