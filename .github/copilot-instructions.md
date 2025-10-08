# Copilot Instructions for tunelrgbdb

## Project Overview
This is a real-time audio visualization tool that creates an animated circular waveform with rainbow colors. The entire application is a single Python script embedded in `README.md`.

## Architecture
- **Single-file application**: All code lives in `README.md` as executable Python
- **Real-time audio processing**: Uses `sounddevice` for capturing microphone input at 44.1kHz
- **Live visualization**: `matplotlib` with `FuncAnimation` renders the waveform as a morphing circle
- **Color animation**: `colorsys` provides HSV-to-RGB conversion for rainbow effect

## Key Components

### Audio Pipeline
- **Buffer size**: 2-second rolling buffer (`fs * 2` samples)
- **Frame parameters**: 2048-sample frames with 25% overlap (`hop_length = frame_length // 4`)
- **Callback pattern**: `audio_callback()` continuously updates `audio_buffer` and `current_audio_frame` globals
- **Stream lifecycle**: Start before animation, cleanup with `stop()` and `close()` after visualization

### Visualization Logic
- **Adaptive normalization**: RMS-based scaling with clipping between `min_rms=0.01` and `max_rms=0.3`
- **Scale factor bounds**: `np.clip(scale_factor, 1.0, 20.0)` prevents extreme distortions
- **Circular mapping**: Polar coordinates with `theta = np.linspace(0, 2π, N)` and `radius = 1 + waveform * scale * 0.5`
- **Color cycle**: Hue advances by `1.0/100` per frame, wrapping with modulo for continuous rainbow

### Display Configuration
- **Black background**: Both figure and axes use `set_facecolor('black')`
- **Square aspect**: `figsize=(7, 7)` with `set_aspect('equal')` and fixed `xlim/ylim=(-2, 2)`
- **Animation timing**: Frame interval calculated as `max(1, int(hop_length * 1000 / fs))` milliseconds

## Development Patterns

### Global State Management
The application uses three globals modified in the audio callback:
```python
audio_buffer = np.zeros(fs * 2)      # Rolling 2-second buffer
current_audio_frame = np.zeros(...)   # Latest frame for visualization
```

### Dependency Stack
Required packages (install with `pip install`):
- `numpy` - Array operations and trigonometry
- `sounddevice` - Audio input streaming
- `matplotlib` - Real-time plotting and animation
- No additional dependencies beyond Python standard library (`colorsys`)

## Running the Code
Since code is in README.md, extract and run it:
```bash
# Extract Python code from README.md (skip first 3 lines)
tail -n +4 README.md > visualizer.py
python visualizer.py
```

The app opens a window and starts capturing audio immediately. Close the window to exit.

## Common Modifications

### Adjusting Sensitivity
Modify `min_rms`, `max_rms`, or the `0.5` multiplier in radius calculation:
```python
radius = 1 + current_audio_frame * scale_factor * 0.5  # Increase 0.5 for more dramatic effects
```

### Changing Colors
Replace the rainbow animation in `update()`:
```python
hue = (frame_num * 1.0 / 100) % 1.0  # Adjust divisor to speed up/slow down color cycling
```

### Buffer/Frame Sizes
All timing depends on `frame_length` and `hop_length` - changing these affects responsiveness vs smoothness tradeoff.

## Project Structure Note
This project intentionally keeps all code in README.md for simplicity and immediate visibility. When suggesting changes, maintain this single-file structure unless explicitly asked to refactor.
