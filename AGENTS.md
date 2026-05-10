# Auto-Wall AI Agent Guidance

## General agent guidelines
- Specification is referred by pseudo-URI: spec://[path.to.file][#header-or-anchor-id]. Path to file always relative to `spec/` folder, if omitted - use SPEC.md. Use this format when referencing specification in code, tests or chat.
- Tests are referred by pseudo-URI: tests://[path.to.file][#test-class-or-function]. Path to file always relative to `tests/` folder. Use this format when referencing test cases in specification, code or chat.

## Project Summary
Auto-Wall is a cross-platform Python desktop app for detecting walls and light sources in TTRPG battle maps and exporting Universal VTT data.

## Key goals for code changes
- Preserve the image processing and wall detection pipeline
- Keep the PyQt6 GUI responsive and cross-platform
- Maintain packaging support for Windows, Linux, and macOS
- Use existing test coverage in `tests/`

## Primary files and directories
- `auto_wall.py` — application entry point and splash flow
- `requirements.txt` — runtime dependencies
- `build.py` — cross-platform build orchestration
- `Makefile` — convenience build/test targets
- `src/gui/` — PyQt6 UI and panels
- `src/core/` — contour/image processing and mode management
- `src/wall_detection/` — detection algorithms and helpers
- `src/utils/` — utilities, logging, geometry, export, update checks
- `tests/` — unit tests using `unittest`

## Build and run commands
Use the repository root as the working directory.

### Setup
- `python3 -m pip install -r requirements.txt`

### Run app
- `python auto_wall.py`

### Build packages
- `python3 build.py` — build for current platform
- `python3 build.py --platform linux`
- `python3 build.py --platform windows`
- `python3 build.py --platform macos`
- `python3 build.py --install-deps`
- `python3 build.py --clean`

### Development tasks
- `python3 -m unittest discover -s tests -p 'test*.py' -v`
- `make test`

## Agent behavior guidance
- Prefer fixes that keep the existing `src/` module boundaries intact
- Use `PyQt6` conventions for GUI updates and event handling
- Verify CV/image-processing changes with the available tests and keep runtime performance in mind
- Do not modify generated artifacts in `dist/`, `build/pyinstaller_work/`, or other packaging outputs
- When editing build behavior, respect the cross-platform packaging strategy in `build.py` and `build/platforms/*`
- Use the local `.venv` virtual environment when possible, unless the user specifies otherwise.

## Useful references
- Project README: `README.md`
- Build script: `build.py`
- Test commands: `Makefile`
- CI workflow: `.github/workflows/build.yml`
- Project specification: `spec/*`
