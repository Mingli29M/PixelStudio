# Animation Timeline

Detailed documentation of PixelStudio's frame-by-frame animation system and timeline features.

## Overview

PixelStudio provides a cel-based animation system for creating frame-by-frame pixel art animations, inspired by traditional animation techniques and modern sprite animation tools.

## Animation Structure

### Frame-Based Animation

```gdscript
class_name Animation
extends Resource

signal frame_added(index: int)
signal frame_removed(index: int)
signal frame_changed(index: int)
signal playback_started()
signal playback_stopped()

@export var name: String = "Animation"
@export var frame_rate: float = 12.0  # FPS
@export var loop: bool = true

var frames: Array[Frame] = []
var current_frame: int = 0
var is_playing: bool = false
```

### Frame Structure

Each frame contains the state of all layers:

```gdscript
class_name Frame
extends Resource

@export var duration: int = 1  # Frame duration in ticks
@export var name: String = ""

var layer_states: Dictionary = {}  # layer_id -> Image

func get_layer_image(layer_id: int) -> Image:
    return layer_states.get(layer_id, null)

func set_layer_image(layer_id: int, image: Image) -> void:
    layer_states[layer_id] = image
```

## Timeline UI

### Timeline Components

```
Timeline Panel
├── Frame Strip        # Visual representation of frames
├── Layer List         # Layers in current frame
├── Playback Controls  # Play, pause, stop
├── Frame Rate Control # FPS settings
└── Onion Skin Toggle  # Preview previous/next frames
```

### Timeline Controls

```gdscript
class_name Timeline
extends Panel

signal frame_selected(index: int)
signal frame_duration_changed(index: int, duration: int)

@onready var frame_strip: HBoxContainer = $FrameStrip
@onready var playback_controls: HBoxContainer = $PlaybackControls
@onready var frame_rate_spin: SpinBox = $FrameRateSpinBox

var animation: Animation
var zoom: float = 1.0  # Timeline zoom level

func _ready() -> void:
    playback_controls.get_node("PlayButton").pressed.connect(_on_play_pressed)
    playback_controls.get_node("StopButton").pressed.connect(_on_stop_pressed)
    frame_rate_spin.value_changed.connect(_on_frame_rate_changed)
```

## Frame Operations

### Creating Frames

```gdscript
# Add new blank frame
func add_frame(index: int = -1) -> Frame:
    var frame: Frame = Frame.new()
    frame.name = "Frame " + str(frames.size() + 1)
    
    # Copy layer structure from current project
    for layer_id in _get_layer_ids():
        var blank_image: Image = Image.create(
            canvas_width,
            canvas_height,
            false,
            Image.FORMAT_RGBA8
        )
        blank_image.fill(Color.TRANSPARENT)
        frame.set_layer_image(layer_id, blank_image)
    
    if index < 0:
        frames.append(frame)
        index = frames.size() - 1
    else:
        frames.insert(index, frame)
    
    frame_added.emit(index)
    return frame

# Duplicate frame
func duplicate_frame(source_index: int) -> Frame:
    var source: Frame = frames[source_index]
    var new_frame: Frame = Frame.new()
    new_frame.name = source.name + " Copy"
    new_frame.duration = source.duration
    
    # Deep copy layer images
    for layer_id in source.layer_states.keys():
        var source_image: Image = source.get_layer_image(layer_id)
        new_frame.set_layer_image(layer_id, source_image.duplicate())
    
    var new_index: int = source_index + 1
    frames.insert(new_index, new_frame)
    frame_added.emit(new_index)
    return new_frame

# Delete frame
func delete_frame(index: int) -> void:
    if frames.size() <= 1:
        push_warning("Cannot delete last frame")
        return
    
    frames.remove_at(index)
    frame_removed.emit(index)
    
    # Adjust current frame if needed
    if current_frame >= frames.size():
        current_frame = frames.size() - 1
```

### Moving Frames

```gdscript
func move_frame(from_index: int, to_index: int) -> void:
    if from_index < 0 or from_index >= frames.size():
        return
    if to_index < 0 or to_index >= frames.size():
        return
    
    var frame: Frame = frames[from_index]
    frames.remove_at(from_index)
    frames.insert(to_index, frame)
    
    # Update current frame if needed
    if current_frame == from_index:
        current_frame = to_index

# Reverse frame order
func reverse_frames(start_index: int, end_index: int) -> void:
    var reversed_section: Array[Frame] = []
    for i in range(end_index, start_index - 1, -1):
        reversed_section.append(frames[i])
    
    for i in range(start_index, end_index + 1):
        frames[i] = reversed_section[i - start_index]
```

## Playback System

### Animation Player

```gdscript
class_name AnimationPlayer
extends Node

var animation: Animation
var current_frame: int = 0
var is_playing: bool = false
var playback_time: float = 0.0
var frame_time: float = 0.0

func _process(delta: float) -> void:
    if not is_playing or animation == null:
        return
    
    playback_time += delta
    var frame_duration: float = 1.0 / animation.frame_rate
    
    while playback_time >= frame_duration:
        playback_time -= frame_duration
        _advance_frame()

func _advance_frame() -> void:
    var frame: Frame = animation.frames[current_frame]
    frame_time += frame.duration
    
    if frame_time >= frame.duration:
        frame_time = 0
        current_frame += 1
        
        if current_frame >= animation.frames.size():
            if animation.loop:
                current_frame = 0
            else:
                current_frame = animation.frames.size() - 1
                stop()
        
        _update_canvas()

func play() -> void:
    is_playing = true
    animation.playback_started.emit()

func stop() -> void:
    is_playing = false
    current_frame = 0
    playback_time = 0.0
    frame_time = 0.0
    animation.playback_stopped.emit()

func pause() -> void:
    is_playing = false

func seek_to_frame(frame_index: int) -> void:
    if frame_index < 0 or frame_index >= animation.frames.size():
        return
    current_frame = frame_index
    playback_time = 0.0
    frame_time = 0.0
    _update_canvas()

func _update_canvas() -> void:
    # Update canvas to show current frame
    var frame: Frame = animation.frames[current_frame]
    canvas.load_frame(frame)
```

## Onion Skinning

### Onion Skin Renderer

```gdscript
class_name OnionSkin
extends Node2D

@export var enabled: bool = false
@export var previous_frames: int = 1  # Number of frames before
@export var next_frames: int = 1      # Number of frames after
@export var opacity_falloff: float = 0.3  # Opacity reduction per frame

var animation: Animation
var current_frame_index: int = 0

func _draw() -> void:
    if not enabled or animation == null:
        return
    
    # Draw previous frames in red tint
    for i in range(1, previous_frames + 1):
        var frame_index: int = current_frame_index - i
        if frame_index >= 0:
            var opacity: float = 1.0 - (i * opacity_falloff)
            _draw_frame(animation.frames[frame_index], Color(1, 0.5, 0.5, opacity))
    
    # Draw next frames in blue tint
    for i in range(1, next_frames + 1):
        var frame_index: int = current_frame_index + i
        if frame_index < animation.frames.size():
            var opacity: float = 1.0 - (i * opacity_falloff)
            _draw_frame(animation.frames[frame_index], Color(0.5, 0.5, 1, opacity))

func _draw_frame(frame: Frame, tint: Color) -> void:
    # Composite frame layers
    var composite: Image = _composite_frame_layers(frame)
    
    # Apply tint
    for y in range(composite.get_height()):
        for x in range(composite.get_width()):
            var pos: Vector2i = Vector2i(x, y)
            var color: Color = composite.get_pixelv(pos)
            color = color * tint
            composite.set_pixelv(pos, color)
    
    # Draw to canvas
    var texture: ImageTexture = ImageTexture.create_from_image(composite)
    draw_texture(texture, Vector2.ZERO)
```

## Frame Tags

Group and label frame ranges:

```gdscript
class_name FrameTag
extends Resource

@export var name: String = "Tag"
@export var start_frame: int = 0
@export var end_frame: int = 0
@export var color: Color = Color.WHITE
@export var animation_direction: Direction = Direction.FORWARD

enum Direction {
    FORWARD,      # Play start to end
    REVERSE,      # Play end to start
    PING_PONG,    # Play forward then backward
}

func is_frame_in_tag(frame_index: int) -> bool:
    return frame_index >= start_frame and frame_index <= end_frame
```

### Tag Manager

```gdscript
class_name TagManager
extends RefCounted

var tags: Array[FrameTag] = []

func add_tag(tag: FrameTag) -> void:
    tags.append(tag)

func get_tags_for_frame(frame_index: int) -> Array[FrameTag]:
    var result: Array[FrameTag] = []
    for tag in tags:
        if tag.is_frame_in_tag(frame_index):
            result.append(tag)
    return result

func play_tag(tag: FrameTag, player: AnimationPlayer) -> void:
    player.seek_to_frame(tag.start_frame)
    player.play()
    # Set loop region to tag range
```

## Frame Duration

Adjust individual frame timing:

```gdscript
# Variable frame duration
var frame_durations: Array[int] = [1, 1, 2, 1, 3]  # In ticks

func get_frame_time(frame_index: int, fps: float) -> float:
    var duration_ticks: int = frames[frame_index].duration
    return duration_ticks / fps

# Example: Frame with duration 2 at 12 FPS
# Time = 2 / 12 = 0.167 seconds (about 167ms)
```

## Export Animations

### GIF Export

```gdscript
func export_gif(path: String, settings: GIFExportSettings) -> bool:
    var gif_exporter: GIFExporter = GIFExporter.new()
    
    for frame in animation.frames:
        var composite: Image = _composite_frame_layers(frame)
        var duration_ms: int = int((frame.duration / animation.frame_rate) * 1000)
        gif_exporter.add_frame(composite, duration_ms)
    
    return gif_exporter.save(path, settings)
```

### Sprite Sheet Export

```gdscript
func export_sprite_sheet(path: String, layout: SheetLayout) -> bool:
    var frames_per_row: int = layout.columns
    var frame_count: int = animation.frames.size()
    var rows: int = ceili(float(frame_count) / float(frames_per_row))
    
    var sheet_width: int = canvas_width * frames_per_row + layout.spacing * (frames_per_row - 1)
    var sheet_height: int = canvas_height * rows + layout.spacing * (rows - 1)
    
    var sheet: Image = Image.create(sheet_width, sheet_height, false, Image.FORMAT_RGBA8)
    sheet.fill(Color.TRANSPARENT)
    
    for i in range(frame_count):
        var frame: Frame = animation.frames[i]
        var composite: Image = _composite_frame_layers(frame)
        
        var col: int = i % frames_per_row
        var row: int = i / frames_per_row
        var x: int = col * (canvas_width + layout.spacing)
        var y: int = row * (canvas_height + layout.spacing)
        
        sheet.blit_rect(composite, Rect2i(0, 0, canvas_width, canvas_height), Vector2i(x, y))
    
    return sheet.save_png(path) == OK
```

### Animation Sequence Export

```gdscript
func export_sequence(directory: String, prefix: String, padding: int = 3) -> bool:
    for i in range(animation.frames.size()):
        var frame: Frame = animation.frames[i]
        var composite: Image = _composite_frame_layers(frame)
        
        var frame_number: String = str(i + 1).pad_zeros(padding)
        var filename: String = prefix + "_" + frame_number + ".png"
        var path: String = directory.path_join(filename)
        
        if composite.save_png(path) != OK:
            return false
    
    return true
```

## Performance Optimization

### Frame Caching

```gdscript
var frame_cache: Dictionary = {}  # frame_index -> Image

func get_cached_frame(index: int) -> Image:
    if frame_cache.has(index):
        return frame_cache[index]
    
    var frame: Frame = animation.frames[index]
    var composite: Image = _composite_frame_layers(frame)
    frame_cache[index] = composite
    return composite

func invalidate_frame_cache(index: int) -> void:
    frame_cache.erase(index)

func clear_frame_cache() -> void:
    frame_cache.clear()
```

### Lazy Loading

```gdscript
# Only load frame data when needed
func load_frame_lazy(index: int) -> void:
    if not frames[index].is_loaded:
        frames[index].load_from_disk()
```

## Animation Tools

### Auto-frame

Automatically create frames from layer changes:

```gdscript
func auto_frame_from_layers() -> void:
    # Create one frame per unique layer state
    var unique_states: Array = []
    
    for layer in layers:
        if layer.has_changed_since_last_frame():
            var new_frame: Frame = create_frame_from_current_state()
            animation.add_frame(new_frame)
```

### Frame Interpolation

Generate in-between frames (tweening):

```gdscript
func interpolate_frames(start_index: int, end_index: int, steps: int) -> void:
    var start_frame: Frame = animation.frames[start_index]
    var end_frame: Frame = animation.frames[end_index]
    
    for step in range(1, steps + 1):
        var t: float = float(step) / float(steps + 1)
        var interpolated: Frame = _interpolate_frame(start_frame, end_frame, t)
        animation.frames.insert(start_index + step, interpolated)
```

## Future Features

- [ ] Bone-based animation system
- [ ] Motion paths
- [ ] Automatic inbetweening
- [ ] Video import/export
- [ ] Audio synchronization
- [ ] Advanced easing functions
- [ ] Layer animation (independent layer timelines)
- [ ] 3D preview mode

## See Also

- [Animation User Guide](../user-guide/animation.md)
- [Layer Management](layer-management.md)
- [Exporting](../user-guide/exporting.md)
