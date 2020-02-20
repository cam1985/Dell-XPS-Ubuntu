# Post 19.10 Upgrade

```bash
sudo apt-get update && sudo apt-get upgrade -y
```

## Change "GRUB_CMDLINE_LINUX_DEFAULT" FROM: "quiet splash":
```bash
GRUB_CMDLINE_LINUX_DEFAULT="mem_sleep_default=deep"
```
## Install GNOME Tweaks
```bash
sudo apt install -y gnome-tweaks
sudo apt install gnome-tweak-tool 
```
* In Tweaks App:
  * Change GTK & icon theme
  * Automatically Centre new windows
  * Show the current weekday in the top bar clock

## Enable Universe Repository

```bash
sudo add-apt-repository -y universe
sudo apt -y update
sudo apt -y full-upgrade
```

## Install Power Management Tools

```bash
sudo add-apt-repository -y ppa:linrunner/tlp
sudo apt -y update
sudo apt -y install thermald tlp tlp-rdw powertop
```

## Fix Sleep/Wake Bluetooth Bug

```bash
sudo sed -i '/RESTORE_DEVICE_STATE_ON_STARTUP/s/=.*/=1/' /etc/default/tlp
```

## Fix Audio Feedback/White Noise from Headphones on Battery Bug

```bash
sudo sed -i '/SOUND_POWER_SAVE_ON_BAT/s/=.*/=0/' /etc/default/tlp
```

## Restart TLP Service

```bash
sudo systemctl restart tlp
```

## nVidia Graphics and codecs

### Enable PRIME Offloading on the NVIDIA GPU
* *This may increase battery drain but will allow dynamic switching of the NVIDIA GPU without having to log out*


### Add repository with Xorg Builds containing required NVIDIA patches.
### Enable Proprietary GPU PPA
```bash
sudo add-apt-repository -y ppa:graphics-drivers/ppa

sudo apt -y update
sudo apt -y upgrade
sudo apt -y install nvidia-driver-440 nvidia-settings # 435 is the minimum version to use PRIME offloading.
```
### Create simple script for launching programs on the NVIDIA GPU
```bash
sudo echo '__NV_PRIME_RENDER_OFFLOAD=1 __GLX_VENDOR_LIBRARY_NAME="nvidia" __VK_LAYER_NV_optimus="NVIDIA_only" exec "$@"' >> /usr/local/bin/prime
sudo chmod +x /usr/local/bin/prime
```
### Create xorg.conf.d directory (If it doesn't already exist) and copy PRIME configuration file
```bash
sudo mkdir -p /etc/X11/xorg.conf.d/
sudo wget https://raw.githubusercontent.com/JackHack96/dell-xps-9570-ubuntu-respin/master/10-prime-offload.conf
sudo mv 10-prime-offload.conf /etc/X11/xorg.conf.d/
sudo apt -y update
sudo ubuntu-drivers autoinstall
```

### Enable modesetting on the NVIDIA Driver (Enables use of offloading and PRIME Sync)
```bash
echo "options nvidia-drm modeset=1" | sudo tee -a /etc/modprobe.d/nvidia-drm.conf
```

### Install codecs
```bash
sudo apt -y install ubuntu-restricted-extras va-driver-all vainfo libva2 gstreamer1.0-libav gstreamer1.0-vaapi
```

### Enable high quality audio
```bash
echo "# This file is part of PulseAudio.
#
# PulseAudio is free software; you can redistribute it and/or modify
# it under the terms of the GNU Lesser General Public License as published by
# the Free Software Foundation; either version 2 of the License, or
# (at your option) any later version.
#
# PulseAudio is distributed in the hope that it will be useful, but
# WITHOUT ANY WARRANTY; without even the implied warranty of
# MERCHANTABILITY or FITNESS FOR A PARTICULAR PURPOSE. See the GNU
# General Public License for more details.
#
# You should have received a copy of the GNU Lesser General Public License
# along with PulseAudio; if not, see <http://www.gnu.org/licenses/>.
## Configuration file for the PulseAudio daemon. See pulse-daemon.conf(5) for
## more information. Default values are commented out.  Use either ; or # for
## commenting.
daemonize = no
; fail = yes
; allow-module-loading = yes
; allow-exit = yes
; use-pid-file = yes
; system-instance = no
; local-server-type = user
; enable-shm = yes
; enable-memfd = yes
; shm-size-bytes = 0 # setting this 0 will use the system-default, usually 64 MiB
; lock-memory = no
; cpu-limit = no
high-priority = yes
; nice-level = -11
; realtime-scheduling = yes
realtime-priority = 9
; exit-idle-time = 20
; scache-idle-time = 20
; dl-search-path = (depends on architecture)
; load-default-script-file = yes
; default-script-file = /etc/pulse/default.pa
; log-target = auto
; log-level = notice
; log-meta = no
; log-time = no
; log-backtrace = 0
resample-method = soxr-vhq
; avoid-resampling = false
; enable-remixing = yes
; remixing-use-all-sink-channels = yes
enable-lfe-remixing = yes
; lfe-crossover-freq = 0
flat-volumes = no
; rlimit-fsize = -1
; rlimit-data = -1
; rlimit-stack = -1
; rlimit-core = -1
; rlimit-as = -1
; rlimit-rss = -1
; rlimit-nproc = -1
; rlimit-nofile = 256
; rlimit-memlock = -1
; rlimit-locks = -1
; rlimit-sigpending = -1
; rlimit-msgqueue = -1
; rlimit-nice = 31
rlimit-rtprio = 9
; rlimit-rttime = 200000
default-sample-format = float32le
default-sample-rate = 48000
alternate-sample-rate = 44100
default-sample-channels = 2
default-channel-map = front-left,front-right
default-fragments = 2
default-fragment-size-msec = 125
; enable-deferred-volume = yes
deferred-volume-safety-margin-usec = 1
; deferred-volume-extra-delay-usec = 0" | sudo tee /etc/pulse/daemon.conf
```

### Intel microcode
```bash
sudo apt -y install intel-microcode iucode-tool
```

### Enable power saving tweaks for Intel chip
```bash
if [[ $(uname -r) == *"4.15"* ]]; then
    echo "options i915 enable_fbc=1 enable_guc_loading=1 enable_guc_submission=1 disable_power_well=0 fastboot=1" | sudo tee /etc/modprobe.d/i915.conf
else
    echo "options i915 enable_fbc=1 enable_guc=3 disable_power_well=0 fastboot=1" | sudo tee /etc/modprobe.d/i915.conf
fi
```

### Let users check fan speed with lm-sensors
```bash
echo "options dell-smm-hwmon restricted=0 force=1" | sudo tee /etc/modprobe.d/dell-smm-hwmon.conf
if < /etc/modules grep "dell-smm-hwmon" &>/dev/null
then
    echo "dell-smm-hwmon is already in /etc/modules!"
else
    echo "dell-smm-hwmon" | sudo tee -a /etc/modules
fi
sudo update-initramfs -u
```

### Ask for disabling tracker
```bash
systemctl mask tracker-extract.desktop tracker-miner-apps.desktop tracker-miner-fs.desktop tracker-store.desktop; break;;
```

### Ask for disabling fingerprint reader - It Doesn't work so might as well disable it.
```bash
echo "# Disable fingerprint reader
        SUBSYSTEM==\"usb\", ATTRS{idVendor}==\"27c6\", ATTRS{idProduct}==\"5395\", ATTR{authorized}=\"0\"" | sudo tee /etc/udev/rules.d/fingerprint.rules
```


## Configure Git
* *Information Source:* https://devconnected.com/how-to-setup-ssh-keys-on-github/

### Pre-Requisite - Ensure Git is installed
```bash
sudo apt-get install -y git
```
### Create GitHub Account if not one

### In terminal configure Git
```bash
git config --global user.name "your_username"
git config --global user.email "email@example.com"
```

### Confirm Settings
```bash
git config --global -l
```

### Create SSH Keys  
```bash
ssh-keygen -t rsa -b 4096 -C "email@example.com" -f ~/.ssh/username_devicename_github_rsa
```

### Configure your SSH keys
```bash
sudo nano ~/.ssh/config
```

* *Add the content below to the file ...*
```bash
Host *
    Hostname github.com
    User git
    IdentityFile ~/.ssh/username_devicename_github_rsa
```

### Add SSH key to your GitHub Account
* https://help.github.com/en/github/authenticating-to-github/adding-a-new-ssh-key-to-your-github-account

### Make sure xclip is installed
```bash
sudo apt-get install -y xclip
```

### Copy GitHub SSH Public Key to Clipboard 
```bash
xclip -sel clip < ~/.ssh/username_devicename_github_rsa.pub
```

* Go to GitHub.com, 
* In the upper-right corner of any page, click your profile photo, then click Settings.
* In the user settings sidebar, click SSH and GPG keys. 
* Click New SSH key or Add SSH key. 
* In the "Title" field, add a descriptive label for the new key. For example, if you're using a personal Mac, you might call this key "username@devicename".
* Paste your key into the "Key" field. 
* Click Add SSH key


### Test GitHub SSH keys

## TODO Everything Below Needs Organising Properly. 


## Software

### Visual Studio Code
	* VSCode Git Setup Directory/Repo Information -  http://www.notyourdadsit.com/blog/2018/4/3/cheatsheet-setup-github-on-visual-studio-code
	
# Install VSCode Insiders Edition

```bash
sudo apt-get install -y snap
sudo snap install --classic code-insiders
```


# Information Sources:
* https://wiki.archlinux.org/index.php/Dell_XPS_15_7590#Hibernate
* https://github.com/JackHack96/dell-xps-9570-ubuntu-respin
* https://github.com/JackHack96/dell-xps-9570-ubuntu-respin/blob/master/xps-tweaks.sh

