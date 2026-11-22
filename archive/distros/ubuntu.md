# Archived Ubuntu notes

Notes I no longer use, kept for reference. The current notes are in [ubuntu.md](../../distros/ubuntu.md).

## Software (apt/deb)

These are no longer required (and/or not supported in Ubuntu 26.04 LTS):

```bash
sudo apt install cheese dconf-editor ddcutil exiftool gnome-builder gnome-sushi nautilus-dropbox obs-studio pasaffe syncthing vlc wl-clipboard

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
- **VSCodium:** now installed via Flatpak only; see [install-flatpaks.md](../../tasks/install-flatpaks.md)

Archived because no longer required:

- **Docker**: https://docs.docker.com/engine/install/ubuntu/
- **fastfetch:** https://github.com/fastfetch-cli/fastfetch
    - ARCHIVED: `hyfetch` works fine.
- **Spotify:** https://www.spotify.com/de-en/download/linux/ (use the DEB method, unless you really like snaps)
    - ARCHIVED: I now just run it in the browser.
- **Vivaldi:** https://vivaldi.com/download/
- **Zed:** https://zed.dev/download instructions for Ubuntu PPA Stable

## Artificial Intelligence

### Reor

**UPDATE: As of May 2026, it seems Reor is no longer maintained.**

Download the latest AppImage at https://github.com/reorproject/reor/releases/tag/v0.2.18

Try to run as normal but if you get an error like "Rather than run without sandboxing I'm aborting now." then try this:

```bash
./Reor_0.2.18.AppImage --no-sandbox
```

Reor seems to automatically pick up the Ollama models in `/usr/share/ollama/.ollama/models/`.

### Ollama

**NOTE:** The use of Ollama in this way is superseded by Alpaca (via Flatpak). See [install-flatpaks.md](../../tasks/install-flatpaks.md).

**Installing**

Use the script method at https://ollama.com/download/linux

After installation, check localhost:11434 to see if ollama is running, you can also run `sudo systemctl status ollama` if you're curious. But then you should probably `sudo systemctl disable ollama` and `sudo systemctl stop ollama` so that it only runs when you need it. Unless this is a server.

To install a model, go to https://ollama.com/library and browse for a model you like, e.g., `llama3.1`, then you can run in the terminal:

```bash
ollama pull llama3.1
```

For a nice web UI:

```bash
python3.11 -m pipx install open-webui
open-webui serve
```

**Starting up**

```bash
sudo systemctl start ollama
open-webui serve
```

## Superseded configurations

### LibreOffice spellcheck

**NOTE:** This configuration is no longer required if you install LibreOffice via Flatpak. See [install-flatpaks.md](../../tasks/install-flatpaks.md).

1. Close all LibreOffice running programs (Writer, Calc, etc.)
2. `sudo apt install hunspell-en-au hunspell-en-ca hunspell-en-gb`
3. Open LibreOffice Writer, go to **Tools &rarr; Options &rarr; Languages and Locales &rarr; General**. Set **Locale setting** to, e.g., `English (Australia)`.
4. Now go to **Tools &rarr; Options &rarr; Languages and Locales &rarr; Writing Aids**. In the "Available Language Modules" box, click on **Hunspell SpellChecker**, then click **Edit**. Select the relevant language and confirm that **Hunspell SpellChecker** is available and checked (ticked).

### Mozilla PPA for Firefox ESR and Thunderbird

**NOTE:** The use of Firefox is superseded by LibreWolf (via Flatpak). The use of Thunderbird DEB is superseded by Thunderbird Flatpak. See [install-flatpaks.md](../../tasks/install-flatpaks.md).

- Go to snap store and remove Firefox
- Add https://launchpad.net/~mozillateam/+archive/ubuntu/ppa
- Install Firefox ESR https://ubuntuhandbook.org/index.php/2023/09/install-firefox-esr-115-ppa/
- Install Thunderbird: https://ubuntuhandbook.org/index.php/2024/03/install-thunderbird-deb-ubuntu-2404/

### Waterfox

**NOTE:** The use of Waterfox is superseded by LibreWolf (via Flatpak). See [install-flatpaks.md](../../tasks/install-flatpaks.md).

As at time of writing, only tar.gz available for Linux.

1. Go to https://www.waterfox.net/download/ and download latest release for 🐧 **Linux**.
2. Extract it to, e.g., `~/00blair/bin`
3. For version-agnostic symlink: `ln -s ~/00blair/bin/waterfox-G6.0.17/waterfox ~/00blair/bin/waterfox`
4. Set up a launcher shortcut:

```bash
sudo nano /usr/share/applications/waterfox.desktop
```

Copy the contents of [waterfox.desktop](../config-files/waterfox.desktop)

## Hardware Support

### Logitech G203 Lightsync with OpenRGB

**NOTE:** Superseded by Piper, see [ubuntu.md](../../distros/ubuntu.md#logitech-g203-lightsync). OpenRGB also works, but Piper is better.

This mouse works very nicely with OpenRGB.

1. Run `sudo apt install libfuse-dev` so that you can run AppImages.
	- WARNING: Do NOT install `fuse`, it will [break your system](https://askubuntu.com/questions/1409496/how-to-safely-install-fuse-on-ubuntu-22-04).
2. Go to https://openrgb.org/releases.html and get the **AppImage**.
3. Run `chmod +x` on the AppImage.
4. Run the AppImage. You may be prompted to install udev rules over at https://openrgb.org/udev - if so then go ahead.
