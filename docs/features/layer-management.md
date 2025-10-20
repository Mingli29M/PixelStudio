# Layer Management

Comprehensive guide to PixelStudio's layer system for organizing and compositing artwork.

## Overview

Layers allow you to separate different elements of your artwork, making it easier to edit, organize, and composite complex pixel art pieces.

## Layer Structure

### Layer Properties

Each layer has the following properties:

```gdscript
class_name Layer
extends RefCounted

signal property_changed(property: String, value: Variant)
signal visibility_changed(visible: bool)

var name: String = "Layer"
var visible: bool = true
var opacity: float = 1.0              # 0.0 to 1.0
var blend_mode: BlendMode = BlendMode.NORMAL
var locked: bool = false
var image: Image                      # Pixel data
var parent: Layer = null
var children: Array[Layer] = []
```

### Layer Types

#### Raster Layer
Standard pixel layer containing an image.
- Supports all drawing operations
- Can be transformed and edited freely
- Default layer type

#### Group Layer
Container for organizing multiple layers.
- No pixel data of its own
- Applies properties to all child layers
- Collapsible in layer panel

#### Reference Layer
Non-exportable layer for reference images.
- Visible only in editor
- Not included in exports
- Useful for tracing or reference

## Layer Operations

### Creating Layers

```gdscript
# Create new layer
func create_layer(name: String, width: int, height: int) -> Layer:
    var layer: Layer = Layer.new()
    layer.name = name
    layer.image = Image.create(width, height, false, Image.FORMAT_RGBA8)
    return layer

# Duplicate layer
func duplicate_layer(source: Layer) -> Layer:
    var new_layer: Layer = Layer.new()
    new_layer.name = source.name + " Copy"
    new_layer.visible = source.visible
    new_layer.opacity = source.opacity
    new_layer.blend_mode = source.blend_mode
    new_layer.image = source.image.duplicate()
    return new_layer
```

### Layer Stack

Layers are organized in a stack from bottom to top:

```gdscript
class_name LayerStack
extends RefCounted

signal layer_added(layer: Layer, index: int)
signal layer_removed(layer: Layer, index: int)
signal layer_moved(from_index: int, to_index: int)
signal active_layer_changed(layer: Layer)

var layers: Array[Layer] = []
var active_layer: Layer = null

func add_layer(layer: Layer, index: int = -1) -> void:
    if index < 0:
        layers.append(layer)
        index = layers.size() - 1
    else:
        layers.insert(index, layer)
    layer_added.emit(layer, index)

func remove_layer(index: int) -> void:
    if index < 0 or index >= layers.size():
        return
    var layer: Layer = layers[index]
    layers.remove_at(index)
    layer_removed.emit(layer, index)

func move_layer(from_index: int, to_index: int) -> void:
    if from_index < 0 or from_index >= layers.size():
        return
    if to_index < 0 or to_index >= layers.size():
        return
    
    var layer: Layer = layers[from_index]
    layers.remove_at(from_index)
    layers.insert(to_index, layer)
    layer_moved.emit(from_index, to_index)
```

## Blend Modes

Blend modes control how layers combine with layers below them.

### Supported Blend Modes

```gdscript
enum BlendMode {
    NORMAL,          # Standard alpha blending
    MULTIPLY,        # Darkens image
    SCREEN,          # Lightens image
    OVERLAY,         # Combines multiply and screen
    DARKEN,          # Keeps darker pixels
    LIGHTEN,         # Keeps lighter pixels
    COLOR_DODGE,     # Brightens based on blend color
    COLOR_BURN,      # Darkens based on blend color
    HARD_LIGHT,      # Strong contrast
    SOFT_LIGHT,      # Gentle contrast
    DIFFERENCE,      # Inverts based on brightness
    EXCLUSION,       # Similar to difference but softer
    HUE,             # Uses hue from blend layer
    SATURATION,      # Uses saturation from blend layer
    COLOR,           # Uses hue and saturation
    LUMINOSITY,      # Uses luminosity from blend layer
}
```

### Blend Mode Implementation

```gdscript
func blend_colors(base: Color, blend: Color, mode: BlendMode, opacity: float) -> Color:
    # Adjust blend color opacity
    blend.a *= opacity
    
    match mode:
        BlendMode.NORMAL:
            return base.lerp(blend, blend.a)
        
        BlendMode.MULTIPLY:
            var result: Color = Color(
                base.r * blend.r,
                base.g * blend.g,
                base.b * blend.b,
                base.a
            )
            return base.lerp(result, blend.a)
        
        BlendMode.SCREEN:
            var result: Color = Color(
                1.0 - (1.0 - base.r) * (1.0 - blend.r),
                1.0 - (1.0 - base.g) * (1.0 - blend.g),
                1.0 - (1.0 - base.b) * (1.0 - blend.b),
                base.a
            )
            return base.lerp(result, blend.a)
        
        BlendMode.OVERLAY:
            var result: Color = Color(
                _overlay_channel(base.r, blend.r),
                _overlay_channel(base.g, blend.g),
                _overlay_channel(base.b, blend.b),
                base.a
            )
            return base.lerp(result, blend.a)
        
        # ... other blend modes

func _overlay_channel(base: float, blend: float) -> float:
    if base < 0.5:
        return 2.0 * base * blend
    else:
        return 1.0 - 2.0 * (1.0 - base) * (1.0 - blend)
```

## Layer Compositing

### Rendering Pipeline

Layers are composited from bottom to top:

```gdscript
func composite_layers(layers: Array[Layer], output_size: Vector2i) -> Image:
    var result: Image = Image.create(
        output_size.x, 
        output_size.y, 
        false, 
        Image.FORMAT_RGBA8
    )
    result.fill(Color.TRANSPARENT)
    
    for layer in layers:
        if not layer.visible:
            continue
        
        composite_layer(result, layer)
    
    return result

func composite_layer(base: Image, layer: Layer) -> void:
    for y in range(layer.image.get_height()):
        for x in range(layer.image.get_width()):
            var pos: Vector2i = Vector2i(x, y)
            var base_color: Color = base.get_pixelv(pos)
            var layer_color: Color = layer.image.get_pixelv(pos)
            
            # Apply layer opacity
            layer_color.a *= layer.opacity
            
            # Blend colors
            var blended: Color = blend_colors(
                base_color, 
                layer_color, 
                layer.blend_mode,
                layer.opacity
            )
            
            base.set_pixelv(pos, blended)
```

### Optimization

Use dirty regions and caching for performance:

```gdscript
class_name LayerCompositor
extends RefCounted

var _composite_cache: Image = null
var _cache_valid: bool = false
var _dirty_layers: Array[int] = []

func invalidate_layer(index: int) -> void:
    if not _dirty_layers.has(index):
        _dirty_layers.append(index)
    _cache_valid = false

func get_composite(layers: Array[Layer]) -> Image:
    if _cache_valid:
        return _composite_cache
    
    if _dirty_layers.is_empty():
        # Full recomposite
        _composite_cache = composite_layers(layers, canvas_size)
    else:
        # Partial update
        for index in _dirty_layers:
            if index < layers.size():
                _update_layer_in_composite(layers[index], index)
        _dirty_layers.clear()
    
    _cache_valid = true
    return _composite_cache
```

## Layer Groups

### Group Hierarchy

```gdscript
class_name LayerGroup
extends Layer

var collapsed: bool = false
var children: Array[Layer] = []

func add_child(layer: Layer) -> void:
    layer.parent = self
    children.append(layer)

func remove_child(layer: Layer) -> void:
    layer.parent = null
    children.erase(layer)

func get_all_descendants() -> Array[Layer]:
    var result: Array[Layer] = []
    for child in children:
        result.append(child)
        if child is LayerGroup:
            result.append_array(child.get_all_descendants())
    return result
```

### Group Properties

Group properties affect all children:
- Opacity multiplies with child opacity
- Visibility hides all children
- Blend mode applies to entire group

```gdscript
func composite_group(group: LayerGroup, base: Image) -> void:
    if not group.visible:
        return
    
    # Composite children to temporary image
    var group_composite: Image = Image.create(
        base.get_width(),
        base.get_height(),
        false,
        Image.FORMAT_RGBA8
    )
    group_composite.fill(Color.TRANSPARENT)
    
    for child in group.children:
        if child is LayerGroup:
            composite_group(child, group_composite)
        else:
            composite_layer(group_composite, child)
    
    # Apply group properties and blend to base
    for y in range(base.get_height()):
        for x in range(base.get_width()):
            var pos: Vector2i = Vector2i(x, y)
            var base_color: Color = base.get_pixelv(pos)
            var group_color: Color = group_composite.get_pixelv(pos)
            group_color.a *= group.opacity
            
            var blended: Color = blend_colors(
                base_color,
                group_color,
                group.blend_mode,
                group.opacity
            )
            base.set_pixelv(pos, blended)
```

## Layer Transformations

### Transform Operations

```gdscript
# Flip layer horizontally
func flip_horizontal(layer: Layer) -> void:
    layer.image.flip_x()

# Flip layer vertically
func flip_vertical(layer: Layer) -> void:
    layer.image.flip_y()

# Rotate layer 90 degrees clockwise
func rotate_90_cw(layer: Layer) -> void:
    var old_image: Image = layer.image
    var new_width: int = old_image.get_height()
    var new_height: int = old_image.get_width()
    var new_image: Image = Image.create(new_width, new_height, false, Image.FORMAT_RGBA8)
    
    for y in range(old_image.get_height()):
        for x in range(old_image.get_width()):
            var old_pos: Vector2i = Vector2i(x, y)
            var new_pos: Vector2i = Vector2i(
                new_width - 1 - y,
                x
            )
            new_image.set_pixelv(new_pos, old_image.get_pixelv(old_pos))
    
    layer.image = new_image

# Move layer contents
func move_layer_contents(layer: Layer, offset: Vector2i) -> void:
    var old_image: Image = layer.image.duplicate()
    layer.image.fill(Color.TRANSPARENT)
    
    for y in range(old_image.get_height()):
        for x in range(old_image.get_width()):
            var old_pos: Vector2i = Vector2i(x, y)
            var new_pos: Vector2i = old_pos + offset
            
            if is_valid_position(new_pos, layer.image):
                layer.image.set_pixelv(new_pos, old_image.get_pixelv(old_pos))
```

## Layer Filters

Apply effects to layers:

```gdscript
# Adjust brightness/contrast
func adjust_brightness_contrast(layer: Layer, brightness: float, contrast: float) -> void:
    for y in range(layer.image.get_height()):
        for x in range(layer.image.get_width()):
            var pos: Vector2i = Vector2i(x, y)
            var color: Color = layer.image.get_pixelv(pos)
            
            # Adjust brightness
            color.r += brightness
            color.g += brightness
            color.b += brightness
            
            # Adjust contrast
            var factor: float = (1.0 + contrast) / 1.0
            color.r = ((color.r - 0.5) * factor) + 0.5
            color.g = ((color.g - 0.5) * factor) + 0.5
            color.b = ((color.b - 0.5) * factor) + 0.5
            
            # Clamp
            color.r = clamp(color.r, 0.0, 1.0)
            color.g = clamp(color.g, 0.0, 1.0)
            color.b = clamp(color.b, 0.0, 1.0)
            
            layer.image.set_pixelv(pos, color)

# Posterize (reduce colors)
func posterize(layer: Layer, levels: int) -> void:
    var step: float = 1.0 / float(levels - 1)
    
    for y in range(layer.image.get_height()):
        for x in range(layer.image.get_width()):
            var pos: Vector2i = Vector2i(x, y)
            var color: Color = layer.image.get_pixelv(pos)
            
            color.r = round(color.r / step) * step
            color.g = round(color.g / step) * step
            color.b = round(color.b / step) * step
            
            layer.image.set_pixelv(pos, color)
```

## Layer Masks

Future feature for non-destructive editing:

```gdscript
class_name LayerMask
extends RefCounted

var mask_image: Image
var enabled: bool = true

func apply_mask(layer_color: Color, mask_position: Vector2i) -> Color:
    if not enabled:
        return layer_color
    
    var mask_alpha: float = mask_image.get_pixelv(mask_position).r
    layer_color.a *= mask_alpha
    return layer_color
```

## Performance Considerations

### Best Practices

1. **Merge layers** when possible to reduce composite overhead
2. **Use layer groups** sparingly (each adds overhead)
3. **Limit blend modes** - NORMAL is fastest
4. **Cache composites** when layers haven't changed
5. **Use dirty region tracking** for partial updates

### Memory Management

```gdscript
func optimize_memory() -> void:
    # Clear undo history for deleted layers
    _clear_deleted_layer_history()
    
    # Compress layer images if needed
    for layer in layers:
        if not layer.visible and layer.can_compress:
            _compress_layer_image(layer)
    
    # Trigger garbage collection
    # (Godot handles this automatically)
```

## Future Enhancements

- [ ] Layer effects (drop shadow, glow, etc.)
- [ ] Layer masks for non-destructive editing
- [ ] Adjustment layers
- [ ] Smart objects/embedded files
- [ ] Layer comps/snapshots
- [ ] Linked layers
- [ ] Clipping masks
- [ ] Vector shape layers

## See Also

- [Layers User Guide](../user-guide/layers.md)
- [Drawing System](drawing-system.md)
- [Architecture](../development/architecture.md)
