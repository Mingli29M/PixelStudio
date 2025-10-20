# Changelog

All notable changes to PixelStudio will be documented in this file.

The format is based on [Keep a Changelog](https://keepachangelog.com/en/1.0.0/),
and this project adheres to [Semantic Versioning](https://semver.org/spec/v2.0.0.html).

## [Unreleased]

### Added
- Initial project structure and repository setup
- Comprehensive documentation system using MkDocs
  - Getting Started guides (Installation, Quick Start, Building)
  - User Guide (Interface, Drawing Tools, Layers, Animation, Palettes, Exporting, Shortcuts)
  - Development documentation (Architecture, Project Structure, Coding Standards, Contributing)
  - Features documentation (Drawing System, Layer Management, Animation Timeline, File Formats)
  - About section (Roadmap, License, Changelog)
- GitHub Actions workflow for automated documentation deployment
- Project README with feature overview
- Contributing guidelines
- MIT License

### Changed
- Nothing yet

### Deprecated
- Nothing yet

### Removed
- Nothing yet

### Fixed
- Nothing yet

### Security
- Nothing yet

---

## Release History

Currently in pre-alpha development. No releases yet.

### Planned Releases

See [Roadmap](roadmap.md) for planned features and timeline.

**Version 1.0.0** (Target: Q2 2026)
- First stable public release
- Core drawing system
- Layer management
- Animation timeline
- Export capabilities
- Cross-platform support (Windows, macOS, Linux)

---

## Version Numbering

PixelStudio follows [Semantic Versioning](https://semver.org/):

- **MAJOR** version for incompatible API changes
- **MINOR** version for backwards-compatible functionality additions
- **PATCH** version for backwards-compatible bug fixes

**Example:** Version 1.2.3
- **1** = Major version
- **2** = Minor version
- **3** = Patch version

### Pre-release Versions

During development, versions may include pre-release identifiers:

- **alpha** - Very early development, unstable
- **beta** - Feature complete, testing phase
- **rc** - Release candidate, nearly ready

**Example:** 1.0.0-alpha.1, 1.0.0-beta.2, 1.0.0-rc.1

---

## Changelog Format

Each release will document changes in these categories:

### Added
New features and capabilities added to PixelStudio.

### Changed
Changes to existing functionality or behavior.

### Deprecated
Features that are marked for removal in future versions.

### Removed
Features that have been removed from PixelStudio.

### Fixed
Bug fixes and corrections.

### Security
Security improvements and vulnerability fixes.

---

## Future Changelog Entries

Once development begins, entries will look like this:

```markdown
## [0.1.0] - 2025-03-15

### Added
- Basic canvas with drawing capabilities
- Pencil tool for pixel-perfect drawing
- Eraser tool for removing pixels
- Fill bucket with flood fill algorithm
- Layer creation and management
- Basic file save/load (PNG)
- Color picker with HSV support

### Fixed
- Issue #12: Canvas not resizing properly
- Issue #15: Fill bucket crashes on large areas

## [0.2.0] - 2025-04-20

### Added
- Line drawing tool with Bresenham's algorithm
- Rectangle and circle shape tools
- Selection tools (rectangular, ellipse)
- Undo/redo system with command pattern
- Keyboard shortcuts for all tools
- Zoom and pan controls

### Changed
- Improved canvas rendering performance by 50%
- Updated UI theme to be more modern

### Fixed
- Issue #23: Memory leak in layer system
- Issue #27: Undo stack not clearing on new project
```

---

## How to Contribute to Changelog

When making changes:

1. **Add entry to Unreleased section** at the top
2. **Use appropriate category** (Added, Changed, Fixed, etc.)
3. **Reference issue numbers** when applicable (#123)
4. **Be clear and concise** about what changed
5. **Focus on user impact** not technical details

**Good examples:**
```markdown
### Added
- Magic wand selection tool with tolerance setting (#45)
- Export to GIF with animation support (#67)

### Fixed
- Fixed crash when opening corrupted project files (#89)
- Corrected color picker not updating preview (#92)
```

**Less helpful:**
```markdown
### Added
- Implemented new selection algorithm

### Fixed
- Fixed bug
```

---

## Release Process

When creating a release:

1. **Move Unreleased changes** to new version section
2. **Add release date** in YYYY-MM-DD format
3. **Add version comparison links** at bottom
4. **Tag release** in Git: `git tag v1.0.0`
5. **Create GitHub release** with changelog excerpt

---

## Staying Updated

Follow PixelStudio development:

- **GitHub Releases:** [Releases Page](https://github.com/Mingli29M/PixelStudio/releases)
- **GitHub Watch:** Click "Watch" → "Releases only"
- **RSS Feed:** Subscribe to releases feed
- **Roadmap:** Check [Roadmap](roadmap.md) for planned features

---

## Changelog Guidelines

We follow [Keep a Changelog](https://keepachangelog.com/) principles:

1. **For humans, not machines** - Written for users, not parsers
2. **One entry per version** - Each version has its own section
3. **Same types of changes grouped** - Organized by category
4. **Latest version first** - Newest at top
5. **Release dates** - ISO format (YYYY-MM-DD)
6. **Version numbers** - Follow Semantic Versioning

---

## Deprecation Policy

When features are deprecated:

1. **Announced** in Minor version release
2. **Documented** in Deprecated section
3. **Removed** in next Major version release
4. **Alternative** provided in documentation

**Minimum deprecation period:** One major version cycle

---

## Breaking Changes

Breaking changes are clearly marked:

```markdown
### Changed
- **BREAKING:** Layer blend modes now use different enum values (#123)
  - Migration guide: docs/migration/v2.md
  - Old values are automatically converted on project load
```

---

## Experimental Features

Features marked as experimental may change or be removed:

```markdown
### Added
- **EXPERIMENTAL:** Real-time collaboration (#234)
  - Enable in Preferences → Experimental Features
  - Subject to change based on feedback
```

---

## Version Comparison Links

When releases are created, comparison links will be added:

```markdown
[Unreleased]: https://github.com/Mingli29M/PixelStudio/compare/v1.0.0...HEAD
[1.0.0]: https://github.com/Mingli29M/PixelStudio/compare/v0.9.0...v1.0.0
[0.9.0]: https://github.com/Mingli29M/PixelStudio/compare/v0.8.0...v0.9.0
```

---

## Questions?

- **Bug Reports:** [Open an issue](https://github.com/Mingli29M/PixelStudio/issues/new?template=bug_report.md)
- **Feature Requests:** [Open an issue](https://github.com/Mingli29M/PixelStudio/issues/new?template=feature_request.md)
- **Questions:** [GitHub Discussions](https://github.com/Mingli29M/PixelStudio/discussions)

---

## See Also

- [Roadmap](roadmap.md) - Future plans and timeline
- [Contributing](../development/contributing.md) - How to contribute
- [Releases](https://github.com/Mingli29M/PixelStudio/releases) - Download releases

---

**Last Updated:** January 2025  
**Current Version:** Unreleased (Pre-Alpha)
