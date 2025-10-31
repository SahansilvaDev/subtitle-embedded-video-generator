# Subtitle-Embedded Video Generator

English AI Video Subtitle Captioning App using OpenAI Whisper and Gradio

This repository contains a small Gradio app and notebooks that use OpenAI's Whisper to generate English subtitles (VTT) from a video file and optionally burn those subtitles into the video using ffmpeg.

## Features
- Convert video to audio (mp3) and transcribe/translate audio to English using Whisper.
- Produce VTT subtitle files (WebVTT) from Whisper segments.
- Burn subtitles into the video using ffmpeg to create a subtitled output video.
- Minimal Gradio UI (`app.py`) to upload a video and generate a subtitled video.

## Repository files
- `app.py` - Main Gradio application. Loads a Whisper model, converts input video to audio, transcribes/translate, writes a VTT file, and runs ffmpeg to burn subtitles into the output video.
- `Generate_English_Subtitles_Video_with_OpenAI_Whisper.ipynb` - Notebook demonstrating steps to transcribe, create VTT, and preview the subtitled video.
- `12 movie youtubeshorts.mp4` - Example input video (if present in the folder).
- `subtitles.vtt`, `subtitles_fixed.vtt`, `output.vtt`, `12 movie youtubeshorts.vtt` - example/outputs VTT subtitle files created during testing.

## Prerequisites
- Python 3.8+ (3.10/3.11 recommended)
- ffmpeg available on PATH (ffmpeg must be installed and callable from the terminal)
- Internet access for installing Whisper from GitHub (unless Whisper is already installed locally)
- Enough RAM/VRAM depending on the Whisper model (small/medium/large). Larger models require more memory.

Optional: GPU with CUDA and a matching PyTorch build will speed up transcription significantly.

## Recommended packages
- gradio
- git+https://github.com/openai/whisper.git (or install whisper as available)
- torch (install the correct wheel for your platform: see https://pytorch.org/)

Example pip installs (work in PowerShell):

```powershell
pip install gradio
pip install git+https://github.com/openai/whisper.git
# Install torch appropriate for your system; for CPU-only you can try:
pip install torch --index-url https://download.pytorch.org/whl/cpu
```

Note: PyTorch installation is platform-dependent. For GPU support, follow the PyTorch website instructions to get the correct CUDA wheel.

## How to run (quick start)
1. Ensure `ffmpeg` is installed and in PATH. In PowerShell, `ffmpeg -version` should print version information.
2. (Optional) Create and activate a virtual environment.
3. Install dependencies (see Recommended packages above).
4. Edit `app.py` if you want outputs written to a different folder: by default the code writes VTTs to `/content/` (this path suits Colab). On Windows, change `output_dir = '/content/'` in `app.py` to `output_dir = './'` or another folder path.
5. Run the Gradio app:

```powershell
python .\app.py
```

6. Open the Gradio URL shown in the terminal (usually `http://127.0.0.1:7860`) and upload a video file. Click "Generate Subtitle Video". The app will:
   - convert the video to audio using ffmpeg
   - transcribe/translate audio with Whisper
   - write a `.vtt` file
   - run ffmpeg to burn subtitles into a new video file (named like `<audio_path>_subtitled.mp4`)

If the app runs but you get no output video visible in the Gradio preview, check the folder where `app.py` writes the output and open the generated `_subtitled.mp4` file directly.

## Recent fixes applied to `app.py`
I updated `app.py` to make it work more reliably on Windows and other environments. Key changes:

- Robust filename handling: uses `os.path.splitext()` instead of splitting on dots so filenames with extra dots or spaces won't break.
- VTT generation: writes a valid WebVTT file next to the audio file (no Colab-only `/content/` path). The VTT is written in HH:MM:SS.mmm format.
- Safer ffmpeg invocation: calls `ffmpeg` via `subprocess.run([...])` with an argument list so file paths with spaces and backslashes are handled correctly.
- Better error reporting: if ffmpeg fails the error output is printed for debugging.
- The `translate` function now returns an absolute path to the generated subtitled video so Gradio can reliably load it.

If you've already been using the repository, these changes should eliminate the common failures caused by path mismatches and unquoted ffmpeg calls.

## Sample `requirements.txt`
Create a `requirements.txt` file with these (adjust model and torch pins for your platform):

```
gradio
git+https://github.com/openai/whisper.git
torch
```

Notes:
- Replace `torch` with the proper wheel for your OS/CUDA version (see https://pytorch.org/get-started/locally/).

## Quick smoke test (local)
1. Ensure `ffmpeg` is installed and on PATH:

```powershell
ffmpeg -version
```

2. Run a minimal smoke test in PowerShell (adjust input filename):

```powershell
python -c "from app import video2mp3, translate; a=video2mp3('12 movie youtubeshorts.mp4'); print('audio', a); out=translate('12 movie youtubeshorts.mp4'); print('output video:', out)"
```

This will:
- convert the example video to an MP3
- run Whisper to produce a VTT
- call ffmpeg to burn the VTT into a new MP4

If the command raises an error, copy the terminal output (especially any ffmpeg stderr) and paste it here so I can help debug.

## Windows-specific tips
- Change any remaining Colab-style paths in `app.py` (e.g., `output_dir = '/content/'`) to a local path such as `output_dir = './'`.
- If ffmpeg isn't found, add the `ffmpeg\bin` folder to your PATH and restart PowerShell.
- If filenames contain spaces, the updated `app.py` should handle them. If you still see problems, check the ffmpeg error printed by the app.

## Troubleshooting checklist
- ffmpeg installed and callable? -> `ffmpeg -version`
- Correct Python packages installed? -> `pip install -r requirements.txt` (after creating it)
- Whisper model choice too large for RAM/VRAM? Try `small` or `base` models: `model = whisper.load_model('small')`
- If subtitles don't appear when burning with ffmpeg, validate the VTT file format (open in a text editor). You can also try converting VTT to SRT with a small script and then use ffmpeg with the SRT.

## Next steps I can do for you
- Add a `requirements.txt` file with pinned versions and a short `test_smoke.py` that runs the sample flow.
- Make `video2mp3` consistently return absolute paths.
- Add a tiny CLI wrapper to run `translate` and open the produced file.

If you'd like any of those, tell me which and I'll implement them now.

## Manual ffmpeg example (burn subtitles separately)
If you already have a `.vtt` file and want to burn subtitles manually:

```powershell
ffmpeg -i "input.mp4" -vf subtitles="subtitle.vtt" "output_subtitled.mp4"
```

Make sure to wrap paths with spaces in quotes. If your VTT has a different encoding or format issue, inspect it in a text editor and ensure timestamps are valid WebVTT format.

## Notes & Troubleshooting
- ffmpeg not found: install ffmpeg and add it to PATH. On Windows, add the `bin` folder of your ffmpeg installation to the PATH environment variable and restart PowerShell.
- Whisper model memory: if you see out-of-memory errors, use a smaller model (e.g., `small` or `base`) in `app.py`:
  - change `model = whisper.load_model("medium")` to `model = whisper.load_model("small")` or `"base"`.
- Output directory: the current `app.py` uses `'/content/'` which is a Colab-style path. Change it to a local path (e.g., `output_dir = './'`) for local runs.
- Filenames with spaces: `app.py` attempts to wrap paths but ffmpeg command building may need quoting—if you encounter failures, ensure the `ffmpeg` command in `app.py` wraps file paths in double quotes.
- Performance: Use a GPU-enabled PyTorch for faster transcription. On CPU, medium and large models will be slow.

## Example adjustments (Windows friendly)
In `app.py`, change:

```python
output_dir = '/content/'
```

to:

```python
output_dir = './'
```

And ensure ffmpeg commands wrap filenames with quotes when needed:

```python
os.system(f'ffmpeg -i "{input_video}" -vf subtitles="{subtitle}" "{output_video}"')
```

## Security & Privacy
- Transcription occurs locally if Whisper and models are run locally (no audio upload to third-party servers beyond package installs). Be mindful of any sensitive content in audio/video files.

## License
MIT

---

If you'd like, I can:
- update `app.py` to use a Windows-friendly `output_dir` and ensure ffmpeg calls are properly quoted, or
- add a `requirements.txt` with pinned dependencies and a small test script.

