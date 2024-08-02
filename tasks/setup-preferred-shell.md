# Setup preferred shell

## Prerequisites

AlmaLinux, Fedora:

```bash
sudo dnf install -y curl git zsh
```

Debian, Ubuntu:

```bash
sudo apt-get install -y curl git zsh
```

## Switch to zsh

```bash
touch ~/.zshrc
chsh -s /bin/zsh
```

Then logout and log back in, then setup oh-my-zsh:

```zsh
sh -c "$(curl -fsSL https://raw.github.com/ohmyzsh/ohmyzsh/master/tools/install.sh)"
```

## Pure

As per https://github.com/sindresorhus/pure

```zsh
mkdir -p "$HOME/.zsh"
git clone https://github.com/sindresorhus/pure.git "$HOME/.zsh/pure"
```

Then edit `~/.zshrc` to include as follows:

```
# https://github.com/sindresorhus/pure
fpath+=($HOME/.zsh/pure)
autoload -U promptinit; promptinit
prompt pure
```
