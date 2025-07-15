# Configure VSC

## VSCode server

- Go to https://github.com/coder/code-server
- The _"Getting started"_ auto-script is pretty good
- To enable HTTPS, edit `~/.config/code-server/config.yaml` and set `cert: true`

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

- Click on the Gear icon (bottom-left corner), then **Keyboard Shortcuts**
- Modify `copyLinesDownAction` and `copyLinesUpAction` as desired, removing any existing shortcuts using the one you want.

Adapted from https://stackoverflow.com/questions/38727047/duplicate-line-in-visual-studio-code
