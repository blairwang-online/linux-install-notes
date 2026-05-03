# Kdenlive installation and usage

## Install

```bash
flatpak install flathub org.kde.kdenlive
```

## Usage

### Warning! Known issue which causes the entire session to crash

_(Based on Flatpak version on Ubuntu 24.04 LTS)_

If you intuitively use Ctrl+scroll to zoom in and out (as you would on, e.g., FCPX), it might even work occasionally but a lot of the time it seems to cause the entire GNOME session to crash, taking the Kdenlive editing (and any other programs you have open) down with it...

I tried to login to GNOME with Xorg instead of Wayland, but Kdenlive does not run under Xorg. In any case, Xorg is deprecated in future LTS releases of Ubuntu.

So what should you do?

1. Use Ctrl+plus and Ctrl+minus to zoom in and out, NOT the scroll wheel!
2. Do Ctrl+S regularly (saves the project). **Unlike FCPX, Kdenlive does not autosave!**


### Height of track preview in the timeline

This is one nice thing about Kdenlive that FCPX does not seem to do.

You can hover your mouse over the track panels' information (left-hand side of the timeline, where you see **V1** and **A1** in green boxes). Then hover your mouse in the edge between the tracks, and the edge will turn red.

Then you can resize it! It will resize the track **above** the edge, so if you want to resize the bottom-most track then you for the bottom-most edge.


### Video preview on a separate display

If you're working on a 1080p laptop screen, and your video itself is 1080p, and you have also adjusted the height of the track preview as per above, then of course the built-in video preview will become extremely small.

One trick is to plug in an external display and then do the following:

1. In Kdenlive, go to View &rarr; Show Title Bars
2. The video preview now has a label below it, called **Project Monitor**. Double-click the label. Now the Project Monitor will pop out and you can move it over to your second screen.

To dock it back, just double-click the **Project Monitor** label again.


### Boosting the volume

Click on the **Effects** tab, search for **Compressor/Expander**. Then drag-and-drop it onto the audio track. Do this before making any cuts to the clip!

Unfortuantely the **audio waveform preview** in the timeline does not automatically update, so you will need to inspect for peaking/gain using manual efforts, listening to the file, and basically good luck.


### Cutting the clip

**Shift+R** will cut the clip where the cursor is currently placed.


### Jump to start/end of clip

Alt-left and Alt-right


### Exporting

Go to **File --> Render...**

For most projects, use **MP4-H264/AAC**.

You can use a system monitoring widget (e.g. Astra Monitor) to keep an eye on CPU load - when it suddenly drops, then the export is probably done!

### When you're done

First, make sure you have saved all changes (Ctrl+S)!

Next, go to **File --> Archive Project...**

Make sure you enable **Compressed archive** - tar.gz is fine.

That's it!

