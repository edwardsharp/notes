# RASPBERRY PI USEFUL BITZ

# contentz

* [mount ext4 fs on OSX](#mount-ext4-fs-on-osx)
* [initialize a disk](#initialize-a-disk)
* [dat](#dat)
* [use .local domain (ex: raspberrypi.local)](#use-local-domain-ex-raspberrypilocal)
* [AUTO X STARTZ](#auto-x-startz)
* [disable screen from blank'n (mostly)](#disable-screen-from-blankn-mostly)
* [AUTO LOGIN (no X)](#auto-login-no-x)
* [no raspberry pi (penguin) logo on boot:](#no-raspberry-pi-penguin-logo-on-boot)
* [no startx](#no-startx)
* [AUTO START X (LXDE)](#auto-start-x-lxde)
* [SCREEN BLANK FIX](#screen-blank-fix)
* [autohide mouse-pointer](#autohide-mouse-pointer)
* [JAVA](#java)
* [BUILD PD-EXTENDED!](#build-pd-extended)
* [BACKUP / CLONE](#backup--clone)
* [SCANNER](#scanner)
* [nodejs](#nodejs)
* [PURE DATA INSTALL FROM GIT](#pure-data-install-from-git)
* [FULL SCREEN LXTERMINAL](#full-screen-lxterminal)
* [TWO NETWORK INTERFACES?](#two-network-interfaces)
* [flick'r upload](#flickr-upload)
* [QUIET SCREEN](#quiet-screen)
* [OSC](#osc)
* [TOUCH SCREEN](#touch-screen)
* [Framebuffer mirroring](#framebuffer-mirroring)
* [2-line LCD](#2-line-lcd)
* [pi printer](#pi-printer)
* [ssh notes](#ssh-notes)


# mount ext4 fs on OSX
```
brew cask install osxfuse
brew install ext4fuse
sudo mkdir /Volumes/rpi
sudo ext4fuse /dev/disk2s2 /Volumes/rpi -o allow_other
#NOTE: use disk utility app to figure out the SD card path. disk utility will show something like "device: disk4s1" so you would translate that into /dev/disk4s2... 
sudo umount /Volumes/rpi
```

# initialize a disk

note: use `sudo parted -l` or `lsblk` to figure out what device you want (/dev/sda in the example below). 

```
sudo parted /dev/sda mklabel gpt
sudo parted -a opt /dev/sda mkpart primary ext4 0% 100%
sudo mkfs.ext4 -L datapartition /dev/sda1
sudo mkdir -p /mnt/data
```

change/add label: `sudo e2label /dev/sda1 newlabel`

mount once: `sudo mount /dev/sda1 /mnt/data`

to mount on boot, edit /etc/fstab & add:  
`LABEL=newlabel /mnt/data ext4 defaults,nofail 0 2`

note: nofail to skip mount on boot if device is not available (e.g. usb disk not plugged in)

# use .local domain (ex: raspberrypi.local)

`sudo apt-get install avahi-daemon`

# AUTO X STARTZ

```
sudo ln -s /home/pi/dmx_ws/dmx_ws.sh /etc/init.d/dmx_ws.sh
sudo update-rc.d dmx_ws.sh defaults
```

-or-

`sudo vim /etc/xdg/lxsession/LXDE/autostart`

-for newer debianz-

`vim ~/.config/lxsession/LXDE-pi/autostart `

# disable screen from blank'n (mostly)

```
@xset s off
@xset -dpms
@xset s noblank

@midori -e Fullscreen -a http://micronemez.com
```

# AUTO LOGIN (no X)

`sudo vi /etc/inittab`

# no raspberry pi (penguin) logo on boot:

`echo "logo.nologo" >> /boot/cmdline.txt`

# no startx 

comment out this:  
`#1:2345:respawn:/sbin/getty 115200 tty1`  
and change to this:  
`1:2345:respawn:/bin/login -f pi tty1 </dev/tty1 >/dev/tty1 2>&1`

# AUTO START X (LXDE)

```
sudo vim /etc/rc.local

# add this above exit 0
su -l pi -c startx
```

# SCREEN BLANK FIX

see: http://www.raspberrypi.org/phpBB3/viewtopic.php?f=66&t=18200&p=180271#p180271


`sudo vim /etc/kbd/config`
```
BLANK_TIME=0 
POWERDOWN_TIME=0
```

# autohide mouse-pointer

`sudo apt-get intall unclutter`

# JAVA

`sudo apt-get install librxtx-java openjdk-6-jdk`


# BUILD PD-EXTENDED!

see: http://log.liminastudio.com/writing/tutorials/how-to-build-pd-extended-on-the-raspberry-pi 

```
sudo apt-get install rsync autoconf libfftw3-dev liblua5.1-0-dev swig libvorbis-dev ladspa-sdk libspeex-dev libmp3lame-dev lua5.1

#rsync -av --delete rsync://128.238.56.50/distros/pd-extended/ pd-extended/

wget http://blinky.at.or.at:8888/auto-build/2012-10-02/Pd-extended_0.43.3~20121002-source.tar.bz2
wget http://blinky.at.or.at:8888/auto-build/2012-10-02/Pd-extended_0.43.3~20121002-source.debian.tar.bz2
wget http://apt.puredata.info/auto-build/latest/Pd-extended_0.44.0~20130404-source.tar.bz2
wget http://apt.puredata.info/auto-build/latest/Pd-extended_0.44.0~20130404-source.debian.tar.bz2

sudo apt-get build-dep puredata gem pdp

sudo chmod 4755 /usr/bin/pd-extended

# WI-FI start command (like if you do not have X autostart!) that could be put in ~/.profile
sudo wpa_supplicant -Dwext -iwlan0 -c/etc/wpa_supplicant/wpa_supplicant.conf -d	

/sbin/wpa_supplicant -s -B -P /var/run/wpa_supplicant.wlan0.pid -i wlan0 -W -D nl80211,wext -c /etc/wpa_supplicant/wpa_supplicant.conf
```


# BACKUP / CLONE 

see: http://raspberrypi.stackexchange.com/questions/311/how-do-i-backup-my-raspberry-pi

`sudo dd if=/dev/rdisk2 bs=1m | gzip > /Users/edward/Desktop/DUMP/raspi-clone00/rpi01_base_config.gz`

restore, note use of sudo

`gzip -dc /Users/edward/Desktop/DUMP/raspi-clone00/rpi01_base_config.gz | sudo dd of=/dev/rdisk2 bs=1m`

misc:

```
#backup
##dd if=/dev/disk2 | gzip > /Users/edward/Desktop/DUMP/raspii-clone00/rpii00.gz
#restore
##gzip -dc /Users/edward/Desktop/DUMP/raspii-clone00/rpii00.gz | dd of=/dev/disk2
#whee: http://superuser.com/questions/421770/dd-performance-on-mac-os-x-vs-linux
# 1M==linux & 1m==mac
# 7948206080 bytes transferred in 1623.679502 secs (4895182 bytes/sec) vs(old): 8068792320 bytes transferred in 8753.619243 secs (921766 bytes/sec)
```

# SCANNER


For plustek scanners  
Some plustek scanners (noticeably Canoscan ones), require a lock directory.   
Make sure that /var/lock/sane directory exists, that its permissions are 660, and that it is owned by <user>:scanner.   
If the directory permissions are wrong, only root will be able to use the scanner.   
Seems (at least on x86-64) that some programs using libusb (noticeably xsane and kooka)   
need scanner group rw permissions also for accessing /proc/bus/usb to work for a normal user.  

`sudo apt-get install sane imagemagick scanbuttond sane-utils lockfile-progs netpbm festival toilet`

```
#lockup?!
# sudo scanimage --format tiff --mode Color --resolution 72 -x 200 -y 200 > /home/pi/scan/out02.tiff 
# sudo scanimage --mode Color --resolution 75 > /home/pi/scan/out02.tif 
# -x 215 -y 190 –resolution 600
# scanimage -d "plustek:libusb:001:005" –format=tiff > test.tiff
# scanimage -d "plustek:libusb:001:005" -–format tiff > test.tiff
# scanimage -–format tiff > test.tiff
# scanimage --format tiff --mode Color --resolution 72 > test2.tiff 
# scanimage --help -d plustek:libusb:001:005
# -x 0..215mm [103]
#        Width of scan-area.
#    -y 0..297mm [76.21]
#<D-h><D-h>        Height of scan-area.
# sudo scanimage -–format=pnm > test.pnm
#ZOMG!
#install -d -m 775 -g pi /run/lock/sane
#install -d -m 775 -g pi /var/lock/sane


#WORKZ-ish
scanimage --format=tiff -x 215 -y 297 > test.tiff


pi@raspberrypi ~ $ sudo vim /etc/sane.d/dll.conf
pi@raspberrypi ~ $ sudo vim /etc/sane.d/dll.d/libsane-extras


#http://robsworldoftech.blogspot.com/2010/09/canon-canoscan-lide-100-on-ubuntu-or.html

echo "about to install libusb-dev, build-essential, and libsane-dev"
read CONTINUE

apt-get install libusb-dev build-essential libsane-dev

echo "git clone git://git.debian.org/sane/sane-backends.git"
echo "./configure --prefix=/usr --sysconfdir=/etc --localstatedir=/var"
echo "copy the files backend/dll.conf and backend/genesys.conf into /etc/sane.d"
You'll be able to scan, but only as root. To fix this, you'll need to add the following two lines to your /lib/udev/rules.d/40-libsane.rules file, which you need to edit as root.
# Canon CanoScan LiDE 100
ATTRS{idVendor}=="04a9", ATTRS{idProduct}=="1904", ENV{libsane_matched}="yes"
Reboot. Everything should work.
```


# nodejs

```
sudo apt-get install nodejs npm
```

-or- for more recent versions, download the linux-armv6l binaries:

```
wget https://nodejs.org/dist/v8.11.1/node-v8.11.1-linux-armv6l.tar.xz
tar -xvf node-v8.11.1-linux-armv6l.tar.xz
cd node-v8.11.1-linux-armv6l
sudo cp -R * /usr/local/
```

also, i'm too lazy to change file permissions so often global (-g) npm installz don't work, getting errors like:

```
gyp WARN EACCES attempting to reinstall using temporary dev dir "/usr/local/lib/node_modules/hypercored/node_modules/utp-native/.node-gyp"
gyp WARN EACCES user "root" does not have permission to access the dev dir "/usr/local/lib/node_modules/hypercored/node_modules/utp-native/.node-gyp/8.11.1"
```

...to fix i guess the `--unsafe-perm` flag workz.

`sudo npm install --unsafe-perm -g somelib`

# dat

`sudo npm install --unsafe-perm -g dat`

see also: https://docs.datproject.org/server

```
npm install --unsafe-perm -g hypercored
touch feeds
```

add to feeds file something like:

```
dat://one-hash
two-hash
website.com/three-hash
```

run it forever

```
sudo npm install --unsafe-perm -g add-to-systemd lil-pids
mkdir ~/dat
echo "hypercored --cwd ~/dat" > ~/dat/services
sudo add-to-systemd dat-lil-pids $(which lil-pids) ~/dat/services ~/dat/pids
sudo systemctl start dat-lil-pids
```


# PURE DATA INSTALL FROM GIT

```
git clone git://pure-data.git.sourceforge.net/gitroot/pure-data/pure-data pure-data
cd pure-data/src
./autogen.sh
./configure CFLAGS="-mfpu=vfp -mfloat-abi=hard"
make
sudo make install

It takes around 20min to build, be patient.
you can start pd using the « pd » command

3. optimising the system for pd :

sudo leafpad /etc/security/limits.conf
or try nano if you don’t start an X server
add
* - rtprio 99
* - memlock 1000000000

Start pd and go to media > preference > startup
add the following flag in the startup flag field :
-rt -alsa -noadc -audiobuf 25

then apply and restart pd.

4. test

download analog synth emulation patch by Cyrille Henry here :
svn checkout https://pure-data.svn.sourceforge.net/svnroot/pure-data/trunk/externals/nusmuk/nusmuk-audio/ ~/nusmuk-audio
cd ~/nusmuk-audio
make
cd examples
pd analog_synth_emulation.pd

…speaker npm
apt-get install xdg-utils build-essential libasound-dev 
```

# FULL SCREEN LXTERMINAL

```
~$ vim ~/.config/openbox/lxde-rc.xml 
#add this node before </applications>
  <application name="lxterminal" class="Lxterminal">
   <decor>no</decor>
   <position force="yes">
     <x>0</x>
     <y>0</y>
   </position>
   <desktop>1</desktop>
   <layer>normal</layer>
   <skip_pager>yes</skip_pager>
   <skip_taskbar>no</skip_taskbar>
   <maximized>true</maximized>
  </application>

~$ vim ~/.config/lxterminal/lxterminal.conf
#replace all with
[general]
fontname=Monospace 10
selchars=-A-Za-z0-9,./?%&#:_
scrollback=1000
bgcolor=#000000000000
bgalpha=65535
fgcolor=#ffff66660000
disallowbold=false
cursorblinks=true
cursorunderline=false
audiblebell=false
tabpos=top
hidescrollbar=true
hidemenubar=true
hideclosebutton=true
disablef10=false
disablealt=false

```

```
~$ sudo vim /etc/xdg/lxsession/LXDE/autostart
#add
@lxterminal -e /home/pi/timesync.sh
```

# TWO NETWORK INTERFACES?

`sudo route add default gw 192.168.1.1 wlan0`

# flick'r upload

`curl -s -F method=photos.upload -F api_key=asdfasdfasdfasdf -F auth_token=asdfasdfasdfasdf -F api_sig=adsfasdfasdfasdfasdf -F photo=myfile.jpg api.flickr.com/services/upload/`

```
sudo apt-get install libflickr-api-perl
wget http://search.cpan.org/CPAN/authors/id/C/CP/CPB/Flickr-Upload-1.32.tar.gz
tar -zxvf Flickr-Upload-1.32.tar.gz
cd flickr-upload
perl Makefile.PL
make
make test
sudo make install
flickr_upload --auth
```

# QUIET SCREEN

`echo ‘startup_message off’ > ~/.screenrc`

# OSC

```
sudo apt-get install python-liblo

test by opening a python shell:
>>> import liblo, sys                                                                                                          
>>> target = liblo.Address('192.168.1.2',9000)
>>> liblo.send(target, "/1/label1”, "foo”)
```



# TOUCH SCREEN  

```
sudo apt-get update
mkdir ~/touchscreen 
cd ~/touchscreen
wget http://adafruit-download.s3.amazonaws.com/libraspberrypi-bin-adafruit.deb
wget http://adafruit-download.s3.amazonaws.com/libraspberrypi-dev-adafruit.deb
wget http://adafruit-download.s3.amazonaws.com/libraspberrypi-doc-adafruit.deb
wget http://adafruit-download.s3.amazonaws.com/libraspberrypi0-adafruit.deb
wget http://adafruit-download.s3.amazonaws.com/raspberrypi-bootloader-adafruit-112613.deb
```

Advanced users! Want to beta test our new DMA-enabled kernel? Its even faster! Instead of the last wget item - grab the Feb 2014 kernel deb file with "wget http://adafruit-download.s3.amazonaws.com/raspberrypi-bootloader-adafruit-20140227-1.deb" You can always install this over the 11-26-13 version or go back and forth

`sudo dpkg -i -B *.deb`

If you have a version of Raspbian more recent than Sept. 2013, you'll need to turn off the accelerated X framebuffer here, run "sudo mv /usr/share/X11/xorg.conf.d/99-fbturbo.conf .” to remove the accelerated X buffer and save it in your home directory

```
sudo reboot

sudo modprobe spi-bcm2708
sudo modprobe fbtft_device name=adafruitts rotate=90
export FRAMEBUFFER=/dev/fb1 startx

sudo vim /etc/modules
  #add two lines
spi-bcm2708
fbtft_device

sudo vim /etc/modprobe.d/adafruit.conf
  #add the following line
 options fbtft_device name=adafruitts rotate=90 frequency=32000000

sudo reboot
sudo mkdir /etc/X11/xorg.conf.d
```

```
sudo vim /etc/X11/xorg.conf.d/99-calibration.conf

  #and enter in the following lines, then save.
Section "InputClass"
        Identifier      "calibration"
        MatchProduct    "stmpe-ts"
        Option  "Calibration"   "3800 200 200 3800"
        Option  "SwapAxes"      "1"
EndSection


  #You can now try to run X again with 
FRAMEBUFFER=/dev/fb1 startx
```

```
sudo vim ~/.profile
  #and adding 
export FRAMEBUFFER=/dev/fb1
```

### CALIBRATION

```
sudo vim /etc/udev/rules.d/95-stmpe.rules
  #add
SUBSYSTEM=="input", ATTRS{name}=="stmpe-ts", ENV{DEVNAME}=="*event*", SYMLINK+="input/touchscreen" 
```

### Remove and re-install the touchscreen with

```
sudo rmmod stmpe_ts; sudo modprobe stmpe_ts

ls -l /dev/input/touchscreen

sudo apt-get install evtest tslib libts-bin
```


### real-time monitor: 
`sudo evtest /dev/input/touchscreen`

### meat of the calibration 
~sudo TSLIB_FBDEVICE=/dev/fb1 TSLIB_TSDEVICE=/dev/input/touchscreen ts_calibrate~

### draw
~sudo TSLIB_FBDEVICE=/dev/fb1 TSLIB_TSDEVICE=/dev/input/touchscreen ts_test`

```
wget http://adafruit-download.s3.amazonaws.com/xinput-calibrator_0.7.5-1_armhf.deb
sudo dpkg -i -B xinput-calibrator_0.7.5-1_armhf.deb

sudo rm /etc/X11/xorg.conf.d/99-calibration.conf

FRAMEBUFFER=/dev/fb1 startx & DISPLAY=:0.0 xinput_calibrator
```

### play video

`mplayer -vo fbdev2:/dev/fb1 -x 240 -y 320 -framedrop file.mp4`


# Framebuffer mirroring 


By mirroring /dev/fb0 onto /dev/fb1, we can take advantage of the GPU for hardware accelrated video playback.
fbcp takes a snapshot of /dev/fb0, copies it to /dev/fb1 and waits 25ms before repeating.
Snapshotting takes ~10ms and with a 25ms delay it gives roughly 1000/(10+25) = 28fps
CPU usage: ~2%
Note: Snapshot and /dev/fb1 driver refresh is not syncronized.

### Install fbcp

```
sudo apt-get install cmake
git clone https://github.com/tasanakorn/rpi-fbcp
cd rpi-fbcp/
mkdir build
cd build/
cmake ..
make
sudo install fbcp /usr/local/bin/fbcp
```

### Load drivers and fbcp

```
sudo modprobe fbtft dma
sudo modprobe fbtft_device name=tinylcd35 rotate=90 speed=48000000 fps=50
```

### Start fb copying process in the background

`fbcp &`

### Play video on /dev/fb0, which will also show up on /dev/fb1

`omxplayer test_480_320.mpg`

### Stop framebuffer copy

```
killall fbcp
syslog output
```

`$ tail /var/log/messages`

Dec 15 17:38:07 raspberrypi fbcp[4836]: Primary display is 720 x 480
Dec 15 17:38:07 raspberrypi fbcp[4836]: Second display is 480 x 320 16bps

see: http://www.stuffaboutcode.com/2012/06/raspberry-pi-run-program-at-start-up.html


# 2-line LCD  

```
sudo apt-get install python-smbus i2c-tools

sudo vim /etc/modules
# add
i2c-bcm2708 
i2c-dev

#reboot -or-
sudo modprobe i2c-bcm2708
sudo modprobe i2c-dev

#python toolz:
git clone https://github.com/adafruit/Adafruit-Raspberry-Pi-Python-Code.git
cd Adafruit-Raspberry-Pi-Python-Code
cd Adafruit_CharLCDPlate

sudo apt-get install python-dev python-rpi.gpio
```

# pi printer  

```
lpadmin -p txt -v usb://Unknown/Printer -P
usermod -a -G lpadmin pi
ssh pi@receipt.local -T -L 3631:localhost:631
 ```

 # ssh notes

disable password logins (add your ssh pubkey to ~/.ssh/authorized_keys) add `PasswordAuthentication no` to `/etc/ssh/sshd_config` then `sudo /etc/init.d/ssh reload`
