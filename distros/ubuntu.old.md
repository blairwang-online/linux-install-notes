# Old notes for Ubuntu, no longer active

## Software (apt/deb)

These are no longer required (and/or not supported in Ubuntu 26.04 LTS):

```bash
sudo apt install cheese dconf-editor ddcutil exiftool gnome-builder nautilus-dropbox obs-studio pasaffe syncthing vlc wl-clipboard

sudo apt install pavucontrol python3.12-venv ttf-mscorefonts-installer
```

I now manage fonts locally using `~/.fonts`, but these should still work if required:

```bash
sudo apt-get install fonts-clear-sans fonts-comic-neue fonts-courier-prime fonts-inconsolata fonts-inter fonts-open-sans fonts-ricty-diminished fonts-roboto

# Do not use because outdated
# sudo apt install fonts-ebgaramond fonts-ebgaramond-extra
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

## Software (manual)

Archived because Flatpaks are available:

- **Brave:** From "App Center" (snaps) (yes it is available as `.deb` as well, but let's keep it simple given how infrequently we use this)
- **Ferdium (e.g., for WhatsApp):** Unfortunately the version from "App Center" (snaps) doesn't work; instead go to https://github.com/ferdium/ferdium-app/releases, download the `.deb` file, and install using `sudo dpkg -i`
- **FSearch:** https://github.com/cboxdoerfer/fsearch?tab=readme-ov-file#download and follow
- **VSCodium:** https://vscodium.com/#install-on-debian-ubuntu-deb-package

Archived because no longer required:

- **Docker**: https://docs.docker.com/engine/install/ubuntu/
- **fastfetch:** https://github.com/fastfetch-cli/fastfetch
    - ARCHIVED: `hyfetch` works fine.
- **Spotify:** https://www.spotify.com/de-en/download/linux/ (use the DEB method, unless you really like snaps)
    - ARCHIVED: I now just run it in the browser.
- **Vivaldi:** https://vivaldi.com/download/
- **Zed:** https://zed.dev/download instructions for Ubuntu PPA Stable

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

