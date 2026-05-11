# Ubuntu forensics environment

## Getting Ubuntu

Obtain the installer from https://mirror.aarnet.edu.au/pub/ubuntu/releases/26.04/

`ubuntu-26.04-desktop-amd64.iso`

## Immediately after installing Ubuntu

Complete the tasks specified at [ubuntu-configure-apt-mirror.md](../tasks/ubuntu-configure-apt-mirror.md)

## Specific software packages

### Autopsy [DONE, NONROOT VERIFIED]

```bash
sudo snap install autopsy
```

### Volatility version 2 [DONE]

First, you need to set up Python version 2 - complete the tasks specified at [python2-on-ubuntu-2404-2604.md](../tasks/python2-on-ubuntu-2404-2604.md).

Once you have done all of that,

https://github.com/volatilityfoundation/volatility/releases/latest

Once unzipped, you can then run `~/Downloads/volatility_2.6_lin64_standalone/volatility_2.6_lin64_standalone`

### Viking (analysing GPS data) [DONE, NONROOT VERIFIED]

```bash
sudo apt install viking
```
