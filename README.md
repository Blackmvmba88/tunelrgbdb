# tunelrgbdb


import numpy as np
import sounddevice as sd
import matplotlib.pyplot as plt
from matplotlib.animation import FuncAnimation
import colorsys

fs = 44100
frame_length = 2048
hop_length = frame_length // 4
audio_buffer = np.zeros(fs * 2)

fig, ax = plt.subplots(figsize=(7, 7))
fig.patch.set_facecolor('black')   # Fondo de la ventana
ax.set_facecolor('black')          # Fondo del área de gráficos
ax.set_aspect('equal')
ax.axis('off')

current_audio_frame = np.zeros(frame_length)

line_circle, = ax.plot([], [], linewidth=2)

def audio_callback(indata, frames, time, status):
    global audio_buffer, current_audio_frame
    audio_buffer = np.roll(audio_buffer, -len(indata[:, 0]))
    audio_buffer[-len(indata[:, 0]):] = indata[:, 0]
    current_audio_frame = audio_buffer[-frame_length:]

stream = sd.InputStream(channels=1, samplerate=fs, callback=audio_callback, blocksize=hop_length)
stream.start()

def update(frame_num):
    # Normalización adaptativa
    rms = np.sqrt(np.mean(current_audio_frame ** 2))
    min_rms = 0.01
    max_rms = 0.3
    if rms < min_rms:
        scale_factor = 1.0 / min_rms
    elif rms > max_rms:
        scale_factor = 1.0 / max_rms
    else:
        scale_factor = 1.0 / rms
    scale_factor = np.clip(scale_factor, 1.0, 20.0)

    # Onda circular
    N = len(current_audio_frame)
    theta = np.linspace(0, 2 * np.pi, N)
    radius = 1 + current_audio_frame * scale_factor * 0.5
    x = radius * np.cos(theta)
    y = radius * np.sin(theta)

    # Color rainbow animado
    hue = (frame_num * 1.0 / 100) % 1.0
    rgb = colorsys.hsv_to_rgb(hue, 1, 1)
    line_circle.set_data(x, y)
    line_circle.set_color(rgb)

    ax.set_xlim(-2, 2)
    ax.set_ylim(-2, 2)
    return line_circle,

ani = FuncAnimation(fig, update, interval=max(1, int(hop_length * 1000 / fs)), blit=True)

plt.tight_layout()
plt.show()

stream.stop()
stream.close()
