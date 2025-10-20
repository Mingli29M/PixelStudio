# File Formats

Documentation of file formats supported by PixelStudio for import, export, and project storage.

## Project Format

### .pxs (PixelStudio Project)

PixelStudio's native project format stores all project data including layers, frames, settings, and metadata.

#### File Structure

```json
{
  "version": "1.0.0",
  "meta": {
    "created": "2025-01-15T10:30:00Z",
    "modified": "2025-01-15T14:45:00Z",
    "author": "Artist Name",
    "application": "PixelStudio"
  },
  "canvas": {
    "width": 128,
    "height": 128,
    "background_color": [0, 0, 0, 0]
  },
  "layers": [
    {
      "id": 0,
      "name": "Background",
      "visible": true,
      "opacity": 1.0,
      "blend_mode": "normal",
      "locked": false,
      "image_data": "base64_encoded_png..."
    }
  ],
  "frames": [
    {
      "index": 0,
      "duration": 1,
      "name": "Frame 1",
      "layer_data": {
        "0": "base64_encoded_png..."
      }
    }
  ],
  "animation": {
    "frame_rate": 12.0,
    "loop": true,
    "tags": [
      {
        "name": "Walk",
        "start": 0,
        "end": 7,
        "color": [255, 100, 100]
      }
    ]
  },
  "palettes": [
    {
      "name": "Default",
      "colors": [
        [0, 0, 0, 255],
        [255, 255, 255, 255]
      ]
    }
  ],
  "settings": {
    "grid_size": 8,
    "show_grid": true,
    "show_pixel_grid": true
  }
}
```

#### Implementation

```gdscript
class_name ProjectFile
extends Resource

const VERSION: String = "1.0.0"

func save_project(path: String, project: Project) -> bool:
    var data: Dictionary = {
        "version": VERSION,
        "meta": _serialize_metadata(project),
        "canvas": _serialize_canvas(project),
        "layers": _serialize_layers(project),
        "frames": _serialize_frames(project),
        "animation": _serialize_animation(project),
        "palettes": _serialize_palettes(project),
        "settings": _serialize_settings(project)
    }
    
    var json_string: String = JSON.stringify(data, "\t")
    var file: FileAccess = FileAccess.open(path, FileAccess.WRITE)
    if file == null:
        push_error("Failed to open file for writing: " + path)
        return false
    
    file.store_string(json_string)
    file.close()
    return true

func load_project(path: String) -> Project:
    var file: FileAccess = FileAccess.open(path, FileAccess.READ)
    if file == null:
        push_error("Failed to open file for reading: " + path)
        return null
    
    var json_string: String = file.get_as_text()
    file.close()
    
    var json: JSON = JSON.new()
    var error: Error = json.parse(json_string)
    if error != OK:
        push_error("Failed to parse JSON: " + json.get_error_message())
        return null
    
    var data: Dictionary = json.data
    
    # Version check
    if not data.has("version") or data.version != VERSION:
        push_warning("Project version mismatch. Attempting to load anyway.")
    
    var project: Project = Project.new()
    _deserialize_metadata(project, data.meta)
    _deserialize_canvas(project, data.canvas)
    _deserialize_layers(project, data.layers)
    _deserialize_frames(project, data.frames)
    _deserialize_animation(project, data.animation)
    _deserialize_palettes(project, data.palettes)
    _deserialize_settings(project, data.settings)
    
    return project

func _serialize_layers(project: Project) -> Array:
    var layers_data: Array = []
    for layer in project.layers:
        var layer_dict: Dictionary = {
            "id": layer.id,
            "name": layer.name,
            "visible": layer.visible,
            "opacity": layer.opacity,
            "blend_mode": BlendMode.keys()[layer.blend_mode].to_lower(),
            "locked": layer.locked,
            "image_data": _image_to_base64(layer.image)
        }
        layers_data.append(layer_dict)
    return layers_data

func _image_to_base64(image: Image) -> String:
    var png_data: PackedByteArray = image.save_png_to_buffer()
    return Marshalls.raw_to_base64(png_data)

func _base64_to_image(base64: String) -> Image:
    var png_data: PackedByteArray = Marshalls.base64_to_raw(base64)
    var image: Image = Image.new()
    image.load_png_from_buffer(png_data)
    return image
```

## Image Formats

### PNG (Portable Network Graphics)

**Import:** ✅ **Export:** ✅

- Lossless compression
- Supports transparency
- Best for pixel art
- Preserves exact pixel colors

```gdscript
# Export PNG
func export_png(path: String, image: Image) -> bool:
    return image.save_png(path) == OK

# Import PNG
func import_png(path: String) -> Image:
    var image: Image = Image.new()
    var error: Error = image.load(path)
    if error != OK:
        push_error("Failed to load PNG: " + path)
        return null
    return image
```

### GIF (Graphics Interchange Format)

**Import:** ⚠️ (Limited) **Export:** ✅

- Animation support
- Maximum 256 colors
- Frame delays
- Looping options

```gdscript
class_name GIFExporter
extends RefCounted

var frames: Array[Image] = []
var delays: Array[int] = []  # Milliseconds

func add_frame(image: Image, delay_ms: int = 100) -> void:
    frames.append(image)
    delays.append(delay_ms)

func save(path: String, settings: Dictionary = {}) -> bool:
    # Settings:
    # - loop: bool (default: true)
    # - dithering: bool (default: false)
    # - palette_size: int (default: 256)
    
    var gif_data: PackedByteArray = _encode_gif(frames, delays, settings)
    
    var file: FileAccess = FileAccess.open(path, FileAccess.WRITE)
    if file == null:
        return false
    
    file.store_buffer(gif_data)
    file.close()
    return true
```

### JPEG/JPG

**Import:** ✅ **Export:** ❌

- Lossy compression
- No transparency
- Not recommended for pixel art
- Import only for reference images

## Palette Formats

### GPL (GIMP Palette)

**Import:** ✅ **Export:** ✅

Text-based palette format used by GIMP and other tools.

```
GIMP Palette
Name: My Palette
Columns: 16
#
  0   0   0  Black
255 255 255  White
255   0   0  Red
  0 255   0  Green
  0   0 255  Blue
```

```gdscript
func import_gpl(path: String) -> Palette:
    var file: FileAccess = FileAccess.open(path, FileAccess.READ)
    if file == null:
        return null
    
    var palette: Palette = Palette.new()
    var line: String = file.get_line()
    
    # Check header
    if line != "GIMP Palette":
        push_error("Invalid GPL file: missing header")
        return null
    
    # Parse metadata
    while not file.eof_reached():
        line = file.get_line().strip_edges()
        if line.is_empty() or line.begins_with("#"):
            continue
        
        if line.begins_with("Name:"):
            palette.name = line.substr(5).strip_edges()
        elif line.begins_with("Columns:"):
            # Optional column hint
            pass
        else:
            # Parse color line: R G B [Name]
            var parts: PackedStringArray = line.split(" ", false)
            if parts.size() >= 3:
                var r: int = parts[0].to_int()
                var g: int = parts[1].to_int()
                var b: int = parts[2].to_int()
                var color: Color = Color8(r, g, b, 255)
                
                var name: String = ""
                if parts.size() > 3:
                    name = " ".join(parts.slice(3))
                
                palette.add_color(color, name)
    
    file.close()
    return palette

func export_gpl(path: String, palette: Palette) -> bool:
    var file: FileAccess = FileAccess.open(path, FileAccess.WRITE)
    if file == null:
        return false
    
    file.store_line("GIMP Palette")
    file.store_line("Name: " + palette.name)
    file.store_line("Columns: 16")
    file.store_line("#")
    
    for i in range(palette.size()):
        var color: Color = palette.get_color(i)
        var r: int = int(color.r * 255)
        var g: int = int(color.g * 255)
        var b: int = int(color.b * 255)
        var name: String = palette.get_color_name(i)
        
        file.store_line("%3d %3d %3d  %s" % [r, g, b, name])
    
    file.close()
    return true
```

### ASE (Aseprite Palette)

**Import:** ✅ **Export:** ✅

Binary palette format from Aseprite.

```gdscript
func import_ase_palette(path: String) -> Palette:
    var file: FileAccess = FileAccess.open(path, FileAccess.READ)
    if file == null:
        return null
    
    # Read ASE file format
    var file_size: int = file.get_32()
    var magic: int = file.get_16()
    if magic != 0xA5E0:  # ASE magic number
        push_error("Invalid ASE file")
        return null
    
    var frames: int = file.get_16()
    var width: int = file.get_16()
    var height: int = file.get_16()
    var color_depth: int = file.get_16()
    
    # Read chunks and extract palette
    var palette: Palette = Palette.new()
    # ... parse ASE chunks for palette data
    
    file.close()
    return palette
```

### ACT (Adobe Color Table)

**Import:** ✅ **Export:** ✅

Simple 256-color palette format.

```gdscript
func import_act(path: String) -> Palette:
    var file: FileAccess = FileAccess.open(path, FileAccess.READ)
    if file == null:
        return null
    
    var palette: Palette = Palette.new()
    
    # ACT files contain 256 RGB triplets (768 bytes)
    for i in range(256):
        var r: int = file.get_8()
        var g: int = file.get_8()
        var b: int = file.get_8()
        palette.add_color(Color8(r, g, b, 255))
    
    file.close()
    return palette

func export_act(path: String, palette: Palette) -> bool:
    var file: FileAccess = FileAccess.open(path, FileAccess.WRITE)
    if file == null:
        return false
    
    # Write 256 colors (pad if necessary)
    for i in range(256):
        var color: Color = palette.get_color(i) if i < palette.size() else Color.BLACK
        file.store_8(int(color.r * 255))
        file.store_8(int(color.g * 255))
        file.store_8(int(color.b * 255))
    
    file.close()
    return true
```

## Sprite Sheet Formats

### Texture Atlas

Export frames as a single image with metadata.

#### PNG + JSON

```json
{
  "frames": [
    {
      "filename": "frame_0.png",
      "frame": {"x": 0, "y": 0, "w": 32, "h": 32},
      "rotated": false,
      "trimmed": false,
      "spriteSourceSize": {"x": 0, "y": 0, "w": 32, "h": 32},
      "sourceSize": {"w": 32, "h": 32}
    }
  ],
  "meta": {
    "app": "PixelStudio",
    "version": "1.0",
    "image": "spritesheet.png",
    "format": "RGBA8888",
    "size": {"w": 256, "h": 256},
    "scale": "1"
  }
}
```

#### PNG + XML

```xml
<?xml version="1.0" encoding="UTF-8"?>
<TextureAtlas imagePath="spritesheet.png">
    <SubTexture name="frame_0" x="0" y="0" width="32" height="32"/>
    <SubTexture name="frame_1" x="32" y="0" width="32" height="32"/>
</TextureAtlas>
```

## Import/Export Options

### Import Options

```gdscript
class_name ImportOptions
extends Resource

@export var resize_to_canvas: bool = false
@export var preserve_transparency: bool = true
@export var as_new_layer: bool = true
@export var center_image: bool = false
@export var dithering: bool = false
```

### Export Options

```gdscript
class_name ExportOptions
extends Resource

@export var format: Format = Format.PNG
@export var scale: int = 1  # 1x, 2x, 4x, etc.
@export var include_hidden_layers: bool = false
@export var flatten: bool = true
@export var background_color: Color = Color.TRANSPARENT

enum Format {
    PNG,
    GIF,
    SPRITE_SHEET,
    ANIMATION_SEQUENCE,
}
```

## Batch Operations

### Batch Import

```gdscript
func batch_import(paths: Array[String], options: ImportOptions) -> Array[Layer]:
    var layers: Array[Layer] = []
    for path in paths:
        var image: Image = import_png(path)
        if image:
            var layer: Layer = Layer.new()
            layer.name = path.get_file().get_basename()
            layer.image = image
            layers.append(layer)
    return layers
```

### Batch Export

```gdscript
func batch_export(layers: Array[Layer], directory: String, format: String = "png") -> bool:
    for layer in layers:
        var filename: String = layer.name + "." + format
        var path: String = directory.path_join(filename)
        if not export_png(path, layer.image):
            return false
    return true
```

## File Format Detection

```gdscript
func detect_file_format(path: String) -> String:
    var extension: String = path.get_extension().to_lower()
    
    match extension:
        "pxs":
            return "PixelStudio Project"
        "png":
            return "PNG Image"
        "gif":
            return "GIF Animation"
        "gpl":
            return "GIMP Palette"
        "ase", "aseprite":
            return "Aseprite File"
        "act":
            return "Adobe Color Table"
        _:
            return "Unknown"

func is_supported_format(path: String) -> bool:
    var extension: String = path.get_extension().to_lower()
    return extension in ["pxs", "png", "gif", "gpl", "ase", "aseprite", "act"]
```

## Future Format Support

- [ ] Photoshop (PSD) import
- [ ] Krita (KRA) import
- [ ] SVG export for scalable graphics
- [ ] MP4/WebM video export
- [ ] APX (Pixaki format)
- [ ] Tiled TMX map format
- [ ] Custom plugin format support

## Best Practices

1. **Save often** using .pxs format to preserve all data
2. **Export PNG** for final pixel-perfect images
3. **Use GIF** for simple animations with limited colors
4. **Sprite sheets** for game engine integration
5. **Backup projects** before format conversion
6. **Test imports** in target application before finalizing

## See Also

- [Exporting Guide](../user-guide/exporting.md)
- [Animation Timeline](animation-timeline.md)
- [Project Structure](../development/project-structure.md)
