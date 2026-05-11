# Python 2 on Ubuntu 24.04 LTS and 26.04 LTS

```bash
sudo apt install distrobox
distrobox create --image ubuntu:22.04
```

If you are asked _Do you want to pull the image now?_ then type `y` and `ENTER`.

```bash
distrobox enter ubuntu-22-04
```

Inside that distrobox, `sudo apt install python2 python2-pip-whl` will get the latest available version of python2

Then:

```bash
curl https://bootstrap.pypa.io/pip/2.7/get-pip.py --output get-pip.py
python2 ./get-pip.py
~/.local/bin/pip install "Pillow<7"
```

As per https://www.centron.de/en/tutorial/install-python-2-on-ubuntu-22-04-with-pip-setup
