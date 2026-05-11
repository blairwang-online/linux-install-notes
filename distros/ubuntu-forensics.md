# Ubuntu forensics environment

## Immediately after installing Ubuntu

Complete the tasks specified at [ubuntu-configure-apt-mirror.md](../tasks/ubuntu-configure-apt-mirror.md)

## Specific software packages

### Autopsy [DONE]

```bash
sudo snap install autopsy
```

### Volatility version 2

First, you need to set up Python version 2 - complete the tasks specified at [python2-on-ubuntu-2404-2604.md](../tasks/python2-on-ubuntu-2404-2604.md).

Once you have done all of that,

https://github.com/volatilityfoundation/volatility/releases/latest

### Viking (analysing GPS data) [DONE, NONROOT VERIFIED]

```bash
sudo apt install viking
```
