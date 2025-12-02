
---

> [!warning] **Disclaimer**  
> I do not have formal training in programming, so this may not be the fastest or most efficient way of doing things. It's just the way I prefer to do things.
## What is this?
In our lab, we use PCIbex to run production studies that collect audio data. If you use one of the PCIbex scripts I use, each participant produces a zip file containing multiple audio recordings in `.webm` format..

It is easier to process these files later if you preprocess the zip files first (see [[Using automatic transcription to speed up sentence production data coding]]). Manually performing this processing can be tedious. This article is meant to give you an idea of how to automate the process, with some basic explanations.

---
## Assumptions

> [!tip] **Check before starting**  
> Make sure you meet the following assumptions before continuing:

- You are using macOS (if you are using Windows, you may need to adjust directory paths).
- You have zip files containing audio data collected via PCIbex. Each trial’s audio recording is stored as a separate sound file inside the zip file.

---

## Creating by-subject folders

Suppose you have three zip files in a folder:

`XXX.zip YYY.zip ZZZ.zip`

The goal is to create a subfolder for each subject. Each subfolder should contain:

1. The original zip file    
2. An unzipped `sound/` subfolder containing all audio files

### Step 1: Save the batch unzip script

Save the following code as `batch_unzipper.sh` in your project folder:

```
#!/usr/bin/env bash
set -e
shopt -s nullglob

zip_files=( *.zip )

if [ ${#zip_files[@]} -eq 0 ]; then
    echo "No .zip files detected."
    exit 0
fi

for zip in "${zip_files[@]}"; do
    echo "Processing $zip ..."

    base="${zip%.zip}"

    unzip -q "$zip" -d "$base"

    # If extraction produced loose files instead of a folder
    if [ ! -d "$base" ]; then
        mkdir "$base"
        find . -mindepth 1 -maxdepth 1 ! -name "$base" -exec mv {} "$base/" \;
    fi

    # Determine next S# folder safely
    maxnum=$(ls -d S*/ 2>/dev/null | sed -E 's/^S([0-9]+).*/\1/' | grep -E '^[0-9]+$' | sort -n | tail -1)
    maxnum=${maxnum:-0}
    next=$((maxnum + 1))
    newdir="S$next"

    # Create new S# folder and sound subfolder
    mkdir -p "$newdir/sound"

    # Move contents of extracted folder directly into sound
    mv "$base"/* "$newdir/sound/"
    rmdir "$base"

    # Move the ZIP file into the new S# folder
    mv "$zip" "$newdir/"

    echo " → Created $newdir/ containing: $zip + sound/"
    echo
done

echo "All done."
```

> [!tip] **Save location**  
> Save this script in the same folder where your zip files are located.

---

### Step 2: Run the script

1. Open Terminal and navigate to your project folder:

```
cd PATH_TO_YOUR_PROJECT_FOLDER`
```

2. Run the script:

```
bash batch_unzipper.sh
```

> [!tip] **tip**  
> The script will create a new `S#` folder with the next available number. Existing folders are not overwritten. Make sure any leftover test folders are numbered appropriately to avoid confusion.

---

### Step 3: Check the result

If successful, you should see output like this:

![[example2.png]]

Each `S#` folder now contains:

- The original zip file
- A `sound/` subfolder with all unzipped audio files

> [!tip] **Next step**  
> Check out [[Using automatic transcription to speed up sentence production data coding]] for a way to speed up your transcription using an automatic transcription software (whisper).