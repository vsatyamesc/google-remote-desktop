# google-remote-desktop on Ubuntu 24 LTS, EC2 t3.small with memory swap

```
#!/usr/bin/env bash
set -euo pipefail

if [ ! -f /swapfile ]; then
  sudo fallocate -l 2G /swapfile
  sudo chmod 600 /swapfile
  sudo mkswap /swapfile
  sudo swapon /swapfile
  echo '/swapfile none swap sw 0 0' | sudo tee -a /etc/fstab
  sudo sysctl vm.swappiness=10
  echo 'vm.swappiness=10' | sudo tee -a /etc/sysctl.conf
fi

sudo apt-get update && sudo apt-get upgrade -y

sudo DEBIAN_FRONTEND=noninteractive apt-get install -y \
  xfce4 \
  xfce4-goodies \
  xvfb \
  dbus-x11 \
  x11-xserver-utils \
  wget \
  epiphany-browser

CRD_DEB="/tmp/chrome-remote-desktop_current_amd64.deb"
wget -q -O "$CRD_DEB" https://dl.google.com/linux/direct/chrome-remote-desktop_current_amd64.deb
chmod 644 "$CRD_DEB"

sudo apt-get install -y "$CRD_DEB"
sudo apt-get install -f -y
rm -f "$CRD_DEB"

cat << 'EOF' > ~/.chrome-remote-desktop-session
unset DBUS_SESSION_BUS_ADDRESS
exec /usr/bin/startxfce4
EOF

chmod +x ~/.chrome-remote-desktop-session

sudo groupadd -f chrome-remote-desktop
sudo usermod -a -G chrome-remote-desktop "$USER"
```
