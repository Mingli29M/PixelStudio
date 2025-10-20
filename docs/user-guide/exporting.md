# Exporting

Learn how to export your pixel art creations from PixelStudio in various formats.

## Export Formats

### PNG Export

Export your artwork as a PNG image file, perfect for web use, game assets, or sharing on social media.

**Steps:**
1. Go to **File > Export > PNG**
2. Choose your export settings:
   - **Resolution**: Original size or scaled up (2x, 4x, 8x)
   - **Transparency**: Preserve or remove transparency
   - **Background**: Apply background color if transparency is removed
3. Select the destination folder and filename
4. Click **Save**

**Tips:**
- Use original size for game assets
- Scale up (4x or 8x) for high-quality prints or display
- Keep transparency for assets that need alpha channels

### GIF Export

Create animated GIFs from your animation timeline.

**Steps:**
1. Ensure your animation timeline has multiple frames
2. Go to **File > Export > GIF**
3. Configure settings:
   - **Frame Rate**: FPS for the animation
   - **Loop**: Enable looping or set loop count
   - **Dithering**: Apply dithering for better color quality
   - **Palette**: Use project palette or generate optimized palette
4. Preview the animation
5. Click **Export**

**Best Practices:**
- Keep GIF file sizes small by limiting colors
- Use lower frame rates (8-12 FPS) for smaller files
- Enable dithering for smoother gradients

### Sprite Sheet Export

Export all animation frames as a single sprite sheet, ideal for game engines.

**Steps:**
1. Go to **File > Export > Sprite Sheet**
2. Choose layout options:
   - **Grid Layout**: Horizontal, vertical, or grid
   - **Spacing**: Padding between frames
   - **Frame Order**: Left-to-right, top-to-bottom
3. Select whether to include metadata:
   - **JSON**: Frame coordinates and dimensions
   - **XML**: Alternative format for frame data
4. Set output format (PNG)
5. Click **Export**

**Use Cases:**
- Game engine integration (Unity, Godot, GameMaker)
- Web animations using CSS or JavaScript
- Texture atlases for efficient rendering

### Animation Sequence Export

Export individual frames as separate PNG files.

**Steps:**
1. Go to **File > Export > Animation Sequence**
2. Configure naming:
   - **Prefix**: Base filename
   - **Numbering**: Frame numbers (001, 002, etc.)
   - **Padding**: Number of digits for frame numbers
3. Choose resolution and format options
4. Select destination folder
5. Click **Export**

**Useful For:**
- Video editing workflows
- Frame-by-frame processing
- Creating flipbooks

## Export Settings

### Resolution Scaling

- **1x**: Original pixel-perfect resolution
- **2x**: Double the resolution (good for HD displays)
- **4x**: Quadruple the resolution (high quality)
- **8x**: Eight times the resolution (print quality)

**Note:** Use nearest-neighbor scaling to maintain pixel-perfect appearance.

### Color Settings

- **Color Space**: sRGB or Linear
- **Bit Depth**: 8-bit or 16-bit per channel
- **Alpha Channel**: Preserve, remove, or premultiply

### Compression

- **PNG Compression**: Choose compression level (0-9)
- **GIF Colors**: Limit color palette (2-256 colors)

## Batch Export

Export multiple files or variations at once.

**Steps:**
1. Go to **File > Batch Export**
2. Add files or select layers to export
3. Configure export settings for all items
4. Choose destination folder
5. Click **Export All**

## Export Presets

Save frequently used export settings as presets.

**Creating a Preset:**
1. Configure your export settings
2. Click **Save as Preset**
3. Name your preset
4. Click **Save**

**Using a Preset:**
1. In any export dialog, click **Load Preset**
2. Select your saved preset
3. Adjust any settings if needed
4. Export

## Quick Export

Use keyboard shortcuts for quick exports:

- `Ctrl+E` - Export with last settings
- `Ctrl+Shift+E` - Open export dialog
- `Ctrl+Alt+E` - Quick export to default location

## Export Tips

- Always save your project (.pxs) before exporting
- Preview exports before finalizing large batches
- Keep original resolution exports for archival purposes
- Use descriptive filenames for organization
- Consider your target platform's requirements
- Test exports in target application before finalizing

## Troubleshooting

**Colors look different after export:**
- Check color space settings (sRGB vs Linear)
- Verify bit depth settings
- Ensure proper color profile embedding

**Transparency not working:**
- Verify PNG format supports transparency
- Check if alpha channel is preserved
- Remove background layers if needed

**Animation not smooth:**
- Adjust frame rate settings
- Check frame timing in timeline
- Verify all frames are included

**File size too large:**
- Reduce resolution if appropriate
- Optimize color palette for GIFs
- Use compression settings
- Consider alternative formats

## Next Steps

- Learn about [Animation](animation.md) to create animated content
- Explore [Layer Management](../features/layer-management.md) for complex compositions
- Check [File Formats](../features/file-formats.md) for format specifications
