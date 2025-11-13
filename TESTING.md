# Testing Guide for ytpipeline_fixed.ipynb

## Pre-Testing Checklist

Before testing the fixed notebook, ensure you have:

- [ ] Python 3.7+ installed
- [ ] Jupyter Notebook or JupyterLab installed
- [ ] FFmpeg installed and in PATH (with libass support for subtitles)
- [ ] Required Python packages: `pyyaml`, `pillow`, `ipywidgets`, `python-dotenv`
- [ ] A test project directory with:
  - `text.txt` with test scene text (one line per scene)
  - Test audio files (MP3/WAV) for each scene
  - Test images (PNG/JPG) for each scene
  - Optional: `.env` file with API keys for TTS/AI generation

## Test Scenarios

### Scenario 1: Basic Scene Rendering (Critical)

**Purpose**: Verify black frame bug is fixed

**Steps**:
1. Create a simple project with 3 scenes
2. Use different audio durations: 2.3s, 5.7s, 8.1s (non-integer durations)
3. Assign images to all scenes
4. Disable all overlays/effects
5. Render the video

**Expected Results**:
- ✅ No black frames between scenes
- ✅ Scene transitions are clean
- ✅ Audio-video sync is perfect
- ✅ Total video duration matches sum of audio durations

**Validation**:
```bash
ffprobe -v error -show_entries format=duration -of default=noprint_wrappers=1:nokey=1 output_final.mp4
```

---

### Scenario 2: Overlay Effects (Critical)

**Purpose**: Verify "falling effect" bug is fixed

**Steps**:
1. Use the same 3-scene project from Scenario 1
2. Enable built-in simple overlay (e.g., snow effect)
3. Set density=30, size=36, speed=120
4. Render the video

**Expected Results**:
- ✅ Overlay appears throughout the entire video
- ✅ No black frames or gaps in the overlay
- ✅ Overlay loops smoothly
- ✅ No "falling" or disappearing effects at scene boundaries

**Validation**:
Watch the video carefully at scene transitions. The snow/rain/leaves should continue seamlessly.

---

### Scenario 3: Advanced Overlay (Important)

**Purpose**: Verify advanced particle system overlays work correctly

**Steps**:
1. Create a project with 2 scenes (5s each = 10s total)
2. Enable built-in advanced overlay
3. Set loop_sec=5 (should loop twice for 10s video)
4. Render the video

**Expected Results**:
- ✅ Advanced overlay WebM is generated
- ✅ Overlay loops exactly 2 times
- ✅ No black frames at loop transitions
- ✅ Overlay stops when video ends (no extra frames)

**Validation**:
```bash
# Check overlay file was created
ls -lah _adv_overlay_*.webm

# Verify it loops correctly in the output
ffplay output_final.mp4
```

---

### Scenario 4: Custom Overlay File (Optional)

**Purpose**: Verify custom overlay files work

**Steps**:
1. Prepare a custom overlay video file (MP4/WebM)
2. Set overlay mode to "file" and select your file
3. Enable scale_to_output
4. Render the video

**Expected Results**:
- ✅ Custom overlay is loaded and applied
- ✅ Overlay is scaled to match output resolution
- ✅ Loops correctly if video is longer than overlay
- ✅ No black frames

---

### Scenario 5: Mixed Duration Scenes (Critical)

**Purpose**: Stress test duration alignment

**Steps**:
1. Create 5 scenes with widely varying durations:
   - Scene 1: 1.2s
   - Scene 2: 3.7s
   - Scene 3: 0.8s (very short)
   - Scene 4: 7.3s
   - Scene 5: 2.1s
2. Enable simple overlay
3. Render the video

**Expected Results**:
- ✅ All scenes render correctly
- ✅ No black frames anywhere
- ✅ Overlay continuous throughout
- ✅ Total duration = sum of individual durations

**Validation**:
```bash
# Check individual scene clips
for i in scene_*_clip.mp4; do
    echo "=== $i ==="
    ffprobe -v error -show_entries format=duration -of default=noprint_wrappers=1:nokey=1 "$i"
done

# Check final output
ffprobe -v error -show_entries format=duration -of default=noprint_wrappers=1:nokey=1 output_final.mp4
```

---

### Scenario 6: Profile Presets (Important)

**Purpose**: Verify all profile presets work

**Steps**:
1. Test with "省資源" (eco) profile
   - Should render at 720p/25fps
   - Simple overlay only
2. Test with "均衡" (balanced) profile
   - Should render at 1080p/25fps
   - Advanced overlay with half-resolution generation
3. Test with "頂規" (ultra) profile
   - Should render at 1080p/25fps (or custom 4K)
   - Advanced overlay with full resolution

**Expected Results**:
- ✅ Each profile renders correctly
- ✅ Resolution matches profile settings
- ✅ No black frames or artifacts in any profile
- ✅ Quality differences are visible between profiles

---

### Scenario 7: Progress Feedback (User Experience)

**Purpose**: Verify enhanced progress output

**Steps**:
1. Create a project with 10 scenes
2. Watch the console output during rendering
3. Note the progress messages

**Expected Results**:
- ✅ Progress shows "場景 001/010 (時長: X.XXs)"
- ✅ Can see which scene is being processed
- ✅ Can see duration of each scene
- ✅ Clear indication when rendering completes

---

## Validation Commands

### Check Video Duration
```bash
ffprobe -v error -show_entries format=duration -of default=noprint_wrappers=1:nokey=1 <video_file>
```

### Check Video Frame Count
```bash
ffprobe -v error -select_streams v:0 -show_entries stream=nb_frames -of default=noprint_wrappers=1:nokey=1 <video_file>
```

### Extract and View Frames (for detailed inspection)
```bash
# Extract frames at scene boundaries
ffmpeg -i output_final.mp4 -vf "select='gte(n\,frame_number)'" -vsync 0 frame_%04d.png

# For example, to check frame 100:
ffmpeg -i output_final.mp4 -vf "select='eq(n\,100)'" -vsync 0 frame_100.png
```

### Visual Inspection
```bash
# Play video with frame display
ffplay -vf "drawtext=text='Frame %{n}':x=10:y=10:fontsize=48:fontcolor=yellow" output_final.mp4

# Play at slow speed to check transitions
ffplay -vf "setpts=4*PTS" output_final.mp4
```

---

## Common Issues and Debugging

### Issue: "ffmpeg not found"
**Solution**: 
```bash
# Install ffmpeg (Ubuntu/Debian)
sudo apt-get install ffmpeg

# Or check if it's in PATH
which ffmpeg
```

### Issue: "No module named 'yaml'"
**Solution**:
```bash
pip install pyyaml pillow ipywidgets python-dotenv
```

### Issue: "Subtitle hardburn fails"
**Solution**:
- Check if ffmpeg has libass support: `ffmpeg -filters | grep subtitles`
- If not, install ffmpeg with libass or use soft subtitles (automatic fallback)

### Issue: "Overlay WebM generation is slow"
**Solution**:
- This is normal on first run
- WebM files are cached (check for `_adv_overlay_*.webm`)
- Subsequent renders reuse cached files

---

## Performance Benchmarks

For a typical 5-scene video (total ~15 seconds):

| Profile | Resolution | Expected Time | File Size |
|---------|-----------|---------------|-----------|
| 省資源 (eco) | 720p | ~30-60s | ~2-4 MB |
| 均衡 (balanced) | 1080p | ~60-120s | ~5-10 MB |
| 頂規 (ultra) | 1080p | ~120-300s | ~10-20 MB |

*Times vary based on CPU, overlay complexity, and codec settings*

---

## Reporting Issues

If you encounter problems:

1. Check console output for error messages
2. Verify input files (audio/images) are accessible
3. Test with overlays disabled first
4. Compare with original notebook to confirm it's a fix-related issue
5. Document the specific scenario and error messages

---

## Success Criteria

The fixed notebook is working correctly if:

- [x] No black frames at any scene transition
- [x] Overlays are smooth and continuous
- [x] Audio-video sync is perfect
- [x] Total video duration matches sum of scene audio durations
- [x] All original UI and features work correctly
- [x] Progress output is clear and informative
- [x] Rendered video plays correctly in all media players

---

**Testing Date**: _______________  
**Tested By**: _______________  
**Result**: ☐ Pass  ☐ Fail  
**Notes**: _______________________________________________
