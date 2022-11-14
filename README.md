# Project BoxWine

#### A guide for setting up ARM platforms to run x86 & x86_64 apps and games, with install script.

## Table of Contents:

1. [Introduction](#intro)
1. [Important Notes](#notes)
1. [BoxWine Installer](#boxwine)
1. [Multiarch Setup](#multi)
1. [Install Box86](#box86)
1. [Install Box64](#box64)
1. [Install Wine Multiarch](#wine64)
1. [Install Wine 32-Bit Only](#wine32)
1. [Install Winetricks](#winetricks)
1. [Install Steam](#steam)
1. [Install Waydroid](#android)
1. [Setup 32-Bit Chroot](#chroot)
1. [Recommended Apps](#apps)
1. [Final Words](#final)
1. [Credits](#credits)

<a name="intro"/>

## Introduction

  This page is for anyone with an ARM Single Board Computer (SBC), System on a Module/Chip(SoM/SoC), or Workstation/Server Platform that wants support for running x86 Linux & Windows games and applications.
  
  Project BoxWine is an installer script and guide to help beginners and advanced users alike easily set up their ARM platforms for x86 computing with minimal effort, as well as provide an all-in-one resource for related projects. With the growing power of the ARM platform year over year we are seeing more powerful integrated GPU's, more PCI Express lanes, RAM, and of course the CPU's themselves. This project includes extra tools for the Solidrun LX2K platform, however I'd be more than happy to add more if others are able to help test or as I aquire more platforms. (seldom...)
  
<a name="notes"/>

## Important Notes

 The Project BoxWine guide is under active development and in it's infancy, especially the installer script itself. Please be aware the guide is primarily geared toward Debian systems, with Ubuntu, Void, Fedora and others to follow, probably in that order. Some distributions are made for multiarch, some with support for setting it up, while others have almost none at all. In the case of the latter you will only be able to run x86_64 Linux binaries and will be limited to Windows applications that are strictly 64-bit. In extremely rare cases, mainly in the server arena, your CPU may only support 64-bit instructions and not have 32-bit backwards compatability. Your milage may vary, however if you run into any issues please report them in the issue section so I can correct this guide/installer or add feature requests.
 
<a name="boxwine"/>

## BoxWine Installer

Description: Used to fully install Box86/64, Wine 32/64-Bit and Steam automatically or individually through the menu. Also includes extra tools and programs as well as patches for the HoneyComb LX2K platform.

Script Version: 0.1

Architectures Supported: aarch64 [currently armhf has limited support

Platforms Supported: Debian 11 [currently other versions have limited support]

Instructions: Run the installer by invoking one of the following commands below:

    ```./boxwine_install -h``` - Display BoxWine Help Screen
    ```./boxwine_install -m``` - Display BoxWine Selection Menu
    ```./boxwine_install -f``` - Install BoxWine for 64-bit Systems
    ```./boxwine_install -p``` - Display HoneyComb/PCIe Platforms Menu
    ```./boxwine_install -l``` - Install BoxWine for HoneyComb LX2K w/ platform specific patches
    
Download: https://github.com/Wooty-B/Project_BoxWine/releases/

<a name="multi"/>

## Multiarch Setup

Description: Some distributions have package managers and/or os level support for multiple architectures. This is required for enabling 32-Bit support which is needed by most Windows games.

Debian/Ubuntu Instructions:

	```
	sudo dpkg --add-architecture armhf
	sudo apt update && sudo apt upgrade
	```

<a name="box86"/>

## Install Box86

Description: Used to translate 32-Bit x86 Linux system calls into armhf compatable ones for native execution of x86 binaries/applications.

Debian/Ubuntu Instructions:

```
mkdir ~/box86_setup && cd ~/box86_setup
sudo apt install --yes --quiet --quiet \
	gcc-arm-linux-gnueabihf \
	git \
	build-essential \
	cmake \
	curl
git clone https://github.com/ptitSeb/box86.git && cd box86
mkdir build && cd build && cmake .. -DARM_DYNAREC=ON -DCMAKE_BUILD_TYPE=RelWithDebInfo && make -j$(nproc)
sudo make install
cd ../.. && sudo rm -rf ~/box86_setup
sudo systemctl restart systemd-binfmt
```
  
<a name="box64"/>

## Install Box64

Description: Used to translate 32-Bit x86_64 Linux system calls into aarch64 compatable ones for native execution of x86_64 binaries/applications.

Debian/Ubuntu Instructions:

```
sudo apt update && sudo apt upgrade
mkdir ~/box64_setup && cd ~/box64_setup
sudo apt install --yes --quiet --quiet \
	gcc-arm-linux-gnueabihf \
	git \
	build-essential \
	cmake \
	curl
git clone https://github.com/ptitSeb/box64.git && cd box64
mkdir build && cd build && cmake .. -DARM_DYNAREC=ON -DCMAKE_BUILD_TYPE=RelWithDebInfo && make -j$(nproc)
sudo make install
cd ../.. && sudo rm -rf ~/box64_setup
sudo systemctl restart systemd-binfmt
```

<a name="wine64"/>

## Install Wine Multiarch

Description: Used to install Wine 32-Bit with experimental 64-Bit support. This will allow you to run Windows 32-bit games and some Windows 64-bit applications.

Requirements: Box86, Box64

Wine Versions Tested: 5.18; 5.21; 6.0.2; 7.0.0.0

Debian/Ubuntu Instructions:

```
mkdir ~/wine3264_setup && cd ~/wine3264_setup
wget https://dl.winehq.org/wine-builds/debian/dists/buster/main/binary-i386/wine-stable-i386_6.0.2~buster-1_i386.deb
wget https://dl.winehq.org/wine-builds/debian/dists/buster/main/binary-i386/wine-stable_6.0.2~buster-1_i386.deb
wget https://dl.winehq.org/wine-builds/debian/dists/buster/main/binary-amd64/wine-stable-amd64_6.0.2~buster-1_amd64.deb
wget https://dl.winehq.org/wine-builds/debian/dists/buster/main/binary-amd64/wine-stable_6.0.2~buster-1_amd64.deb
dpkg-deb -xv wine-stable-i386_6.0.2~buster-1_i386.deb wine-installer
dpkg-deb -xv wine-stable_6.0.2~buster-1_i386.deb wine-installer
dpkg-deb -xv wine-stable-amd64_6.0.2~buster-1_amd64.deb wine-installer
dpkg-deb -xv wine-stable_6.0.2~buster-1_amd64.deb wine-installer
mv wine-installer/opt/wine* ~/wine
rm wine*.deb && sudo rm -rf wine-installer
sudo ln -s ~/wine/bin/wine /usr/local/bin/wine
sudo ln -s ~/wine/bin/wineboot /usr/local/bin/wineboot
sudo ln -s ~/wine/bin/winecfg /usr/local/bin/winecfg
sudo ln -s ~/wine/bin/wineserver /usr/local/bin/wineserver
sudo chmod +x /usr/local/bin/wine /usr/local/bin/wineboot /usr/local/bin/winecfg /usr/local/bin/wineserver
WINEPREFIX=~/.wine WINEARCH=win32 wine winecfg
WINEPREFIX=~/.wine64 WINEARCH=win64 wine winecfg
echo 'alias wine64="WINEPREFIX=~/.wine64 wine"' | tee ~/.bashrc
source ~/.bashrc
cd .. && sudo rm -rf ~/wine3264_setup
```
  
<a name="wine32"/>

## Install Wine 32-Bit

Description: Used to install Wine 32-Bit for platforms only supporting 32-Bit execution such as armhf. This will allow you to run Windows 32-bit games and applications.

Requirements: Box86

Wine Versions Tested: 5.18; 5.21; 6.0.2; 7.0.0.0

Debian/Ubuntu Instructions:

```
mkdir ~/wine32_setup && cd ~/wine32_setup
wget https://dl.winehq.org/wine-builds/debian/dists/buster/main/binary-i386/wine-stable-i386_6.0.2~buster-1_i386.deb
wget https://dl.winehq.org/wine-builds/debian/dists/buster/main/binary-i386/wine-stable_6.0.2~buster-1_i386.deb
dpkg-deb -xv wine-stable-i386_6.0.2~buster-1_i386.deb wine-installer
dpkg-deb -xv wine-stable_6.0.2~buster-1_i386.deb wine-installer
mv wine-installer/opt/wine* ~/wine
rm wine*.deb && sudo rm -rf wine-installer
sudo ln -s ~/wine/bin/wine /usr/local/bin/wine
sudo ln -s ~/wine/bin/wineboot /usr/local/bin/wineboot
sudo ln -s ~/wine/bin/winecfg /usr/local/bin/winecfg
sudo ln -s ~/wine/bin/wineserver /usr/local/bin/wineserver
sudo chmod +x /usr/local/bin/wine /usr/local/bin/wineboot /usr/local/bin/winecfg /usr/local/bin/wineserver
WINEPREFIX=~/.wine WINEARCH=win32 wine winecfg
source ~/.bashrc
cd .. && sudo rm -rf ~/wine32_setup
```

<a name="winetricks"/>

## Install Winetricks

Description: Winetricks is a useful tool that helps install Windows libraries required by certain applications, which can fix or help improve performance in games and applications. Below will help with installing Winetricks as well as information on useful packages.

Requirements: Wine 32-Bit and/or 64-Bit, Box86 and/or Box64

Debian/Ubuntu Instructions:

```
sudo apt-get install --yes --quiet --quiet \
	cabextract \
	zenity
wget https://raw.githubusercontent.com/Winetricks/winetricks/master/src/winetricks
sudo chmod +x winetricks && sudo mv winetricks /usr/local/bin/
BOX86_NOBANNER=1 winetricks
```

Useful/Recommended Packages: [LINK]

<a name="steam"/>

## Install Steam

Description: Nothing needs to be said, however Steam is one of the largest gaming platforms supporting Windows, macOS, and Linux. Steam Proton can be used to enable Windows game support and can be enabled in Steam's Settings menu.

Requirements: Box86, Box64 [Only for 64-Bit support, not needed for 32-Bit only systems/games]

Debian/Ubuntu Instructions:

```
mkdir ~/steam_setup && cd ~/steam_setup
echo 'export STEAMOS=1' | sudo tee /etc/profile.d/steam.sh
echo 'export STEAM_RUNTIME=1' | sudo tee /etc/profile.d/steam.sh
source /etc/profile.d/steam.sh
wget https://steamcdn-a.akamaihd.net/client/installer/steam.deb
sudo dpkg -i steam.deb
sudo apt install --yes --quiet --quiet \
	libnm0 \
	libtcmalloc-minimal4 \
	libgmp10:armhf \
	libgnutls30:armhf \
	libvorbisfile3:armhf \
	libsdl2-2.0-0:armhf \
	libxtst6:armhf \
	libbz2-1.0:armhf \
	libselinux1:armhf \
	libjpeg62-turbo:armhf \
	libcairo2:armhf \
	libfontconfig1:armhf \
	libxrender1:armhf \
	libxinerama1:armhf \
	libxi6:armhf \
	libxrandr2:armhf \
	libxcursor1:armhf \
	libxcomposite1:armhf \
	libfreetype6:armhf \
	libpng16-16:armhf \
	libopenal1:armhf \
	libsm6:armhf \
	libice6:armhf \
	libuuid1:armhf \
	libc6:armhf \
	libegl1:armhf \
	libgbm1:armhf \
	libgl1-mesa-dri:armhf \
	libgl1:armhf
echo 'steam -no-browser -noreactlogin steam://open/minigameslist' | tee ~/run_steam.sh
chmod 755 ~/run_steam.sh
cd .. && sudo rm -rf ~/steam_setup
cd ~ && ./run_steam.sh
```

Additional Description: The instructions below rename the install script to ```runsteam``` and moves it to ```/usr/local/bin``` so it can be executed directly from the terminal. 

Additional Instructions:

```
sudo mv ~/run_steam.sh /usr/local/bin/runsteam
```
<a name="android"/>

## Install Waydroid

Description: Waydroid is used for running Android applications natively in their own windows, or optionally a Desktop environment for running them inside of. Performance and milage may vary, but still looking for a better way of running android apps natively under ARM Linux.

Instructions:

Follow the guide here: https://docs.waydro.id/usage/install-on-desktops

Edit the file /etc/gbinder.conf:
```
    	[General]
    	ApiLevel = 29
```

<a name="chroot"/>

## Setup 32-Bit Chroot

Description: This is not required in most cases if your distribution supports multiarch. However, this may be useful for getting a 32-Bit environment on distributions that don't support multiarch but have 32-Bit package support. This could also be useful for other use cases.

Debian/Ubuntu Instructions:

   	```
    	sudo apt install schroot debootstrap
    	sudo mkdir -p /srv/chroot/ubuntu-armhf
    	sudo debootstrap --arch armhf --foreign hirsute /srv/chroot/ubuntu-armhf http://ports.ubuntu.com/ubuntu-ports
    	sudo chroot "/srv/chroot/ubuntu-armhf" /debootstrap/debootstrap --second-stage
    	sudo nano /etc/schroot/chroot.d/ubuntu-armhf.conf
    	
	    	[ubuntu-armhf]
	    	description=Ubuntu Armhf 32-Bit chroot
	    	aliases=ubuntu-armhf
	    	type=directory
	    	directory=/srv/chroot/ubuntu-armhf
	    	profile=desktop
	    	personality=linux
	    	preserve-environment=true
	    	root-users=<username>
	    	users=<username>
	
	sudo nano /etc/schroot/desktop/nssdatabases
	
	    	# System databases to copy into the chroot from the host system.
	    	#
	    	# <database name>
	    	#passwd
	    	shadow
	    	#group
	    	gshadow
	    	services
	    	protocols
	    	#networks
	    	#hosts
	    	#user
      	
    	sudo nano /srv/chroot/ubuntu-armhf/var/lib/dpkg/statoverride
	 
   	   	root root 2755 /usr/bin/crontab
	
    	sudo schroot -c ubuntu-armhf
    	apt install nano
    	nano /etc/apt/sources.list
	  
   	    	deb http://us.ports.ubuntu.com/ubuntu-ports/ hirsute main restricted universe multiverse
	    	deb http://us.ports.ubuntu.com/ubuntu-ports/ hirsute-updates main restricted universe multiverse
	    	deb http://us.ports.ubuntu.com/ubuntu-ports/ hirsute-backports main restricted universe multiverse
	    	deb http://us.ports.ubuntu.com/ubuntu-ports/ hirsute-security main restricted universe multiverse
    	
    	apt update && apt upgrade
    	nano ~/.bashrc
	   
   	    	export LANGUAGE="C"
	    	export LC_ALL="C"
	    	export DISPLAY=:0
	    	export CPU_MHZ=2000
	
    	adduser <username>
    	usermod -a -G sudo <username>
    	su - <username>
    	nano ~/.bashrc
	
    	    	export LANGUAGE="C"
	    	export LC_ALL="C"
	    	export DISPLAY=:0
	    	export CPU_MHZ=2000
  	
    	exit
    	exit
 	```
  
<a name="apps"/>

## Recommended Apps

Description: Useful applications for installing common apps and games, or tools specific to ARM systems. Recommendations welcome!

Pi-Apps: A very useful application for installing apps like Balena Etcher, Minecraft, Arduino, LibreOffice and more! Only supports Debian systems and unofficially Ubuntu...

```
wget -qO- https://raw.githubusercontent.com/Botspot/pi-apps/master/install | bash

```
GLmark2: A GL benchmarking tool for determining general GL performance for older systems, and can be useful for comparing benchmanerks between ARM boards or testing GPU performance.

GLmark2 Debian/Ubuntu Instructions:
```
mkdir ~/glmark_setup && cd ~/glmark_setup
sudo apt install --yes --quiet --quiet \
	meson \
	libpng-dev \
	libjpeg-dev \
	libx11-dev \
	libwayland-dev \
	wayland-protocols \
	libegl-dev \
	libgl-dev \
	build-essential \
	bison \
	flex \
	bc
git clone https://github.com/glmark2/glmark2.git && cd glmark2
meson setup build -Dflavors=wayland-gl,wayland-glesv2,x11-gl,x11-glesv2
ninja -C build
ninja -C build install
cd .. && sudo rm -rf ~/glmark_setup
```
Freon: A temperature monitor for Gnome, which can show CPU, HDD and GPU temperatures for supported platforms

Install using the Gnome Estension Manager, link to Freon here: https://extensions.gnome.org/extension/841/freon/

<a name="final"/>

## Final Words

This is heavily under construction at the moment, and I appreciate all feedback to help make this guide and installer as best as it can be. This guide was written with the main intention of improving the library of 3D games available to users and fans of the ARM and RISC architectures. I hope one day this project can include many distributions, and other RISC based arcitechtures like RISC-V and PowerPC 64-Bit Little Endian. (Both supported by Box86 & Box64!) 

If you have a distribution you would like to see supported, are getting an error during any of the steps/installer, or anything else, please open an issue and anyone else having similar requests or issues please bump them so I can put more focus on them.

<a name="credits"/>

## Credits

ptitseb [Box86 & Box64]: https://github.com/ptitSeb 

WineHQ [Wine]: https://www.winehq.org/

Valve: https://www.valvesoftware.com/en/

Pi-Apps: https://pi-apps.io/

GLmark2: https://github.com/glmark2/glmark2
