# ffmpeg-montage

A shell script that creates video montages from a single input video file. Supports two modes:
- **Grid mode**: Creates a spatial grid layout of video segments (like a contact sheet)
- **Time-based mode**: Concatenates video segments chronologically with audio

## Requirements

- ffmpeg with ffprobe
- Standard POSIX shell

## Usage

```bash
ffmpeg-montage -o offsets -d duration [-g WxH] [--print] input.mp4 output.mp4
```

### Options

- `-o, --offset`: Comma-separated list of start times (required)
- `-d, --duration`: Duration(s) for each segment (required)
- `-g, --grid`: Grid dimensions (WxH) for spatial layout (optional)
- `--print`: Print the ffmpeg command instead of executing it

### Time Format

Times can be specified as:
- Seconds: `30`
- Minutes:seconds: `1:30`
- Hours:minutes:seconds: `1:30:45`

## Grid Mode Examples

Grid mode creates a spatial layout where multiple video segments are displayed simultaneously in a grid pattern.

### Basic 2x2 Grid

Extract 4 segments of 5 seconds each and arrange them in a 2x2 grid:

```bash
ffmpeg-montage -o "0:30,1:45,3:20,5:10" -d "5" -g "2x2" input.mp4 grid_output.mp4
```

This creates:
```
┌─────────┬─────────┐
│ 0:30-35 │ 1:45-50 │
├─────────┼─────────┤
│ 3:20-25 │ 5:10-15 │
└─────────┴─────────┘
```

### Auto-sized Grid

Let the script determine optimal grid size for 6 segments:

```bash
ffmpeg-montage -o "10,25,40,55,70,85" -d "3" -g "3x2" input.mp4 auto_grid.mp4
```

### Creating a Contact Sheet

Sample every 2 minutes for the first 16 minutes:

```bash
ffmpeg-montage -o "0,2:00,4:00,6:00,8:00,10:00,12:00,14:00,16:00" -d "2" -g "3x3" movie.mp4 contact_sheet.mp4
```

### Preview Just the Command

See what ffmpeg command will be generated without executing:

```bash
ffmpeg-montage -o "30,60,90" -d "5" -g "2x2" --print input.mp4 output.mp4
```

## Time-based Mode Examples

Time-based mode concatenates segments chronologically, preserving audio.

### Basic Concatenation

Extract and join three 10-second clips:

```bash
ffmpeg-montage -o "1:30,5:45,12:20" -d "10" input.mp4 highlights.mp4
```

Creates: `[1:30-1:40] → [5:45-5:55] → [12:20-12:30]`

### Variable Duration Segments

Different duration for each segment:

```bash
ffmpeg-montage -o "0:30,2:15,5:45" -d "5,8,12" input.mp4 variable_cuts.mp4
```

Creates: `[0:30-0:35] → [2:15-2:23] → [5:45-5:57]`

### Quick Trailer

Create a 30-second trailer from key moments:

```bash
ffmpeg-montage -o "1:00,15:30,28:45,42:10,55:20" -d "6" movie.mp4 trailer.mp4
```

### Commercial Break Removal

Remove ads by keeping only the content segments:

```bash
ffmpeg-montage -o "0,5:32,11:45,18:20,25:10" -d "5:30,6:10,6:30,6:45,4:20" show.mp4 no_ads.mp4
```

## Practical Use Cases

### Content Review
```bash
# Create a contact sheet for quick content review
ffmpeg-montage -o "0,30,60,90,120,150,180,210,240" -d "3" -g "3x3" raw_footage.mp4 review.mp4
```

### Highlight Reel
```bash
# Combine best moments into a highlight reel
ffmpeg-montage -o "2:30,8:45,15:20,22:10" -d "8,6,10,7" gameplay.mp4 highlights.mp4
```

### Time-lapse Summary
```bash
# Sample every 5 minutes for a 1-hour video
ffmpeg-montage -o "0,5:00,10:00,15:00,20:00,25:00,30:00,35:00,40:00,45:00,50:00,55:00" -d "5" -g "4x3" long_video.mp4 summary.mp4
```

### A/B Testing Cuts
```bash
# Compare different edit points
ffmpeg-montage -o "1:23,1:25,1:27" -d "10" -g "1x3" scene.mp4 timing_test.mp4
```

## Tips

1. **Grid vs Time-based**: Use `-g` for visual comparison, omit for sequential playback
2. **Testing**: Use `--print` to preview commands before processing large files
3. **Performance**: Shorter durations and smaller grids process faster
4. **Quality**: The script uses CRF 18 for high quality output
5. **Audio**: Grid mode discards audio; time-based mode preserves it

## Common Errors

- **"More segments than grid cells"**: Your grid is too small for the number of segments
- **"Number of durations does not match"**: In time-based mode, provide either one duration (applied to all) or one per segment
- **"Must provide --duration"**: The `-d` parameter is always required

## Output Quality

The script uses these ffmpeg settings:
- Video codec: libx264
- CRF: 18 (high quality)
- Preset: fast (good speed/quality balance)
- Audio codec: aac (time-based mode only)
