# Dell Ubuntu 18.04 Upgrade To 19.10

## Fix Expired Google GPG Key on Dell Image
```bash
wget -q -O - https://dl.google.com/linux/linux_signing_key.pub | sudo apt-key add -

wget -q -O - http://dell.archive.canonical.com/updates/dists/bionic-dell-service/Release.gpg | sudo apt-key add -
```

## Install Any Updates Prior and Reboot to Starting

```bash
sudo apt-get update && sudo apt-get upgrade -y

sudo apt autoremove -y && sudo apt clean -y

sudo apt-get upgrade -y fwupd gir1.2-javascriptcoregtk-4.0 gir1.2-webkit2-4.0 icedtea-8-plugin icedtea-netx initramfs-tools initramfs-tools-bin initramfs-tools-core libgl1-mesa-dri libgnome-desktop-3-17 libjavascriptcoregtk-4.0-18 libreoffice-avmedia-backend-gstreamer libreoffice-base-core libreoffice-calc libreoffice-core libreoffice-draw libreoffice-gnome libreoffice-gtk3 libreoffice-help-en-gb libreoffice-help-en-us libreoffice-impress libreoffice-l10n-en-gb libreoffice-l10n-en-za libreoffice-math libreoffice-ogltrans libreoffice-writer libwebkit2gtk-4.0-37 libxatracker2 linux-headers-oem linux-image-oem linux-oem netplan.io python3-software-properties python3-uno software-properties-common software-properties-gtk somerville-meta

sudo reboot now
```

## Manually Build AX1650 Wi-Fi Drivers Prior To Upgrade - Only If Kernel Version Is Prior To 5.1

* **Wi-Fi Drivers for the Killer AX1650 need to be manually buit via the Back Port**

* **Information Source:** https://support.killernetworking.com/knowledge-base/killer-ax1650-in-debian-ubuntu-16-04/


### 1. Download the Latest Git and Build-Essential packages:

```bash
sudo apt update

sudo apt-get install -y git build-essential
```

### 2. Download the Iwlwifi-Firmware.git repository:

```bash
git clone git://git.kernel.org/pub/scm/linux/kernel/git/firmware/linux-firmware.git

cd linux-firmware

sudo cp iwlwifi-* /lib/firmware/

cd ..

```

### 3. Create the Backported Iwlwifi Driver for your current setup:

```bash
git clone https://git.kernel.org/pub/scm/linux/kernel/git/iwlwifi/backport-iwlwifi.git

cd backport-iwlwifi

sudo make defconfig-iwlwifi-public

sudo make -j4

sudo make install
```

### ? This command might be necessary to force your machine to use the Driver from boot:

```bash
update-initramfs -u
```

### Try a Reboot

```bash
sudo reboot now
```
## ! PLEASE NOTE!!! - If you update Ubuntu after using a Backported driver, you may have to repeat Step 3 from the beginning to build a new Driver if the Kernel is.

## Upgrade Ubuntu 18.04 to 19.10
```bash
sudo apt update && sudo apt dist-upgrade -y
sudo apt install -y update-manager-core
```

### At the bottom of this release-upgrades, change the value of Prompt from Prompt=lts to Prompt=normal
#### Manual Method

```bash
sudo nano /etc/update-manager/release-upgrades
```

* From:
```
Prompt=lts
```
* To:
```
Prompt=normal
```
* Press Ctrl + X
* Press Y
* Press Enter

#### Auto Method

```bash
sudo sed -i 's/Prompt=lts/Prompt=normal/' /etc/update-manager/release-upgrades

```

### Change Version Name In sources.list

```bash
sudo sed -i 's/bionic/eoan/g' /etc/apt/sources.list
```

### Comment Out 3rd Party Source Lists - Excluding 'canonical-hwe-team-ubuntu-backport-iwlwifi-bionic.list'

```bash
sudo find /etc/apt/sources.list.d/ -type f -not -name 'canonical-hwe-team-ubuntu-backport-iwlwifi-bionic.list' -exec sed -i 's/^/#/' {} ';' 
```

### UPGRADE TIME :(

```bash
sudo apt-get update && sudo apt-get upgrade -y && sudo apt dist-upgrade && sudo apt autoremove && sudo apt clean 

sudo reboot now
```