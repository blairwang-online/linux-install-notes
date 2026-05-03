# Install Flatpaks

## Regular workstation

```bash
flatpak install flathub com.github.tchx84.Flatseal
flatpak install flathub com.obsproject.Studio
flatpak install flathub com.vscodium.codium
flatpak install flathub io.github.cboxdoerfer.FSearch
flatpak install flathub org.keepassxc.KeePassXC
flatpak install flathub org.pwsafe.pwsafe
flatpak install flathub org.mozilla.Thunderbird
flatpak install flathub com.brave.Browser
flatpak install flathub org.ferdium.Ferdium
flatpak install flathub it.mijorus.smile
```

## Configuration

If you want all your Flatpak apps to use a custom UI font, go to **Flatseal &rarr; All Applications &rarr; Filesystem** and add your custom fonts folder to the **Other Files** section. It needs to be the original location, not a symlink to it.

## Specific apps

### LibreOffice

```bash
flatpak install flathub org.libreoffice.LibreOffice
```

Note: DEB version might be installed by default on Ubuntu.

- Check using `apt list --installed | grep libreoffice`
- Remove specific packages using `sudo apt remove`
- Cleanup (if required) using `sudo apt autoremove`

### Speed of Sound (voice-to-text input)

Discovered thanks to https://www.omgubuntu.co.uk/2026/04/speed-of-sound-linux-voice-typing-app

```bash
flatpak install flathub io.speedofsound.SpeedOfSound
```

If you accidentally stopped the sharing (orange badge in the top-right corner), go to `~/.var/app/io.speedofsound.SpeedOfSound` and delete everything.

### Ungoogled Chromium

In my opinion, it is superior to regular Chromium (e.g., on Ubuntu, regular Chromium will install a snap), and superior to Google Chrome (no forced "AI Mode" in address bar).

```bash
flatpak install flathub io.github.ungoogled_software.ungoogled_chromium
```

Please note that the steps in `chrome://ungoogled-first-run/` will need to be run for each new profile.

## If required (depending on distro)

Some of these might already be installed, natively or otherwise:

```bash
flatpak install flathub app.devsuite.Ptyxis

# On Ubuntu, run `apt-cache policy evince` first.
# If `evince` is available on `main`,
# then use evince (via APT), NOT papers (via Flatpak).
flatpak install flathub org.gnome.Papers

# On Ubuntu, run `apt-cache policy gnome-calendar` first.
# If `gnome-calendar` is available on `main`,
# then use APT to install gnome-calendar, NOT flatpak.
flatpak install flathub org.gnome.Calendar
```

## If running a Podman host

```bash
flatpak install flathub com.github.marhkb.Pods
```

## If running offline AI using Alpaca and Ollama

See [alpaca.md](alpaca.md)

## Kdenlive

See [kdenlive.md](kdenlive.md)

## Extras

```bash
flatpak install flathub app.comaps.comaps
flatpak install flathub com.rafaelmardojai.Blanket
flatpak install flathub cz.ondrejkolin.Barcoder
flatpak install flathub io.github.alainm23.planify
flatpak install flathub io.github.flattool.Warehouse
flatpak install flathub io.gitlab.librewolf-community
flatpak install flathub io.github.quodlibet.QuodLibet
flatpak install flathub md.obsidian.Obsidian
flatpak install flathub net.codelogistics.webapps
flatpak install flathub net.nokyan.Resources # already installed on Ubuntu 26.04 LTS
flatpak install flathub org.cryptomator.Cryptomator
flatpak install flathub org.gimp.GIMP
flatpak install flathub org.gnome.Epiphany
flatpak install flathub org.gnome.World.Secrets
flatpak install flathub org.kde.krita
flatpak install flathub org.kde.okular
flatpak install flathub org.scratchmark.Scratchmark
flatpak install flathub org.pwsafe.pwsafe
flatpak install flathub page.codeberg.censor.Censor
flatpak install flathub page.codeberg.libre_menu_editor.LibreMenuEditor
flatpak install flathub page.tesk.Refine
flatpak install flathub re.sonny.Eloquent
```

Note: you can disable network access, and you should for any apps that don't inherently need it (e.g., Eloquent, Quod Libet).

## Exclusions

- NextCloud client &rarr; not verified (as of April 2026)
