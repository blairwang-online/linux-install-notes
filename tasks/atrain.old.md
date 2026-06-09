# aTrain

## Note

This has been superseded by the Flatpak. See [install-flatpaks.md](install-flatpaks.md).

## Old instructions

First you will need to `sudo apt install ffmpeg python3.11 python3.11-venv`

You can then follow the rest of the "Debian" instructions:

```bash
# Go to the folder you want to setup ATrain in
python3.11 -m venv atrain_venv
source atrain_venv/bin/activate

# Check venv in use
which python

# Update pip
python -m pip install --upgrade pip

# Install aTrain
python -m pip install aTrain@git+https://github.com/BANDAS-Center/aTrain.git --extra-index-url https://download.pytorch.org/whl/cu118

# Next, you need aTrain to download all the necessary models for transcription and speaker detection. This has only to be done once when installing it.
aTrain init

# You are now ready to start aTrain. The following command should open a webrowser window with the user interface.
aTrain start
```

(From https://github.com/JuergenFleiss/aTrain/wiki/Linux-Support but adapted for Ubuntu here)
