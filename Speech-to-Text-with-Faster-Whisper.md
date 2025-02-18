# Run and test faster-whisper (in terminal)

Prerequisites:
- Python 3.11

## 1. Install Dependencies

```bash
pip install faster-whisper sounddevice soundfile numpy
```

## 2. Create the Test Script

```python
import os
import numpy as np
import sounddevice as sd
import soundfile as sf
from faster_whisper import WhisperModel

def record_audio(duration, filename, samplerate=16000):
    """Record audio for a given duration (in seconds) and save it as a WAV file."""
    print(f"Recording audio for {duration} seconds...")
    # Record audio with 1 channel (mono) at the specified samplerate
    recording = sd.rec(int(duration * samplerate), samplerate=samplerate, channels=1, dtype='int16')
    sd.wait()  # Wait until recording is finished
    sf.write(filename, recording, samplerate)
    print(f"Audio recorded and saved to {filename}")

def transcribe_audio(filename, model_size="base", device="cpu", beam_size=5):
    """Transcribe the audio file using faster-whisper."""
    print("Loading faster-whisper model...")
    model = WhisperModel(model_size, device=device, compute_type="float32")
    
    print("Transcribing audio...")
    segments, info = model.transcribe(filename, beam_size=beam_size)
    
    print("\nTranscription:")
    for segment in segments:
        print(f"[{segment.start:.2f}s -> {segment.end:.2f}s] {segment.text}")

if __name__ == "__main__":
    audio_filename = "test_audio.wav"
    duration = 10  # Record 10 seconds of audio
    
    # Step 1: Record your voice
    record_audio(duration, audio_filename)
    
    # Step 2: Transcribe the recorded audio using faster-whisper
    # You can choose among "tiny", "base", "small", "medium", or "large"
    # depending on your needs. Note that larger models are more accurate but slower.
    transcribe_audio(audio_filename, model_size="medium", device="cpu")
    
    # Optional: Clean up the recorded audio file
    if os.path.exists(audio_filename):
        os.remove(audio_filename)
```

## 3. Run the Script

```bash
python3 test_faster_whisper.py
Recording audio for 10 seconds...
Audio recorded and saved to test_audio.wav
Loading faster-whisper model...
Transcribing audio...

Transcription:
[0.00s -> 7.76s]  this is testing, please rejoin the meeting. now please go to the home page
[7.76s -> 11.12s]  select number 8
```
