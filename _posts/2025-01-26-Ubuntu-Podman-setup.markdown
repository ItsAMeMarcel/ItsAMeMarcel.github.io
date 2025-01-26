---
layout: post
title:  "Install Podman Desktop on Ubuntu 22.04 LTS"
date:   2025-01-26 11:21:10 +0100
categories: container
---

<img src="https://github.com/ItsAMeMarcel/blog-resources/blob/main/images/2025-01-26-Ubuntu-Podman-setup/title.png?raw=true" width="600" height="600" alt="">



Podman is a daemonless container engine for developing, managing, and running OCI Containers on your Linux system. In this guide, we will walk you through the steps to install Podman Desktop on Ubuntu 22.04 LTS.

# Install Podman

First, update your package list and install Podman:

```sh
sudo apt-get update
sudo apt-get -y install podman
```
# Install Podman Compose

To install `podman-compose`, follow these steps:

## 1. Install `python3-pip`:
```sh
sudo apt-get -y install python3-pip
```
## 2. Install `podman-compose` using `pip3`:
```sh
sudo pip3 install podman-compose
```
# Install Podman Desktop

To install Podman Desktop, we will use Flatpak. First, install Flatpak:

```sh
sudo apt install flatpak
```
Then, install Podman Desktop from Flathub:

```sh
flatpak install --user flathub io.podman_desktop.PodmanDesktop
```
Finatlly you can run Podman Desktop with:

```sh
flatpak run io.podman_desktop.PodmanDesktop
```
# Fix CNI Plugin Problem

It can be that you are facing the following problem with Podman Compose:

```
WARN[0000] Error validating CNI config file /home/mike/.config/cni/net.d/cni-podman1.conflist: [plugin bridge does not support config version "1.0.0" plugin portmap does not support config version "1.0.0" plugin firewall does not support config version "1.0.0" plugin tuning does not support config version "1.0.0"]
```
It can be fixed with the following installation:

```bash
sudo snap install curl

curl -O https://archive.ubuntu.com/ubuntu/pool/universe/g/golang-github-containernetworking-plugins/containernetworking-plugins_1.1.1+ds1-3build1_amd64.deb

sudo dpkg -i containernetworking-plugins_1.1.1+ds1-3build1_amd64.deb
```
More infomation about the background you can found in [this blog post](https://www.michaelmcculley.com/updating-cni-plugins-for-podman-a-step-by-step-guide/) and in this [bug report](https://bugs.launchpad.net/ubuntu/+source/libpod/+bug/2024394) 

# Optional: Create Desktop Entry for Podman Desktop

To create a desktop entry for Podman Desktop, create a symbolic link to the application:

```sh
ln -s ~/.local/share/flatpak/exports/share/applications/io.podman_desktop.PodmanDesktop.desktop ~/.local/share/applications/
```
As you can see, Flatpak already offers a `.desktop` file; we just need to link it to the correct folder. This step creates a symbolic link to the Podman Desktop application in your local applications directory. This allows your desktop environment to recognize and display the Podman Desktop application in your application menu, making it easier to launch.

Edit the desktop entry to set the correct icon path:
```sh
vim ~/.local/share/applications/io.podman_desktop.PodmanDesktop.desktop
```
Change the `Icon=` line to:
```sh
Icon=<your_home_path>/.local/share/flatpak/exports/share/icons/hicolor/128x128/apps/io.podman_desktop.PodmanDesktop.png
```
Finally, restart your desktop environment to apply the changes. For X11, you can do this by pressing `[ALT] + [F2]`, entering `r`, and pressing enter.

# Further Resources
- **[Podman Installation Documentation](https://podman.io/docs/installation)**

- **[Podman Compose Repository](https://github.com/containers/podman-compose)**

- **[Podman Desktop Installation](https://podman-desktop.io/docs/installation/linux-install)**

