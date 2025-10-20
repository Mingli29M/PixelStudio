# Contributing to PixelStudio

Thank you for your interest in contributing to PixelStudio! This guide will help you get started with contributing to the project.

## Table of Contents

- [Code of Conduct](#code-of-conduct)
- [Getting Started](#getting-started)
- [Development Setup](#development-setup)
- [How to Contribute](#how-to-contribute)
- [Development Workflow](#development-workflow)
- [Coding Guidelines](#coding-guidelines)
- [Testing](#testing)
- [Documentation](#documentation)
- [Pull Request Process](#pull-request-process)
- [Community](#community)

## Code of Conduct

### Our Pledge

We are committed to providing a welcoming and inclusive environment for everyone, regardless of:
- Gender, gender identity and expression
- Sexual orientation
- Disability
- Physical appearance
- Race or ethnicity
- Age
- Religion or lack thereof
- Experience level

### Our Standards

**Positive behavior includes:**
- Using welcoming and inclusive language
- Being respectful of differing viewpoints
- Gracefully accepting constructive criticism
- Focusing on what's best for the community
- Showing empathy towards others

**Unacceptable behavior includes:**
- Harassment or discriminatory comments
- Personal or political attacks
- Public or private harassment
- Publishing others' private information
- Spam or off-topic comments

### Enforcement

Instances of unacceptable behavior can be reported to the project maintainers. All complaints will be reviewed and investigated, resulting in a response deemed necessary and appropriate.

## Getting Started

### Prerequisites

Before contributing, you should have:
- **Godot Engine 4.x** installed ([download here](https://godotengine.org/download))
- **Git** for version control
- Basic knowledge of **GDScript** (Godot's scripting language)
- Familiarity with **pixel art** concepts (helpful but not required)

### Finding an Issue

1. Browse [open issues](https://github.com/Mingli29M/PixelStudio/issues)
2. Look for issues labeled:
   - `good first issue` - Great for newcomers
   - `help wanted` - We need assistance
   - `bug` - Bug fixes needed
   - `enhancement` - New features
3. Comment on the issue to express interest
4. Wait for maintainer approval before starting work

## Development Setup

### Fork and Clone

1. **Fork** the repository on GitHub
2. **Clone** your fork locally:
   ```bash
   git clone https://github.com/YOUR_USERNAME/PixelStudio.git
   cd PixelStudio
   ```
3. **Add upstream** remote:
   ```bash
   git remote add upstream https://github.com/Mingli29M/PixelStudio.git
   ```

### Open in Godot

1. Launch **Godot Engine 4.x**
2. Click **Import**
3. Navigate to the cloned repository
4. Select `project.godot`
5. Click **Import & Edit**

### Verify Setup

1. The project should open without errors
2. Press `F5` to run the application
3. Verify basic functionality works

## How to Contribute

### Types of Contributions

**Code Contributions:**
- New features
- Bug fixes
- Performance improvements
- Code refactoring

**Non-Code Contributions:**
- Documentation improvements
- Bug reports
- Feature suggestions
- UI/UX design
- Pixel art assets
- Translations
- Testing

### Reporting Bugs

**Before reporting:**
1. Check if the bug is already reported
2. Try the latest development version
3. Verify it's not a configuration issue

**Bug report should include:**
- Clear, descriptive title
- Steps to reproduce
- Expected vs actual behavior
- Screenshots or videos (if applicable)
- Environment details:
  - OS and version
  - Godot version
  - PixelStudio version

**Example:**
```markdown
## Bug: Fill bucket doesn't work on transparent background

**Steps to reproduce:**
1. Create new project with transparent background
2. Select fill bucket tool
3. Click on canvas

**Expected:** Fill should apply to entire canvas
**Actual:** Nothing happens

**Environment:**
- OS: Windows 11
- Godot: 4.2.1
- PixelStudio: main branch (commit abc123)

**Screenshots:** [attached]
```

### Suggesting Features

**Feature requests should include:**
- Clear use case and motivation
- Detailed description of desired behavior
- Examples from other software (if applicable)
- Mockups or sketches (if UI-related)
- Consideration of implementation complexity

## Development Workflow

### Branching Strategy

1. **Create a feature branch:**
   ```bash
   git checkout -b feature/your-feature-name
   ```

   Branch naming conventions:
   - `feature/` - New features
   - `bugfix/` - Bug fixes
   - `refactor/` - Code refactoring
   - `docs/` - Documentation
   - `test/` - Tests

2. **Keep branch updated:**
   ```bash
   git fetch upstream
   git rebase upstream/main
   ```

### Making Changes

1. **Write code** following [Coding Standards](coding-standards.md)
2. **Test your changes** thoroughly
3. **Add tests** for new functionality
4. **Update documentation** if needed
5. **Commit changes** with clear messages

### Commit Messages

Follow conventional commit format:

```
<type>(<scope>): <subject>

<body>

<footer>
```

**Types:**
- `feat` - New feature
- `fix` - Bug fix
- `docs` - Documentation
- `style` - Code style (formatting)
- `refactor` - Code refactoring
- `test` - Tests
- `chore` - Maintenance

**Example:**
```
feat(tools): Add ellipse drawing tool

Implement ellipse drawing tool with support for:
- Perfect circles (Shift+drag)
- Filled and outline modes
- Configurable line width

Closes #42
```

### Code Quality

Before committing:
- [ ] Code follows [Coding Standards](coding-standards.md)
- [ ] All tests pass
- [ ] No console errors or warnings
- [ ] Code is properly commented
- [ ] Documentation is updated

## Coding Guidelines

Please review our detailed [Coding Standards](coding-standards.md) document.

**Quick checklist:**
- Use type hints everywhere
- Follow GDScript naming conventions
- Keep functions small and focused
- Comment complex logic
- Handle errors appropriately
- Avoid magic numbers
- Use signals over direct calls

## Testing

### Running Tests

```bash
# Using GUT (Godot Unit Test)
# Run from Godot editor: Scene -> Run Tests
```

### Writing Tests

All new features should include tests:

```gdscript
# test_my_feature.gd
extends GutTest

func test_feature_works() -> void:
    var result = my_function()
    assert_not_null(result)
    assert_eq(result, expected_value)
```

### Test Coverage

- Aim for **80%+ code coverage**
- All public APIs should be tested
- Critical paths must have tests
- Edge cases should be covered

## Documentation

### Code Documentation

Use doc comments for public APIs:

```gdscript
## Draws a line from start to end position.
##
## Uses Bresenham's algorithm for efficient line drawing.
## Color is taken from the current foreground color.
##
## @param start: Starting position in canvas coordinates
## @param end: Ending position in canvas coordinates
func draw_line(start: Vector2i, end: Vector2i) -> void:
    pass
```

### User Documentation

Update relevant docs in `docs/` directory:
- User guides for new features
- API documentation for developers
- Update README if needed

### Building Documentation

```bash
# Install MkDocs
pip install mkdocs mkdocs-material

# Serve locally
mkdocs serve

# Build
mkdocs build
```

## Pull Request Process

### Before Submitting

1. **Update from upstream:**
   ```bash
   git fetch upstream
   git rebase upstream/main
   ```

2. **Run tests:**
   - All tests pass
   - No new warnings

3. **Review your changes:**
   ```bash
   git diff upstream/main
   ```

### Creating Pull Request

1. **Push to your fork:**
   ```bash
   git push origin feature/your-feature-name
   ```

2. **Create PR on GitHub:**
   - Clear title describing the change
   - Reference related issues
   - Describe what changed and why
   - Include screenshots for UI changes
   - List breaking changes (if any)

3. **PR template:**
   ```markdown
   ## Description
   Brief description of changes

   ## Type of Change
   - [ ] Bug fix
   - [ ] New feature
   - [ ] Breaking change
   - [ ] Documentation

   ## Related Issues
   Fixes #123

   ## Testing
   - [ ] All tests pass
   - [ ] New tests added
   - [ ] Manually tested

   ## Screenshots
   (if applicable)

   ## Checklist
   - [ ] Code follows style guidelines
   - [ ] Self-reviewed code
   - [ ] Commented complex code
   - [ ] Documentation updated
   - [ ] No new warnings
   ```

### Review Process

1. **Automated checks** run on PR
2. **Maintainer review** - may request changes
3. **Address feedback** - make requested changes
4. **Approval** - PR approved by maintainer
5. **Merge** - Maintainer merges PR

### After Merge

1. **Delete branch:**
   ```bash
   git branch -d feature/your-feature-name
   git push origin --delete feature/your-feature-name
   ```

2. **Update local main:**
   ```bash
   git checkout main
   git pull upstream main
   ```

## Community

### Communication Channels

- **GitHub Issues** - Bug reports and feature requests
- **GitHub Discussions** - General questions and discussions
- **Pull Requests** - Code review and discussion

### Getting Help

- Check existing [documentation](https://mingli29m.github.io/PixelStudio/)
- Search [closed issues](https://github.com/Mingli29M/PixelStudio/issues?q=is%3Aissue+is%3Aclosed)
- Ask in [Discussions](https://github.com/Mingli29M/PixelStudio/discussions)

### Recognition

Contributors are recognized:
- In release notes
- In project credits
- With GitHub contributor badge

## License

By contributing, you agree that your contributions will be licensed under the [MIT License](../about/license.md).

## Questions?

Don't hesitate to ask! Open an issue or discussion if you have questions about:
- How to implement a feature
- Where code should go
- Best practices
- Anything else!

## Thank You!

Your contributions make PixelStudio better for everyone. We appreciate your time and effort!

## Next Steps

- Review [Architecture](architecture.md) for design understanding
- Check [Project Structure](project-structure.md) for code organization
- Read [Coding Standards](coding-standards.md) for code style
- Join the community and start contributing!
