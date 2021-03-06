#Copyright 2012 CurlyMo, Koen Kanters, Robin Nilsson <development@xbian.org>
#
#This file is part of XBian - XBMC on the Raspberry Pi.
#
#XBian is free software: you can redistribute it and/or modify it under the 
#terms of the GNU General Public License as published by the Free Software 
#Foundation, either version 3 of the License, or (at your option) any later 
#version.
#
#XBian is distributed in the hope that it will be useful, but WITHOUT ANY 
#WARRANTY; without even the implied warranty of MERCHANTABILITY or FITNESS 
#FOR A PARTICULAR PURPOSE. See the GNU General Public License for more 
#details.
#
#You should have received a copy of the GNU General Public License along 
#with XBian. If not, see <http://www.gnu.org/licenses/>
#

# - IMPORTANT - 
# Restore the official Raspbian image from: http://www.raspberrypi.org/downloads to your 
#SD card (28-10-2012). First boot: Select 'Finish' to dimiss initial tutorial (raspi-config). 
#SSH will be running (on bootup, IP is printed in logs). 
#Now SSH into your Pi with the following credentials Username: pi Password: raspbian

#Removing unnecessary packages and installing required packages
sudo su

apt-get update

apt-get purge -y libice6 libxt6 xserver-common xserver-xorg xserver-xorg-core xserver-xorg-input-all \
xserver-xorg-video-fbdev xserver-xorg-input-synaptics xserver-xorg-input-evdev x11-utils x11-xkb-utils aspell \
lxappearance lxde lxde-common lxde-core lxde-icon-theme lxinput lxmenu-data lxpanel lxpolkit lxrandr lxsession \
lxsession-edit lxshortcut lxtask lxterminal gnome-icon-theme gnome-themes-standard libgnome-keyring-common \
libgnome-keyring0:armhf libsoup-gnome2.4-1:armhf idle desktop-base desktop-file-utils dbus-x11 dictionaries-common \
fbset fonts-freefont-ttf gconf2 gdb gtk2-engines:armhf hicolor-icon-theme libgdk-pixbuf2.0-0:armhf \
libgl1-mesa-glx:armhf menu omxplayer penguinspuzzle x11-common xdg-utils xkb-data libatasmart4:armhf \
libcairo-gobject2:armhf libcups2:armhf aptitude-common curl debconf-utils debian-reference-common debian-reference-en \
dphys-swapfile dpkg-dev fake-hwclock g++ gcc hardlink keyboard-configuration libatasmart4:armhf libblas3 libblas3gf \
libbluetooth3:armhf libboost-iostreams1.50.0 libdrm2:armhf libgfortran3:armhf libglib2.0-data libgudev-1.0-0:armhf \
libiw30:armhf liblapack3 liblapack3gf libluajit-5.1-common libpci3:armhf libpolkit-gobject-1-0:armhf libsgutils2-2 \
libusb-1.0-0:armhf luajit menu-xdg module-init-tools ncdu pciutils pkg-config python-rpi.gpio python3 python3-minimal \
python3-numpy python3-rpi.gpio python3.2 python3.2-minimal raspi-copies-and-fills rsyslog shared-mime-info strace \
usbutils dillo fontconfig-config libbluetooth3 libfltk1.3:armhf libfontconfig1:armhf libfreetype6:armhf \
libjbig2dec0 libopenjpeg2 libxext6 libxft2 libxinerama1 libxrender1 mupdf ttf-dejavu-core fuse gettext-base \
libasprintf0c2 libasyncns0 libaudit0 libcaca0 libdirectfb-1.2-9 libflac8 libfm-data libfuse2 libgif4 libid3tag0 \
libjavascriptcoregtk-3.0-0 libjson0 libmad0 libmenu-cache1 libmikmod2 libmtdev1 libogg0 libpciaccess0 libportmidi0 \
libsndfile1 libts-0.0-0 libvorbis0a libvorbisenc2 libvorbisfile3 libvte-common libwebkitgtk-3.0-common libwebp2 \
libwnck-common libxkbfile1 python-support tcl8.5 tsconf zenity-common tasksel tasksel-data libept1.4.12 libxapian22 \
libxau6 libxcb1 libxdmcp6:armhf raspi-config ed kbd krb5-locales alsa-base alsa-utils gcc-4.4-base:armhf \
gcc-4.5-base:armhf gcc-4.6 gcc-4.6-base libasound2 libgomp1 libsamplerate0 triggerhappy libtagcoll2-dev libgmp10 \
libmpc2 libmpfr4 libtirpc1 libp11-kit0 libswitch-perl perl perl-modules tasksel tasksel-data libx11-data libxau6 libxcb1 \
libxdmcp6 make libx11-6 libraspberrypi-doc

apt-get install -y libtiff-dev liblockdev1-dev wget rpcbind screen git dialog libsamplerate0 \
libmysqlclient18 libsmbclient libssh-4 libavahi-client-dev libmicrohttpd-dev libtinyxml-dev libyajl-dev liblzo2-dev \
libjpeg-dev libfribidi-dev libpcre3-dev libcdio-dev libfreetype6 libpython2.7 libnfs-dev nfs-common samba-common-bin avahi-daemon \
udisks usbmount git-core ntfs-3g wireless-tools libjasper-dev  libass-dev --fix-missing

apt-get autoremove -y

apt-get purge -y $(dpkg --get-selections | grep deinstall | awk '{print $1}')

rm -rf /var/cache/apt/archives/*

apt-get clean

#Regenerating SSH unique SSH keys:
rm /etc/ssh/ssh_host_* && dpkg-reconfigure openssh-server

#Setting up xbian user and deleting pi user
useradd -G sudo,input -m -s /bin/bash xbian
echo "xbian:raspberry" | chpasswd

#Remove user pi from sudoers and add xbian user
sed -i 's/pi ALL=(ALL) NOPASSWD: ALL//g' /etc/sudoers
echo "xbian ALL=(ALL) NOPASSWD: /usr/local/sbin/xbian-config, /sbin/halt, /sbin/reboot" >> /etc/sudoers

# Relogin with ssh; user:xbian pass:raspberry

#Kill all processes started by user pi
echo raspberry | sudo -S chmod +s /bin/su
sudo su
for i in $(pgrep -u pi); do kill -9 $i; done;

#Deleting user pi
userdel pi
rm -rf /home/pi

#Disabling root login
sed -i 's/PermitRootLogin yes/PermitRootLogin no/g' /etc/ssh/sshd_config

#Set hostname
echo "xbian" > /etc/hostname
hostname xbian
sed -i 's/raspberrypi/xbian/g' /etc/hosts

#Clone xbian git
cd /home/xbian/
git clone --depth 5 -b xbian-alpha3 https://github.com/xbianonpi/xbian.git source
cd source

#Installation
cp -R etc/* /etc/
cp -R usr/* /usr/
rm -rf /lib/modules/*
cp -R lib/* /lib/
rm -rf /boot/*
cp -R boot/* /boot/
cp -R home/* /home/
rm -rf /opt/vc
cp -r opt/vc /opt
cp -R var/* /var/

# Correcting permissions
chmod +x /etc/init.d/*
chmod +x /usr/local/sbin/*
chmod +x /usr/local/bin/*
chmod +x /usr/local/lib/xbmc/xbmc.bin
chmod +x /opt/vc/bin/*
chmod +x /opt/vc/sbin/*
chown -R xbian:xbian /home/xbian/.xbmc

#Update rc.d
update-rc.d xbmc defaults
update-rc.d bootlocal defaults
update-rc.d lirc defaults
update-rc.d rpcbind defaults

#Fixing broken symbolic links
rm /usr/local/lib/libtag_c.so.0
ln -s /usr/local/lib/libtag_c.so.0.0.0 /usr/local/lib/libtag_c.so.0
rm /usr/local/lib/libshairport.so.0
ln -s /usr/local/lib/libshairport.so.0.0.0 /usr/local/lib/libshairport.so.0
rm /usr/local/lib/libtag.so.1
ln -s /usr/local/lib/libtag.so.1.12.0 /usr/local/lib/libtag.so.1
ln -s /usr/local/lib/liblirc_client.so.0.2.1 /usr/local/lib/liblirc_client.so.0
ln -s /usr/local/lib/liblirc_client.so.0.2.1 /usr/local/lib/liblirc_client.so
ldconfig

# Set locale to c to prevent weird characters in xbian-config and c compiling
dpkg-reconfigure locales
#First window select ok
#Second window select en_GB.UTF-8

#Login as user xbian and set locates and as user root
export LANG=C
export LC_ALL=C
export LANGUAGE=C

#Make locale settings permanent
sed -i 's/export PATH/export PATH\nexport LANG=C\nexport LC_ALL=C\nexport LANGUAGE=C/g' /etc/profile

#Delete auto start of raspi-config
rm /etc/profile.d/raspi-config.sh

#Enable xbian autologin
sed -i 's/1:2345:respawn:\/sbin\/getty --noclear 38400 tty1/\#1:2345:respawn:\/sbin\/getty 38400 tty1\n1:2345:respawn:\/bin\/login -f xbian tty1 <\/dev\/tty1 >\/dev\/tty1 2>\&1/g' /etc/inittab
sed -i '/root/d' /etc/inittab
reboot

#After the reboot execute this command
sudo su
depmod -a

#Clear logs
rm /var/log/*.gz
rm /var/log/*log.*
for logs in `find /var/log -type f`; do echo '' > $logs; done
echo "" > /root/.bash_history
echo "" > /home/xbian/.bash_history

#Removing swap file & XBian repo clone
rm /var/swap
rm -rf /home/xbian/source

#Add apt sources in /etc/apt/sources.list
wget -O - http://[aptsource]/xbian.gpg.key | apt-key add -
apt-get update 
apt-get install -y xbian-update
apt-get autoremove -y

#Resize SD upon next boot
update-rc.d firstrun defaults
