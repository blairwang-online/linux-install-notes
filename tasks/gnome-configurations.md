# GNOME Configurations

## Sensible settings

In the GNOME settings app:

- About &rarr; Device Name
- Power
    - Automatic Screen Brightness: turn OFF (default: on)
    - Dim Screen: turn OFF (default: on)
    - Screen black: set to NEVER if this is an always-on machine
    - Automatic suspend: set to OFF if this is an always-on machine
- Removeable Media
    - Turn everything off, we don't want this autoplay nonsense!
- Apply Network profiles if needed (e.g., static IP addresses)

### Tweaks

```bash
gsettings set org.gnome.desktop.wm.preferences button-layout ":minimize,maximize,close"
gsettings set org.gnome.desktop.wm.preferences resize-with-right-button true
gsettings set org.gnome.desktop.wm.preferences mouse-button-modifier '<Alt>'

gsettings get org.gnome.desktop.interface font-antialiasing # e.g., 'rgba'
gsettings set org.gnome.desktop.interface font-antialiasing "grayscale"

gsettings get org.gnome.desktop.interface font-hinting
gsettings set org.gnome.desktop.interface font-hinting 'none'

gsettings get org.gnome.desktop.interface font-name # e.g., 'Ubuntu Sans 11'
gsettings set org.gnome.desktop.interface font-name "Adwaita Sans 11"

gsettings get org.gnome.desktop.interface monospace-font-name # e.g., 'Ubuntu Sans Mono 11'
gsettings set org.gnome.desktop.interface monospace-font-name "Adwaita Mono 11"
```

## GNOME Terminal change padding

1. `nano ~/.config/gtk-3.0/gtk.css`
2. Copypaste the following:

```
vte-terminal {
  padding: 10px;
}
```

Adapted from https://askubuntu.com/questions/1443328/gnome-terminal-padding-only-when-fullscreen

