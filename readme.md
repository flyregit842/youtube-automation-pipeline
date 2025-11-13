# YouTube Automation Pipeline

## Repository
https://github.com/flyregit842/youtube-automation-pipeline.git

## Files

### ytpipeline_202511121955.ipynb
Original notebook with known "black scene with falling effect" bug.

### ytpipeline_fixed.ipynb
**Fixed version** - Addresses all timing and overlay bugs. Use this for production.

### FIXES.md
Comprehensive documentation of all bug fixes, root causes, and technical details.

## Quick Start

1. Open `ytpipeline_fixed.ipynb` in Jupyter
2. Set your project directory in the `main_dir` variable
3. Run Step 0 to configure environment and settings
4. Run Step 1 to edit scenes and assign audio/images
5. Click "輸出影片" to render your video

## What Was Fixed

The fixed notebook resolves:
- ✅ Black frames at segment joins
- ✅ "Falling effect" artifacts with overlays
- ✅ Audio-video duration misalignment
- ✅ Overlay loop timing issues

All original UI and features are preserved. See `FIXES.md` for complete details.

## Requirements

- Python 3.7+
- FFmpeg with libass support
- Required packages: pyyaml, pillow, ipywidgets, python-dotenv
- Optional: API keys for TTS/AI services (OpenAI, Azure, Minimax)

## Features

- Multi-scene video production
- Text-to-speech integration (multiple providers)
- AI image generation
- Subtitle generation (hardburn or soft-sub)
- Post-processing effects (color correction, denoise, sharpen)
- Built-in overlay effects (snow, rain, leaves, custom)
- Profile presets (eco/balanced/ultra)
- Full editing UI with thumbnails
