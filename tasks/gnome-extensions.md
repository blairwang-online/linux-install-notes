# GNOME Extensions

## Must have

- Astra Monitor by `AstraExt` [1]
- Power Tracker by `pijuli`
- Desktop Icons NG (DING) by `rastersoft`

## Should have

- Dash to Dock by `michele_g`
- ArcMenu by `andrew.zaech` (note: `sudo pacman -S gnome-menus` to get it to work)

## Could have

- Brightness control using ddcutil by `themightydeity`
- Blur my Shell by `aunetx`
- You may also consider installing some from https://www.debugpoint.com/gnome-40-extensions/

## Won't have

- System Monitor by `fmuellner` (alright, pleasant enough; similar to Astra Monitor)
- System Monitor by `elvetemedve` (I found that this one causes the GNOME session to crash)
- Panel Corners by `aunetx` (no longer supported)
- Wiggle by `mechtifs` (no longer supported)

&nbsp;  

Notes:

1. If this doesn't work, try:

    ```
    # As per https://github.com/elvetemedve/gnome-shell-extension-system-monitor/wiki/Installation#dependencies
    sudo apt install gir1.2-gtop-2.0 libgtop2-dev

    # You should reboot after this point (to ensure libgtop installs properly)
    ```
