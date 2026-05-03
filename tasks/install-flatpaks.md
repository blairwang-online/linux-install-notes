# Install Flatpaks

## Regular workstation

```bash
flatpak install flathub com.github.tchx84.Flatseal
flatpak install flathub com.obsproject.Studio
flatpak install flathub com.vscodium.codium
flatpak install flathub io.github.cboxdoerfer.FSearch
flatpak install flathub org.keepassxc.KeePassXC
flatpak install flathub org.mozilla.Thunderbird
flatpak install flathub com.brave.Browser
flatpak install flathub org.ferdium.Ferdium
flatpak install flathub it.mijorus.smile
flatpak install flathub io.github.ungoogled_software.ungoogled_chromium
```

## LibreOffice

```bash
flatpak install flathub org.libreoffice.LibreOffice
```

Note: DEB version might be installed by default on Ubuntu.

- Check using `apt list --installed | grep libreoffice`
- Remove specific packages using `sudo apt remove`
- Cleanup (if required) using `sudo apt autoremove`

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

```bash
flatpak install flathub com.jeffser.Alpaca
```

As of April 2026, there is no need to download Ollama separately. Just then go through the in-app onboarding process for Ollama; when you get to the step called **Instances**, select **Create Ollama Instance**.

## Extras

```bash
flatpak install flathub cz.ondrejkolin.Barcoder
flatpak install flathub io.github.alainm23.planify
flatpak install flathub io.github.flattool.Warehouse
flatpak install flathub io.gitlab.librewolf-community
flatpak install flathub io.github.quodlibet.QuodLibet
flatpak install flathub net.codelogistics.webapps
flatpak install flathub org.cryptomator.Cryptomator
flatpak install flathub net.nokyan.Resources
flatpak install flathub org.gimp.GIMP
flatpak install flathub org.gnome.Epiphany
flatpak install flathub org.gnome.World.Secrets
flatpak install flathub org.kde.krita
flatpak install flathub org.scratchmark.Scratchmark
flatpak install flathub org.pwsafe.pwsafe
flatpak install flathub page.codeberg.libre_menu_editor.LibreMenuEditor
flatpak install flathub page.tesk.Refine
```

## Exclusions

- NextCloud client &rarr; not verified (as of April 2026)
