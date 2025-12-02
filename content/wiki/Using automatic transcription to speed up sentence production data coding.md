## Why this tutorial? Who is this for?
One of the bottlenecks in studying sentence-level production is transcription. This tutorial aims to make this process as painless, fast, and accurate as possible using automatic transcription followed by human checking.

This tutorial is intended to be useful for people who study sentence-level production (or researchers who need to transcribe a bunch of small audio files containing speech).
## Assumptions
This tutorial is written with the following assumptions:
- Audio recording of each trial is stored separately as individual sound files (here we assume `.mp3` format, though other file formats should work with minimal tweaks).
- You are using MacOS (if you are using another OS, you may need to modify the Bash scripts accordingly) and have Python installed.
- You have already installed Homebrew ([https://brew.sh/](https://brew.sh/)).
- Your sound files are organized in a specific way. To comply with this structure, create a folder for your project. Inside that folder, create a subfolder named `SX`, where `X` is the subject number (e.g., `S1`, `S2`, `S3`, etc.). All sound files associated with a subject must be inside the corresponding `SX` subfolder. For example (each `SX` folder should contain all sound files for that subject):
![[images/example1.png]]
## Installing Whisper
[Whisper](https://github.com/openai/whisper) is an automatic speech recognition software developed by OpenAI. Its outputs are not perfect and must be human-checked for scientific purposes. However, using Whisper and then reviewing the output is far faster than transcribing everything manually.

```
pip install -U openai-whisper
```

Whisper requires ffmpeg, so you need to install it if you haven't done so.

```
brew install ffmpeg
```

## Running Whisper on all sound files in a specified folder (single subject data)
Here's a Python script (assuming your sound files are `.mp3`; you can change `.mp3` to another format, like `.wav`, by adjusting the relevant part of the script). Save it as `folder_transcript.py` (without quotes):

```
import os  
import csv  
import whisper  
import re  
import sys  
  
# define arguments  
INPUT_FOLDER = sys.argv[1]  
OUTPUT_CSV = sys.argv[2]  
MODEL_NAME = sys.argv[3]  
  
# natural sort  
def natural_key(string):  
    return [int(s) if s.isdigit() else s.lower() for s in re.split(r'(\d+)', string)]  
  
def main():  
    # Load Whisper model  
    print(f"Loading Whisper model: {MODEL_NAME}...")  
    model = whisper.load_model(MODEL_NAME)  
  
    # Get list of .mp3 files (change to, e.g., .WAV if you are dealing with .WAV files)
    files = [f for f in os.listdir(INPUT_FOLDER) if f.lower().endswith(".mp3")]  
  
    # Natural sort  
    files.sort(key=natural_key)  
  
    # Check if there was mp3 files  
    if not files:  
        print("No relevant files found in the folder.")  
        return  
  
    print(f"Found {len(files)} files.")  
    print("Starting transcription...\n")  
  
    # Open CSV for writing  
    with open(OUTPUT_CSV, "w", newline='', encoding="utf-8") as csvfile:  
        writer = csv.writer(csvfile)  
        writer.writerow(["filename", "transcript"])  # header row  
  
        for i, filename in enumerate(files, 1):  
            full_path = os.path.join(INPUT_FOLDER, filename)  
            print(f"[{i}/{len(files)}] Transcribing: {filename}")  
  
            try:  
                result = model.transcribe(full_path)  
                transcript = result["text"].strip()  
            except Exception as e:  
                transcript = f"ERROR: {e}"  
                print(f"   !!! Error transcribing {filename}: {e}")  
  
            # Write CSV row  
            writer.writerow([filename, transcript])  
  
    print("\nDone! Saved CSV file:", OUTPUT_CSV)  
  
if __name__ == "__main__":  
    main()

```

You need to pass three arguments to the script:
- `INPUT_FOLDER`: path to your folder containing the audio files
- `OUTPUT_CSV`: location where the CSV file containing transcripts will be stored    
- `MODEL_NAME`: Whisper model to use (`tiny`, `base`, `small`, `medium`, `large`, `turbo`). In my experience, `small` is sufficient since the transcription will be human-checked.

Example usage in Terminal:
```
# define your working directory - this folder must contain the transcribe_folder.py

WD="YOUR_DIRECTORY" 
SUBJ="S1" # assuming you are processing sound files for S1 (the first subject)

AUDIO_DIR="$WD$SUBJ"  
TRANSCRIPTION_LOC="$WD$SUBJ/transcription.csv"
MODEL="small"

python transcribe_folder.py "$AUDIO_DIR" "$TRANSCRIPTION_LOC" "$MODEL"
```

After running this script, the `S1` folder will contain a `transcription.csv` file with two columns: `filename` (which sound file corresponds to each transcription) and `transcript` (the transcription). You should listen to each audio file to verify the accuracy and correct any errors.
## Processing data from multiple subjects at once

Running the above script manually for each subject can be tedious. You can create a `for` loop to process multiple subjects in a batch:

```
# List subjects here:  
SUBJECTS="S1 S2 S3 S4 S5 S6"  
WD="XXXX" # Replace XXX with the path to your project folder
echo "Subjects: $SUBJECTS" # prints the subject list
MODEL="small" # Model size (Whisper)

for SUBJ in $SUBJECTS; do
    echo "===== Processing $SUBJ ====="
    AUDIO_DIR="$WD/$SUBJ/sound/converted"
    TRANSCRIPTION_LOC="$WD/$SUBJ/transcription.csv"
    python folder_transcript.py "$AUDIO_DIR" "$TRANSCRIPTION_LOC" "$MODEL"
    echo "===== Finished $SUBJ ====="
done

```

Paste this code in Terminal. Make sure to define the `SUBJECTS` variable appropriately (e.g., if you want to batch-transcribe `S10`–`S15`, set `SUBJECTS="S10 S11 S12 S13 S14 S15"`). If you have many subjects, consider writing a script to automatically extract all `SX` folder names in a directory.





