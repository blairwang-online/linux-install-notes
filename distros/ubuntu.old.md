# Old notes for Ubuntu, no longer active

## Software (apt/deb)

```bash
sudo apt-get install -y audacity cheese dconf-editor ddcutil exiftool geany geany-plugins gcolor3 gimp gitg gnome-builder gnome-contacts gnome-maps gnome-sushi gnome-tweaks gnome-weather meld nautilus-dropbox nautilus-nextcloud obs-studio pasaffe synaptic syncthing tree vlc wl-clipboard

# Snaps where the apt/deb version is problematic or unavailable
sudo snap install 0ad
```

## Software (snap)

```bash
# as of Ubuntu 26.04, firmware updater is now a snap, and relies on this one
snap remove gnome-46-2404

# keep these just in case needed, per above
snap remove mesa-2404
snap remove snap-store
snap remove snapd-desktop-integration
```

## Artificial Intelligence

### aTrain

**UPDATE: Use whisper-ctranslate instead**

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

### Reor

**UPDATE: As of May 2026, it seems Reor is no longer maintained.**

Download the latest AppImage at https://github.com/reorproject/reor/releases/tag/v0.2.18

Try to run as normal but if you get an error like "Rather than run without sandboxing I'm aborting now." then try this:

```bash
./Reor_0.2.18.AppImage --no-sandbox
```

Reor seems to automatically pick up the Ollama models in `/usr/share/ollama/.ollama/models/`.

