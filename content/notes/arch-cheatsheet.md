---
id: Arch Install Cheat Sheet
aliases:
  - Arch Install Cheat Sheet
tags:
  - linux
---

1. Connect wifi

```
iwctl
station wlan0 show
station wlan scan
station wlan connect <WIFI>
exit
```

2. Update Pacman

```
pacman-key --init
pacman-key --populate archlinux
archinstall
(additional package git)
(include multilib)

install & reboot
```

3. nmcli device wifi connect <WIFI> password <PASSWORD>
4. ml4w

```
bash <(curl -s https://raw.githubusercontent.com/mylinuxforwork/dotfiles/main/setup.sh)
```

5. Nvidia

```
yay -S linux-headers nvidia-dkms qt5-wayland qt5ct libva libva-nvidia-driver-git

sudo nvim /etc/mkinitcpio.conf
MODULES=(... nvidia nvidia_modeset nvidia_uvm nvidia_drm ...)

echo "options nvidia_drm modeset=1 fbdev=1" | sudo tee /etc/modprobe.d/nvidia.conf

sudo mkinitcpio -P
reboot
```

6. Bluetooth

```
yay -S bluez bluez-utils

bluetoothctl
power on
scan on
pair XXX-XXX
connect XXX-XXX
```

7. Clone dotfiles

```
git config --global credential.helper store

mkdir ~/projects
cd ~/projects

git clone https://github.com/masonyc/dotfiles.git
cd dotfiles && chmod u+x arch-setup.sh && ./arch-setup.sh
```

8. Enable time shift in ml4w &

```
sudo systemctl enbale cronie.service

sudo -E timeshift-launcher
```

9. Containers & friends

```
newgrp docker
sudo groupadd docker
sudo usermod -aG docker $USER
sudo systemctl enable docker.socket

# init containerd - mannual /etc/containerd/config.toml
# containerd config default
```

And Change /etc/containerd/config.toml

```
[plugins."io.containerd.grpc.v1.cri".containerd.runtimes.runc.options]
  SystemdCgroup = true
```
