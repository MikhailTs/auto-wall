# Auto-Wall Project Specification

**Version:** 1.3.1  
**Last Updated:** May 2026  
**Project Repository:** https://github.com/ThreeHats/auto-wall  
**Project Website:** https://autowallvtt.com

---

## 1. Project Overview

Auto-Wall is a cross-platform desktop application that automatically detects walls and light sources in TTRPG (Tabletop Role-Playing Game) battle maps using computer vision algorithms. The application exports detected walls in the Universal VTT (UVTT) format, enabling seamless integration with multiple virtual tabletop platforms including FoundryVTT, Roll20, Fantasy Grounds, and Arkenforge.

### Key Problem Being Solved
Manual wall tracing on detailed dungeon maps is time-consuming (20–30 minutes or more). Auto-Wall reduces this to minutes by using automated edge detection and color-based algorithms.

### Target Users
- TTRPG game masters and campaign organizers
- Map creators and publishers
- VTT administrators setting up dynamic lighting

---

## 2. Core Features

### 2.1 Automated Wall Detection
Two detection algorithms provide flexibility across different map styles:

- **Edge Detection (Canny Algorithm)**
  - Best for maps with clear wall outlines
  - Detects walls based on brightness gradients
  - Configurable sensitivity and threshold parameters
  - Edge margin parameter to exclude detection near image borders
  - Optional high-resolution processing for enhanced accuracy

- **Color Detection**
  - Best for maps with distinct wall/background colors
  - Multi-color support with per-color threshold configuration
  - Direct color picker tool for selecting colors from the image
  - Threshold parameter (0-100) controls color matching sensitivity

### 2.2 Light Source Detection
- Automatic identification of torches, lamps, and bright areas
- Brightness threshold configuration for light sensitivity
- Size filters (minimum and maximum) for light spot detection
- Optional color-based light detection
- Light merging by proximity distance
- Integrated light point data in UVTT exports

### 2.3 Manual Refinement Tools
Comprehensive drawing and editing capabilities in Paint Mode:

- **Drawing Tools:**
  - Brush tool with adjustable size for freehand drawing
  - Line tool for straight wall segments
  - Rectangle tool for structured wall creation
  - Circle and ellipse tools for curved features
  - Fill tool for area-based wall generation

- **Editing Modes:**
  - Draw mode: Add walls to the mask
  - Erase mode: Remove walls from the mask
  - Brush size adjustment for precise work
  - Undo/redo functionality for all operations

### 2.4 Detection Presets
Configuration management for rapid detection workflow:

- Save custom detection configurations with user-defined names
- Load previously saved detection presets
- Built-in default presets for common map scenarios
- Manage presets: rename, delete user-created presets
- Detection presets store: blur kernel, Canny thresholds, area filters, edge margins, color settings

### 2.5 Export Capabilities
Multi-format export with advanced configuration:

- **Universal VTT Export (Primary Format)**
  - JSON-based format compatible with all major VTTs
  - Wall segments with positioning and orientation
  - Light source data with range, intensity, and shadow settings
  - Grid alignment options (full-grid or half-grid)
  - Wall snapping to configurable grid sizes
  - Simplification tolerance for detail control
  - Maximum wall length parameter for VTT performance optimization
  - Export presets for saving common configurations
  - Wall endpoint merging by distance
  - Angle tolerance for straight wall detection

- **Export Presets**
  - Save and load export parameter configurations
  - Manage saved export presets
  - Apply presets to quickly regenerate exports with different settings

### 2.6 Interactive Preview System
- Visual preview of detected walls overlaid on the original image
- Real-time grid overlay visualization
- Grid position configuration (offset X/Y)
- Overlay transparency control
- Wall editing interface within preview:
  - Draw new walls by clicking endpoints
  - Edit existing wall positions
  - Delete individual walls
  - Portal/door placement tool
  - Mode indicator showing current operation

### 2.7 Cross-Platform Packaging
Native builds for multiple operating systems:

- **Windows:** Standalone .exe executable
- **Linux:** 
  - Debian package (.deb) for apt-based systems
  - AppImage for universal Linux distribution
- **macOS:** 
  - .dmg bundle with drag-to-install interface
  - Note: App is unsigned due to $99/year notarization cost

---

## 3. Architecture

### 3.1 Project Structure

```
auto_wall.py                    # Application entry point with splash screen
requirements.txt                # Python dependencies
build.py                        # Cross-platform build orchestration
Makefile                        # Development convenience targets

build/
├── build_utils.py             # Shared build utilities
└── platforms/
    ├── linux.py               # Linux-specific build logic
    ├── windows.py             # Windows-specific build logic
    └── macos.py               # macOS-specific build logic

src/
├── gui/                        # PyQt6 user interface components
│   ├── __init__.py
│   ├── app.py                 # Main application window (WallDetectionApp)
│   ├── image_viewer.py        # Interactive image display (InteractiveImageLabel)
│   ├── detection_panel.py     # Detection mode UI controls
│   ├── export_panel.py        # UVTT export and preview interface
│   ├── drawing_tools.py       # Paint mode drawing tools
│   ├── preset_manager.py      # Detection and export preset management
│   └── styles/
│       └── style.qss          # Qt stylesheet for UI theming

├── core/                       # Image and contour processing
│   ├── image_processor.py     # Image loading, scaling, preprocessing
│   ├── contour_processor.py   # Contour manipulation and scaling
│   ├── mask_processor.py      # Mask layer management for paint mode
│   ├── mode_manager.py        # Centralized tool and mode state management
│   └── selection.py           # Wall selection and interaction handling

├── wall_detection/            # Wall and light detection algorithms
│   ├── __init__.py
│   ├── detector.py            # Main detection algorithms (edge/color)
│   ├── image_utils.py         # Image preprocessing utilities
│   ├── light_detector.py      # Light source detection
│   ├── mask_editor.py         # Contour to wall conversion (UVTT format)
│   └── __pycache__/           # Python bytecode cache

└── utils/                      # Utility modules
    ├── __init__.py
    ├── debug_logger.py        # Debug logging and diagnostics
    ├── geometry.py            # Geometric calculations
    ├── output.py              # File I/O operations
    ├── performance.py         # Performance optimization utilities
    ├── svg_export.py          # SVG export functionality
    ├── ui_helpers.py          # Qt UI helper functions
    └── update_checker.py      # Auto-update checking

tests/                         # Unit test suite
├── __init__.py
├── test_detector.py          # Wall detection algorithm tests
├── test_image_utils.py       # Image utility function tests
└── test_webp_loading.py      # WebP image format tests

data/
├── input/                    # Test/demo input images
└── output/                   # Generated export files

resources/
├── create_splash.py          # Splash screen generation utility
└── icon.ico                  # Application icon

scripts/
├── detect_walls.py           # Standalone wall detection script
└── draw_walls.py             # Standalone wall drawing utility

spec/
└── HOTKEYS_SPEC.md          # Keyboard shortcut documentation

Configuration Files:
├── detection_presets.json    # Saved detection preset configurations
└── export_presets.json       # Saved export preset configurations
```

### 3.2 Core Components

#### Application Entry Point (`auto_wall.py`)
- Version management (currently 1.3.1)
- Base path resolution for PyInstaller bundles
- Logging setup with file-based output in user directories
- Splash screen display
- Update checker initialization
- Debug mode support

#### Main Application Window (`src/gui/app.py` - WallDetectionApp)
- PyQt6 QMainWindow-based main UI
- State management for:
  - Original and working images with scaling factor
  - Detected contours and light points
  - Mask layer for paint mode
  - Mode flags (deletion, color selection, edit mask, thin mode)
  - Display transformation parameters (scale, offset)
- Unified undo shortcut management
- Maximized window on launch

#### Detection Panel (`src/gui/detection_panel.py`)
- Edge detection parameter controls (blur, sensitivity, threshold, area filters)
- Color detection interface (color picker, threshold adjustment)
- Light detection configuration (threshold, size filters, color selection)
- Preset loading and preset-based parameter application
- High-resolution processing toggle
- Parameter-to-algorithm binding

#### Export Panel (`src/gui/export_panel.py`)
- UVTT export dialog with comprehensive parameter configuration
- Export parameters:
  - Simplification tolerance (0 = full detail, 0.0005-0.01+ = increasing smoothing)
  - Maximum wall segment length (5-500 pixels)
  - Maximum generation points (100-20000)
  - Point merge distance (0-500 pixels)
  - Angle tolerance for wall merging (0-30 degrees)
  - Maximum straight gap connection (0-100 pixels)
  - Grid snapping size (0 = disabled)
  - Half-grid position allowance
  - Grid offset (X/Y in grid units)
  - Grid overlay visualization (toggle, size, offset)
- Export preset management
- UVTT preview with interactive wall editing
- Wall export file saving (.uvtt or .json format)
- Clipboard copy functionality for JSON data

#### Image Processor (`src/core/image_processor.py`)
- Image loading from files or URLs
- Image scaling with maximum working dimension (1500px default)
- Caching with ImageCache for performance optimization
- Debounced update mechanism (250ms delay)
- Performance timing instrumentation
- Gaussian blur, edge detection, and Canny thresholding
- Contour extraction and hierarchy processing
- Touching contour detection and merging

#### Detector (`src/wall_detection/detector.py`)
**Edge-based Detection Pipeline:**
1. Grayscale conversion
2. Gaussian blur (configurable kernel size and disable option)
3. Canny edge detection (dual threshold)
4. Contour extraction with hierarchy support (RETR_CCOMP for inner contours)
5. Contour filtering by area (min/max)
6. Touching contour detection via dilation and connected components
7. Contour merging and splitting
8. Optional hatching line removal

**Color-based Detection Pipeline:**
1. Multi-color mask generation (per-color threshold support)
2. Contour extraction on color masks
3. Hierarchy processing (include both outer and inner contours)
4. Area-based filtering
5. Contour simplification and approximation

**Parameters:**
- min_contour_area, max_contour_area: Artifact filtering
- blur_kernel_size: 1 for no blur, odd values for blur
- canny_threshold1, canny_threshold2: Edge sensitivity
- edge_margin: Margin from image edges to exclude
- wall_colors: BGR color tuples with per-color thresholds
- color_threshold: Default threshold for color detection

#### Light Detector (`src/wall_detection/light_detector.py`)
**Detection Modes:**
1. Brightness-based: Threshold against grayscale intensity
2. Color-based: Match specific BGR colors with per-color thresholds

**Features:**
- Contour detection on brightness/color masks
- Area filtering (min/max)
- Light point centroid calculation
- Proximity merging (configurable merge distance)
- Light data structure:
  - Position (x, y coordinates)
  - Color information
  - Range setting
  - Intensity value
  - Shadow configuration

#### Mask Editor (`src/wall_detection/mask_editor.py`)
**Contour to Wall Conversion:**
- Contour simplification with tolerance parameter
- Wall endpoint merging by distance
- Angle tolerance for straight wall alignment
- Straight wall connection by maximum gap
- Wall segment length limiting
- Total wall point capping

**UVTT Export Function (`contours_to_uvtt_walls`):**
- Converts processed contours to UVTT format
- Outputs JSON structure with:
  - `line_of_sight`: Array of wall segments (each wall is array of [x, y] coordinates)
  - Light sources with position, range, intensity, shadow settings
  - Grid configuration data (pixels_per_grid, grid_offset)
- Grid snapping: full-grid and half-grid modes
- Grid offset support (X/Y in grid units)
- Optional overlay grid size preservation
- Preview pixel data embedding

#### Mode Manager (`src/core/mode_manager.py`)
**Enumerations:**
- `ToolType`: DETECT, PAINT, UVTT_EDITOR
- `DetectionMode`: EDGE, COLOR
- `SelectMode`: DELETE, THIN
- `PaintMode`: DRAW, ERASE
- `DrawingTool`: BRUSH, LINE, RECTANGLE, CIRCLE, ELLIPSE, FILL
- `UVTTMode`: DRAW_WALLS, EDIT_WALLS

**Purpose:** Centralized state management replacing scattered mode flags

### 3.3 Dependency Stack

**Runtime Dependencies:**
- `opencv-python`: Image processing and computer vision
- `numpy`: Numerical array operations
- `PyQt6`: Cross-platform GUI framework
- `Pillow`: Image file format support
- `scikit-learn`: Color clustering algorithms
- `matplotlib`: Plotting and visualization
- `requests`: HTTP requests for URL image loading

**Build Dependencies:**
- `pyinstaller>=5.6.2`: Executable packaging for all platforms

---

## 4. Functional Specifications

### 4.1 Application Workflow

#### Standard User Workflow

1. **Launch Application**
   - Displays splash screen
   - Initializes PyQt6 GUI
   - Loads detection and export presets from JSON files
   - Checks for application updates
   - Main window maximized to screen dimensions

2. **Load Image**
   - File menu → "Open Image" (file dialog) OR
   - Menu option for "Load from URL"
   - Image scaling applied if dimensions exceed max_working_dimension (1500px)
   - Scale factor calculated for later conversion between working and original coordinates

3. **Detection Mode (Detect Tab)**
   - Select detection algorithm: Edge or Color
   - Adjust parameters via detection panel sliders and controls
   - Real-time preview updates (debounced at 250ms)
   - Optional: Load detection preset to apply saved parameters
   - Optional: Save current parameters as new preset

4. **Manual Refinement (Paint Mode)**
   - Select Paint mode from left sidebar
   - Choose drawing tool (brush, line, rectangle, etc.)
   - Toggle between Draw (add walls) and Erase (remove walls)
   - Adjust brush size
   - Freehand drawing/erasing on mask layer
   - Undo/redo via Ctrl+Z

5. **Export to UVTT (Walls/UVTT Editor Mode)**
   - Switch to Walls/UVTT mode
   - Click "Export to UVTT" or menu item
   - Configuration dialog appears:
     - Load export preset (optional)
     - Adjust all export parameters
     - Select grid size and alignment options
     - Configure grid overlay visibility
   - Click OK to generate preview
   - Preview displayed with wall editing interface
   - Interactive wall editing:
     - Draw new walls by clicking endpoints
     - Edit/move existing walls
     - Delete specific walls
     - Add portals/doors
   - Save exported file: File menu → "Save File" (Ctrl+S)
   - Output file format: .uvtt (JSON-based) or .json

#### Navigation
- **Zoom:** Scroll wheel (forward = zoom in, backward = zoom out)
- **Pan:** Right-click and drag with mouse
- **View Reset:** Menu option for fit-to-window
- **Hotkeys:** Defined in `spec/HOTKEYS_SPEC.md`

### 4.2 Image Processing Pipeline

#### Input Image Processing

1. **Load & Decode**
   - Supported formats: JPEG, PNG, WebP, BMP, TIFF
   - RGB/BGR conversion handling

2. **Scaling**
   - If max(width, height) > max_working_dimension (1500):
     - Calculate scale_factor = original_max / max_working_dimension
     - Resize to max_working_dimension on largest axis
     - Store original_image and current_image separately

3. **Preprocessing for Detection**
   - Conversion to grayscale (for edge detection)
   - Gaussian blur application (kernel configurable, disable if size=1)
   - Canny edge detection or color mask generation

4. **Contour Extraction**
   - OpenCV findContours with RETR_CCOMP (retrieves hierarchy)
   - Hierarchy processing for inner contours
   - Area-based filtering (min/max area)

5. **Contour Processing**
   - Touching contour detection via morphological operations
   - Contour merging and split detection
   - Optional hatching line removal
   - Contour simplification via epsilon approximation

6. **Wall Generation**
   - Contours converted to wall segments
   - Endpoint merging by distance threshold
   - Straight wall connection detection (angle + gap tolerance)
   - Wall length limiting (max segment length parameter)
   - Total point capping (max_walls parameter)

#### Mask Layer (Paint Mode)
- RGBA image representing user-drawn walls
- Alpha channel: 0 = transparent (no wall), 255 = opaque (wall)
- Scaled to match current_image dimensions
- Converted back to original resolution for export
- Supports standard drawing operations:
  - Brush strokes (freehand)
  - Line segments (start/end click)
  - Geometric shapes (rectangle, circle, ellipse)
  - Fill operations (flood fill)
  - Erasing (drawing with alpha=0)

### 4.3 Export Format: Universal VTT (UVTT)

**Structure (JSON):**
```json
{
  "line_of_sight": [
    [[x1, y1], [x2, y2], [x3, y3], ...],  // Wall segment 1
    [[x4, y4], [x5, y5], ...],              // Wall segment 2
    ...
  ],
  "portals": [
    {"x": x, "y": y, "rotation": 0},
    ...
  ],
  "lights": [
    {
      "x": x,
      "y": y,
      "range": range_value,
      "intensity": intensity_value,
      "shadows": shadow_config,
      "color": "color_hex"
    },
    ...
  ],
  "pixels_per_grid": grid_size,
  "grid_offset_x": offset_x,
  "grid_offset_y": offset_y
}
```

**Wall Coordinate System:**
- X/Y coordinates in pixels
- Origin: top-left corner (0,0)
- Y increases downward (standard image coordinates)

**VTT Integration:**
- **FoundryVTT:** Via [Universal Battlemap Importer](https://foundryvtt.com/packages/dd-import/) module
- **Roll20:** Via [UniversalVTTImporter](https://wiki.roll20.net/Script:UniversalVTTImporter) API script
- **Arkenforge & Fantasy Grounds:** Native UVTT import support

### 4.4 State Management

**Application State (WallDetectionApp):**
```python
original_image          # Full resolution image
current_image           # Working resolution image (≤1500px max dimension)
scale_factor            # Ratio between original and current
processed_image         # Detection result overlay
current_contours        # List of detected/processed contours
current_lights          # List of detected light points
mask_layer              # RGBA mask for paint mode edits
display_scale_factor    # GUI zoom factor
display_offset          # Pan offset (x, y)
sliders                 # Detection parameter slider references
highlighted_contour_idx # UI hover state for contour highlighting
```

**Mode Flags:**
- `deletion_mode_enabled`: Wall deletion mode active
- `color_selection_mode_enabled`: Color detection mode
- `edit_mask_mode_enabled`: Paint mode drawing
- `thin_mode_enabled`: Wall thinning operation
- `uvtt_preview_active`: UVTT export preview displayed
- `uvtt_draw_mode`: Drawing new walls in UVTT preview
- `uvtt_edit_mode`: Editing existing walls in preview
- `uvtt_delete_mode`: Deleting walls in preview
- `uvtt_portal_mode`: Adding portals in preview

**Undo/Redo System:**
- History stack for all operations
- Unified undo shortcut (Ctrl+Z)
- Deque-based implementation for performance

### 4.5 GUI Layout

**Main Window Layout:**
```
┌─────────────────────────────────────────────────────────┐
│ Menu Bar: File, Edit, View, Help                       │
├─────────┬───────────────────────────┬─────────────────┤
│         │                           │                 │
│ Mode    │  Image Viewer (Center)    │  Detection or  │
│ Selector│  Interactive Canvas       │  Export Panel  │
│         │  (Zoom/Pan)               │  Controls      │
│ • Detect│  Grid Overlay (optional)  │  (Right)       │
│ • Paint │  Wall Preview             │                 │
│ • UVTT  │  (if in UVTT preview)     │                 │
│         │                           │                 │
│         │                           │                 │
├─────────┴───────────────────────────┴─────────────────┤
│ Status Bar: Detection info, mode indicators           │
└─────────────────────────────────────────────────────────┘
```

**Left Panel (Mode Selector):**
- Radio buttons or clickable tabs for: Detect, Paint, Walls/UVTT
- Mode-specific control switches

**Right Panel (Context-Sensitive):**
- **Detect Mode:** DetectionPanel with sliders and toggles
- **Paint Mode:** Brush settings, tool selector, draw/erase toggle
- **Walls/UVTT Mode:** ExportPanel with export parameters and preview controls

**Center Canvas:**
- InteractiveImageLabel (PyQt6 QLabel subclass)
- Scroll wheel zoom with cursor-centered scaling
- Right-click drag for panning
- Visual feedback for wall hover/selection
- Grid overlay (optional, configurable)

---

## 5. Testing Strategy

### 5.1 Test Coverage

**Unit Tests Location:** `tests/`

**Test Files:**

#### `test_detector.py`
- Tests for `src/wall_detection/detector.py`
- Test cases:
  - `test_detect_walls()`: Verifies wall contour detection on synthetic image
  - `test_draw_walls()`: Verifies contour drawing produces correct output shape
- Uses synthetic test image (100×100 px with white rectangle)

#### `test_image_utils.py`
- Tests for `src/wall_detection/image_utils.py`
- Test cases:
  - `test_preprocess_image()`: Verifies grayscale conversion (2D output)
  - `test_detect_edges()`: Verifies edge output is binary 2D image
  - `test_convert_to_rgb()`: Verifies RGB conversion

#### `test_webp_loading.py`
- Tests for WebP image format support
- Verifies image loading from WebP sources

### 5.2 Test Execution

**Run All Tests:**
```bash
python3 -m unittest discover -s tests -p 'test*.py' -v
```

**Run Specific Test File:**
```bash
python3 -m unittest tests.test_detector -v
```

**Via Makefile:**
```bash
make test
```

### 5.3 Testing Framework
- **Framework:** Python `unittest` (standard library)
- **Test Discovery:** Recursive directory scanning for test_*.py files
- **Assertions:** Standard unittest assertions (assertEqual, assertTrue, etc.)
- **Setup/Teardown:** setUp() method for test initialization

---

## 6. Build and Deployment

### 6.1 Build System

**Build Orchestration:** `build.py`
- Unified cross-platform build system
- Replaces separate platform-specific scripts
- Version extraction from `auto_wall.py`

**Platform-Specific Logic:** `build/platforms/`
- `windows.py`: .exe packaging via PyInstaller
- `linux.py`: .deb package (dpkg-deb) and AppImage creation
- `macos.py`: .dmg bundle creation

**Build Commands:**
```bash
# Current platform
python3 build.py

# Specific platform
python3 build.py --platform linux
python3 build.py --platform windows
python3 build.py --platform macos

# Dependency management
python3 build.py --install-deps

# Clean build artifacts
python3 build.py --clean
```

### 6.2 PyInstaller Configuration

**Entry Point:** `auto_wall.py`

**Resource Bundling:**
- Hooks for numpy integration (`numpy_hook.py`)
- Startup customization (`startup_hook.py`)
- Cross-platform base path resolution (PyInstaller bundle vs. script mode)

**Distribution Artifacts:**
- `dist/`: Final executable/package
- `build/pyinstaller_work/`: Build intermediates
- `.spec`: PyInstaller specification file

### 6.3 Platform-Specific Details

#### Windows
- Output: `Auto-Wall.exe`
- Code signing: Not implemented (self-signed executables)
- Installer: Standalone executable (no installer wrapper)

#### Linux
- **Debian Package (.deb)**
  - Output: `auto-wall_{version}_amd64.deb`
  - Installation: `sudo dpkg -i auto-wall_X.Y.Z_amd64.deb`
  - Command: `auto-wall` in PATH
  
- **AppImage**
  - Output: `Auto-Wall-{version}-x86_64.AppImage`
  - Execution: Direct run as executable
  - Portability: Works on any Linux distribution

#### macOS
- Output: `Auto-Wall-{version}.dmg`
- Installation: Drag-to-Applications
- Signing: Not notarized (security warning on first launch)
- First-run: User must approve in System Settings → Privacy & Security

### 6.4 Version Management
- Source version: `APP_VERSION = "1.3.1"` in `auto_wall.py`
- GitHub workflow: Auto-update version on release
- Semantic versioning: MAJOR.MINOR.PATCH

---

## 7. Performance Considerations

### 7.1 Optimization Techniques

**Image Caching:**
- ImageCache class in `src/utils/performance.py`
- Stores processed images for rapid retrieval
- Configurable cache size (default: 8 items)

**Debouncing:**
- 250ms debounce delay on slider updates
- Prevents excessive re-detection during parameter adjustment
- Improves GUI responsiveness

**Maximum Working Dimension:**
- Default: 1500px
- Large images auto-scaled to reduce processing time
- Original resolution preserved for export

**Performance Timing:**
- PerformanceTimer instrumentation in critical paths
- Debug logging of operation durations

### 7.2 Scalability Constraints

**Image Size Limits:**
- Working resolution: ≤1500px max dimension
- Original image: No hard limit (system RAM dependent)

**Wall/Point Limits:**
- Maximum generation points (configurable): 100-20000
- Default: 5000
- Prevents excessive wall count degrading VTT performance

---

## 8. Configuration and Presets

### 8.1 Detection Presets (`detection_presets.json`)

**File Location:** Project root directory

**Structure:**
```json
{
  "preset_name": {
    "blur_kernel_size": 5,
    "canny_threshold1": 50,
    "canny_threshold2": 150,
    "min_contour_area": 100,
    "max_contour_area": null,
    "edge_margin": 0,
    "smoothing": 5,
    "light_threshold": 0.8,
    "light_min_area": 5,
    "light_max_area": 500,
    "light_merge_distance": 20.0
  },
  ...
}
```

**Usage:**
- Loaded at application startup
- Selectable from detection panel dropdown
- User can save current parameters as new preset
- Built-in default presets provided

### 8.2 Export Presets (`export_presets.json`)

**Structure:**
```json
{
  "preset_name": {
    "simplify_tolerance": 0.0005,
    "max_wall_length": 50,
    "max_walls": 5000,
    "merge_distance": 25.0,
    "angle_tolerance": 1.0,
    "max_gap": 10.0,
    "grid_size": 0,
    "allow_half_grid": false,
    "grid_offset_x": 0.0,
    "grid_offset_y": 0.0,
    "show_grid_overlay": false,
    "overlay_grid_size": 70,
    "overlay_offset_x": 0.0,
    "overlay_offset_y": 0.0
  },
  ...
}
```

**Usage:**
- Loaded at application startup
- Selectable from export dialog dropdown
- User can save current export settings as new preset
- Applied to export parameters via PresetManager

### 8.3 Preset Management UI

**Detection Presets:**
- Load dropdown in DetectionPanel
- Save button to create new preset from current settings
- Manage button for preset administration (delete, rename)

**Export Presets:**
- Load dropdown in export dialog
- Save button during export workflow
- Manage button for preset administration

---

## 9. Error Handling and Logging

### 9.1 Logging Strategy

**Log Locations:**
- **Development:** Console output in debug mode
- **Production:** User directory (platform-specific):
  - Windows: `%APPDATA%\Auto-Wall\logs\`
  - Linux: `~/.local/share/auto-wall/logs/`
  - macOS: `~/Library/Application Support/Auto-Wall/logs/`

**Log Rotation:**
- Timestamped log files
- Automatic cleanup of old logs

**Debug Logger (`src/utils/debug_logger.py`):**
- `log_info()`: Informational messages
- Conditional output based on debug mode flag
- Performance timer integration

### 9.2 User-Facing Error Messages

**Common Errors:**
- No image loaded: "Please load an image first"
- No walls to export: "No walls detected. Please run detection first"
- Invalid file format: "Unsupported image format"
- Export presets missing: "No export presets found"

**Message Display:**
- QMessageBox dialogs (warning/error/information)
- Status bar messages for non-critical issues

### 9.3 Update Checker

**Module:** `src/utils/update_checker.py`

**Functionality:**
- Checks GitHub releases for new versions at startup
- Fetches version from `https://api.github.com/repos/ThreeHats/auto-wall/releases/latest`
- Notifies user if newer version available
- Provides link to download page

---

## 10. Platform-Specific Considerations

### 10.1 PyInstaller Bundle Handling

**Base Path Resolution (`auto_wall.py`):**
```python
def get_base_path():
    if getattr(sys, 'frozen', False):
        if hasattr(sys, '_MEIPASS') and sys.platform != "darwin":
            return sys._MEIPASS  # Windows/Linux onefile mode
        elif sys.platform == "darwin":
            return os.path.join(os.path.dirname(sys.executable), "..", "Resources")
        else:
            return os.path.dirname(sys.executable)
    else:
        return os.path.abspath(os.path.dirname(__file__))
```

**Resource Access:**
- Bundles include Qt plugins and resource files
- Cross-platform path handling ensures resource availability

### 10.2 Platform Quirks

**Windows:**
- File path separators: backslash (handled by pathlib.Path)
- Line endings: CRLF in source files
- Icon support: .ico format

**Linux:**
- File permissions: Executable flag for .AppImage
- Package installation: dpkg integration
- Desktop integration: Application menu registration

**macOS:**
- App bundle structure: `Auto-Wall.app/Contents/`
- Resource path: `Contents/Resources/`
- Security: Unsigned (user approval required on first launch)
- Intel vs. Apple Silicon: Currently x86_64 only

---

## 11. Dependencies and Requirements

### 11.1 Python Dependencies

**File:** `requirements.txt`

```
opencv-python           # Image processing
numpy                   # Numerical operations
matplotlib              # Visualization
Pillow                  # Image format support
pyQt6                   # GUI framework
scikit-learn            # Machine learning utilities
pyinstaller>=5.6.2      # Executable packaging
requests                # HTTP operations
```

### 11.2 System Requirements

**Minimum:**
- Python 3.8+
- 2 GB RAM (for typical map processing)
- 500 MB disk space

**Recommended:**
- Python 3.10+
- 4+ GB RAM (for high-resolution maps)
- 1 GB disk space

**Runtime Dependencies:**
- Qt6 runtime (included in PyQt6)
- OpenCV runtime (included in opencv-python)

---

## 12. Future Extensibility

### 12.1 Planned Features (Roadmap)

- [ ] Additional detection algorithms (Hough transforms, ML-based detection)
- [ ] Real-time detection preview with parameter adjustment
- [ ] Batch processing for multiple maps
- [ ] Theme customization
- [ ] Plugin system for custom detection algorithms
- [ ] Integration with online map libraries
- [ ] WebP-specific optimization
- [ ] Mobile companion app

### 12.2 Architecture for Extension

**Mode System:**
- New tool types can be added to `ToolType` enum
- New detection modes can be added to `DetectionMode` enum
- Mode-specific UI panels inherit from base class pattern

**Detection Algorithms:**
- New detection functions in `src/wall_detection/`
- Integration via detector.py main function
- Preset system supports new algorithm parameters

**UI Components:**
- PyQt6 allows custom widget creation
- Stylesheet system (style.qss) for consistent theming
- Panel-based layout supports dynamic control addition

---

## 13. Documentation and References

### 13.1 User Documentation
- **Main README:** `README.md` - Feature overview and quick start
- **Hotkey Reference:** `spec/HOTKEYS_SPEC.md` - Keyboard shortcuts
- **Website:** `https://autowallvtt.com` - Feature demos and getting started

### 13.2 API Documentation
- **Module docstrings:** Self-documenting code with comprehensive docstring blocks
- **Type hints:** Python type annotations throughout codebase
- **Demo scripts:** `scripts/detect_walls.py`, `scripts/draw_walls.py` for standalone usage

### 13.3 Project Resources
- **Repository:** `https://github.com/ThreeHats/auto-wall`
- **Releases:** `https://github.com/ThreeHats/auto-wall/releases`
- **Discord Community:** `https://discord.gg/HUzEnZy8uJ`
- **YouTube Demo:** `https://youtu.be/gqkEIWwuJX4`

---

## 14. Changelog Highlights (Current Version 1.3.1)

- Unified cross-platform build system (build.py)
- Universal UVTT export format with grid alignment
- Interactive wall editing in export preview
- Multi-color detection with per-color thresholds
- Light source detection with proximity merging
- Detection and export preset management
- Grid overlay visualization
- Performance optimizations (caching, debouncing)
- Comprehensive error handling and logging

---

## 15. Summary of Key Design Principles

1. **Cross-Platform First:** Same codebase for Windows, Linux, macOS
2. **User-Friendly:** Minimal learning curve with sensible defaults
3. **Performance Optimized:** Caching, debouncing, working resolution scaling
4. **Extensible:** Modular architecture with clear component boundaries
5. **Standards-Compliant:** Universal VTT format compatibility
6. **Open Source:** MIT license with community support
7. **Visual Feedback:** Real-time preview and interactive editing
8. **Configuration Flexible:** Extensive parameter adjustment with presets

---

*End of Specification Document*

Generated on: May 10, 2026  
Project Version: 1.3.1  
Specification Version: 1.0