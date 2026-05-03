# Airplay (RAOP)

Tested on Ubuntu 26.04 LTS:

```bash
# gives us pactl
sudo apt install pulseaudio-utils

# confirm not already running
pactl list modules | grep raop

# load RAOP
pactl load-module module-raop-discover

# confirm now running
pactl list modules | grep raop
```

Audio is a bit glitchy with VLC (apt/deb), but worked fine with Quod Libet (Flatpak).
