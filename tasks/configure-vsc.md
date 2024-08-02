# Configure VSC

## Handy extensions

- Catppuccin Pack
- Markdown All in One

## Be careful when changing the editor font

Caution! Certain fonts will change the functionality of the backtick key (`) into a diacritic marker. I had this issue with **Ricty Diminished**.

See also: https://www.reddit.com/r/vscode/comments/t64ega/cant_type_backtick/

## Linux-specific

### Fonts may have subpixel (ClearType-style) rendering by default

To fix this:

```bash
ln -s ~/00blair/gitrepos-github/linux-install-notes/config-files/dot-fonts.conf ~/.fonts.conf
```

### Configure copy line down keybinding

See https://stackoverflow.com/questions/38727047/duplicate-line-in-visual-studio-code
