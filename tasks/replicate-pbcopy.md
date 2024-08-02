# Replicate pbcopy

Adapted from https://ostechnix.com/how-to-use-pbcopy-and-pbpaste-commands-on-linux/

1. Install components
    - AlmaLinux / Fedora: `sudo dnf install -y xclip xsel`
    - Debian / Ubuntu: `sudo apt-get install -y xclip xsel`
2. `code ~/.zshrc` (unless you are using bash, in which case it's `~/.bashrc`)
3. Aliases...
	```
	alias pbcopy='xclip -selection clipboard'
	alias pbpaste='xclip -selection clipboard -o'
	```
4. Now you can use `pbcopy`! For example, grabbing the current directory onto the clipboard: `pwd | tr -d '\n' | pbcopy`

