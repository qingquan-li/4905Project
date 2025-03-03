Prerequisites:

- Python 3.11
- Windows (I use Windows 11)
- Nvidia GPU (I use NVIDIA GeForce RTX 3060 Laptop GPU)

# 1. Install Dependencies

```
pip install faster-whisper sounddevice soundfile numpy
```

### Optional: Use the PyTorch backend (recommended)

You can use the PyTorch backend so that you don't have to manually download and configure cuDNN. (Note: The NVIDIA CUDA® Deep Neural Network library (cuDNN) is a GPU-accelerated library of primitives for deep neural networks). PyTorch’s pre-built binaries come with CUDA and the matching cuDNN libraries already bundled and configured.

````
pip install torch torchvision torchaudio --extra-index-url https://download.pytorch.org/whl/cu118
````

Note: If you want the GPU-enabled version of PyTorch (i.e. one built with CUDA support), it’s recommended to include the extra index URL.

To do this with faster‑whisper, you can tell it to use the torch backend instead of its default CTranslate2 backend. One common way to do this is to set the environment variable FASTER_WHISPER_BACKEND to "torch" before loading the model:

```python
import os
os.environ["FASTER_WHISPER_BACKEND"] = "torch"
```

# 2. Create the Test Script

```python
# test_faster_whisper.py

import os
# Set faster-whisper to use the torch backend
os.environ["FASTER_WHISPER_BACKEND"] = "torch"

import numpy as np
import sounddevice as sd
import soundfile as sf
from faster_whisper import WhisperModel

def record_audio(duration, filename, samplerate=16000):
    """
    Record audio for a given duration (in seconds) and save it as a WAV file.
    """
    print(f"Recording audio for {duration} seconds...")
    # Record mono audio at the specified samplerate
    recording = sd.rec(int(duration * samplerate), samplerate=samplerate, channels=1, dtype='int16')
    sd.wait()  # Wait until recording is finished
    sf.write(filename, recording, samplerate)
    print(f"Audio recorded and saved to {filename}")

def transcribe_audio(filename, model_size="medium", device="cuda", beam_size=5):
    """
    Transcribe the audio file using faster-whisper with the torch backend.
    """
    print("Loading faster-whisper model with torch backend...")
    model = WhisperModel(model_size, device=device, compute_type="float32")
    
    print("Transcribing audio...")
    segments, info = model.transcribe(filename, beam_size=beam_size)
    
    print("\nTranscription:")
    for segment in segments:
        print(f"[{segment.start:.2f}s -> {segment.end:.2f}s] {segment.text}")

if __name__ == "__main__":
    audio_filename = "test_audio.wav"
    duration = 10  # Record for 10 seconds

    # Step 1: Record audio
    record_audio(duration, audio_filename)
    
    # Step 2: Transcribe using faster-whisper with CUDA via the torch backend
    transcribe_audio(audio_filename, model_size="medium", device="cuda")
    
    # Cleanup: Remove the temporary audio file
    # if os.path.exists(audio_filename):
    #     os.remove(audio_filename)
```

# 3. Run the Script
```
PS C:\Users\jake\code\whisper_demo> python test_faster_whisper.py
Recording audio for 10 seconds...
Audio recorded and saved to test_audio.wav
Loading faster-whisper model with torch backend...
config.json: 100%|████████████████████████████████████████████████████████████████████████| 2.26k/2.26k [00:00<00:00, 273kB/s]
C:\Users\jake\code\whisper_demo\.venv\Lib\site-packages\huggingface_hub\file_download.py:142: UserWarning: `huggingface_hub` cache-system uses symlinks by default to efficiently store duplicated files but your machine does not support them in C:\Users\jake\.cache\huggingface\hub\models--Systran--faster-whisper-medium. Caching files will still work but in a degraded version that might require more space on your disk. This warning can be disabled by setting the `HF_HUB_DISABLE_SYMLINKS_WARNING` environment variable. For more details, see https://huggingface.co/docs/huggingface_hub/how-to-cache#limitations.
To support symlinks on Windows, you either need to activate Developer Mode or to run Python as an administrator. In order to activate developer mode, see this article: https://docs.microsoft.com/en-us/windows/apps/get-started/enable-your-device-for-development
  warnings.warn(message)
vocabulary.txt: 100%|██████████████████████████████████████████████████████████████████████| 460k/460k [00:00<00:00, 3.79MB/s]
tokenizer.json: 100%|████████████████████████████████████████████████████████████████████| 2.20M/2.20M [00:00<00:00, 11.3MB/s]
model.bin: 100%|█████████████████████████████████████████████████████████████████████████| 1.53G/1.53G [01:09<00:00, 21.8MB/s]
Transcribing audio...████████████████████████████████████████████████████████████████████| 2.20M/2.20M [00:00<00:00, 11.4MB/s]

Transcription:
[0.00s -> 10.00s]  This is testing, please rejoin the meeting. Now please go to the home page, select number 8.
```


----

# Another Example

## Install Dependencies

```
pip install faster-whisper torch --extra-index-url https://download.pytorch.org/whl/cu118
```

## Create the Test Script

```python
#  test_whisper.py

import argparse
import torch
from faster_whisper import WhisperModel

def main():
    parser = argparse.ArgumentParser(
        description="Test faster_whisper on Windows with Nvidia GPU acceleration using the medium model."
    )
    parser.add_argument(
        "--model",
        type=str,
        default="medium",
        help="Name of the Whisper model to use (default: medium)"
    )
    parser.add_argument(
        "audio_file",
        type=str,
        help="Path to the audio file you want to transcribe."
    )
    args = parser.parse_args()

    # Check if CUDA (Nvidia GPU) is available.
    if torch.cuda.is_available():
        device = "cuda"
        compute_type = "float16"  # Use half precision for faster computation on GPU.
        print("CUDA detected: Using Nvidia GPU acceleration.")
    else:
        device = "cpu"
        compute_type = "int8"  # Fallback compute type for CPU.
        print("CUDA not available: Using CPU (this may be slower).")

    # Load the faster_whisper model on the selected device.
    model = WhisperModel(args.model, device=device, compute_type=compute_type)

    # Transcribe the provided audio file.
    print("Transcribing audio...")
    segments, info = model.transcribe(args.audio_file, beam_size=5)

    print(f"Detected language: {info.language}")
    for segment in segments:
        print(f"[{segment.start:.2f}s - {segment.end:.2f}s] {segment.text}")

if __name__ == "__main__":
    main()
```

## Run the Script

```
PS C:\Users\jake\code\whisper_demo> python test_whisper.py test_audio.wav
CUDA detected: Using Nvidia GPU acceleration.
Transcribing audio...
Detected language: en
[0.00s - 10.00s]  This is testing, please rejoin the meeting. Now please go to the home page, select number 8.
```
