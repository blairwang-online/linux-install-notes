# How to: run a task with no network access

```zsh
# Step 1 - Use `unshare` to create
# a bash instance with a new network namespace
sudo unshare --net --fork bash

# Step 2 - Go back to your usual account
su - blair

# Step 3 - Verify that there's no network access
ping 8.8.8.8
```
