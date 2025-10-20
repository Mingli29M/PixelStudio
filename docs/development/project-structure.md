# Project Structure

!!! info "Status"
    Placeholder page. Will be refined with actual directories as they stabilize.

## Proposed Layout
```
PixelStudio/
├─ addons/
├─ assets/
├─ docs/
├─ scenes/
│  ├─ ui/
│  ├─ canvas/
│  └─ tools/
├─ src/
│  ├─ core/
│  ├─ tools/
│  ├─ layers/
│  ├─ commands/
│  └─ io/
├─ tests/
├─ project.godot
```

## Conventions
- PascalCase for scenes, snake_case for scripts
- Signals prefixed with on_
- Type hints for public APIs

---