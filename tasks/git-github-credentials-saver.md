# Git / GitHub credentials saver

As per https://unix.stackexchange.com/questions/379272/storing-username-and-password-in-git

```bash
git config --global credential.helper store

# to remove
git config --global --unset credential.helper
```

