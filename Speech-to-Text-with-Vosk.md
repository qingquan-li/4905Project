# Run and test Vosk (in terminal)

## 1. Install Dependencies

```bash
pip install vosk sounddevice
```

## 2. Download a Vosk Model

- Go to [Vosk’s models page](https://alphacephei.com/vosk/models) and download an English model [vosk-model-en-us-0.22](https://alphacephei.com/vosk/models/vosk-model-en-us-0.22.zip) (1.8G).
- Unpack the downloaded model, rename it as `model`, and put it in the same directory where you’ll place your Python script.

## 3. Create the Test Script

Create a file (e.g., `test_vosk.py`) with the following code:

```python
import queue
import sys
import json
import sounddevice as sd
from vosk import Model, KaldiRecognizer

# Create a queue to hold audio data
q = queue.Queue()

# Callback function to capture audio and put it in the queue
def audio_callback(indata, frames, time, status):
    if status:
        print(status, file=sys.stderr)
    q.put(bytes(indata))

# Initialize the Vosk model (ensure the 'model' folder exists in your directory)
model = Model("model")
# Initialize recognizer with sample rate 16000 Hz
recognizer = KaldiRecognizer(model, 16000)

print("Please say a sentence. The system will record your voice and transcribe it...")

# Open a raw input stream from the default microphone.
with sd.RawInputStream(samplerate=16000, blocksize=8000, dtype='int16', channels=1, callback=audio_callback):
    print("Recording... Press Ctrl+C if you need to stop.")
    try:
        while True:
            data = q.get()
            if recognizer.AcceptWaveform(data):
                # When a final result is available
                result_json = recognizer.Result()
                result = json.loads(result_json)
                print("Final Transcription:", result.get("text", ""))
                break
            else:
                # Optionally, print partial results
                partial_result = recognizer.PartialResult()
                sys.stdout.write("\r" + partial_result)
                sys.stdout.flush()
    except KeyboardInterrupt:
        print("\nRecording interrupted.")

print("\nTranscription complete.")
```

## 4. Run the Script

```bash
python3 test_vosk.py
LOG (VoskAPI:ReadDataFiles():model.cc:213) Decoding params beam=13 max-active=7000 lattice-beam=6
LOG (VoskAPI:ReadDataFiles():model.cc:216) Silence phones 1:2:3:4:5:11:12:13:14:15
LOG (VoskAPI:RemoveOrphanNodes():nnet-nnet.cc:948) Removed 0 orphan nodes.
LOG (VoskAPI:RemoveOrphanComponents():nnet-nnet.cc:847) Removing 0 orphan components.
LOG (VoskAPI:ReadDataFiles():model.cc:248) Loading i-vector extractor from model/ivector/final.ie
LOG (VoskAPI:ComputeDerivedVars():ivector-extractor.cc:183) Computing derived variables for iVector extractor
LOG (VoskAPI:ComputeDerivedVars():ivector-extractor.cc:204) Done.
LOG (VoskAPI:ReadDataFiles():model.cc:279) Loading HCLG from model/graph/HCLG.fst
LOG (VoskAPI:ReadDataFiles():model.cc:294) Loading words from model/graph/words.txt
LOG (VoskAPI:ReadDataFiles():model.cc:303) Loading winfo model/graph/phones/word_boundary.int
LOG (VoskAPI:ReadDataFiles():model.cc:310) Loading subtract G.fst model from model/rescore/G.fst
LOG (VoskAPI:ReadDataFiles():model.cc:312) Loading CARPA model from model/rescore/G.carpa
LOG (VoskAPI:ReadDataFiles():model.cc:318) Loading RNNLM model from model/rnnlm/final.raw
Please say a sentence. The system will record your voice and transcribe it...
Recording... Press Ctrl+C if you need to stop.
{
  "partial" : ""
{
  "partial" : ""
{
  "partial" : "the"
{
  "partial" : "the"
{
  "partial" : "the"
{
  "partial" : "the a"
{
  "partial" : "this is testing"
{
  "partial" : "this is testing"
{
  "partial" : "the a test high priest was"
{
  "partial" : "the a test high priest was dried up"
{
  "partial" : "the a test high priest was dried up meeting"
{
  "partial" : "the a test high priest was dried up meeting"
{
  "partial" : "the a test high priest was dried up meeting now"
{
  "partial" : "the a test high priest was dried up meeting now please go to"
{
  "partial" : "the a test high priest was dried up meeting now please go to the"
{
  "partial" : "the a test high priest was dried up meeting now please go to the home page"
{
  "partial" : "the a test high priest was dried up meeting now please go to the home page"
{
  "partial" : "the a test high priest was dried up meeting now please go to the home page slash number"
{
  "partial" : "the a test high priest was dried up meeting now please go to the home page slash number eight"
{
  "partial" : "the a test high priest was dried up meeting now please go to the home page slash number eight"
}Final Transcription: this is testing praised was dried up meeting now press go to the home page slash number eight

Transcription complete.
```

---

## Comparing Faster-Whisper and Vosk

- Faster-Whisper:
   - Pros: Higher accuracy in challenging conditions and can be very fast on a GPU.
   - Cons: Typically heavier in terms of resource usage and might be overkill for simple command recognition. Slightly higher latency.

- Vosk:
   - Pros: Lightweight, runs entirely offline on CPU, offers low latency.
   - Cons: May not achieve high accuracy with challenging audio, but is usually "good enough" for very clear and standard English transcriptions.
