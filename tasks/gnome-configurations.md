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

## GNOME Terminal change padding

1. `nano ~/.config/gtk-3.0/gtk.css`
2. Copypaste the following:

```
vte-terminal {
  padding: 10px;
}
```

Adapted from https://askubuntu.com/questions/1443328/gnome-terminal-padding-only-when-fullscreen

