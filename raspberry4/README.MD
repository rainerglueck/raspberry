# Raspberry 4

## Setup 5.5'' Amoled Display
Amoled display was already configured prior starting this documention. I will start therefore at the point at which I found the PI and the display.

5.5inch AMOLED, 1920x1080 resolution
1280x720

* Pi4 started without Amoled Display, connected via VNC only, OK
* Mounted Amoled Display, startet Pi4, worked, Orientation Landscape

Screensaver is already installed  

`cat /boot/config.txt`
```
... last lines
# config for AMOLED, taken from waveshares.com
max_framebuffer_height=1920
max_usb_current=1
config_hdmi_boost=10
hdmi_group=2
hdmi_force_hotplug=1
hdmi_mode=87
hdmi_timings=1080 1 26 4 50 1920 1 8 2 6 0 0 0 60 0 135580000 3
#dtoverlay=vc4-fkms-v3d
display_rotate=1 #1: 90; 2: 180; 3: 270
``` 

## Kodi
[Internetradio](https://www.reichelt.de/magazin/how-to/kodi-internetradio/)
```
sudo apt-get update
sudo apt-get upgrade
sudo rebootsudo r
sudo apt-get install kodi
sudo reboot
```

Starting Kodi via VNC not working, disconnect VNC, start Kodi without VNC Connection also NOK (shows black screen for few seconds, than shows desktop again)

### Debugging Kodi
(kodi.wiki)[https://kodi.wiki/view/Log_file/Advanced]
```
pi@raspberrypi:~ $ cd .kodi/userdata/
pi@raspberrypi:~/.kodi/userdata $ ls
addon_data  Database  guisettings.xml  keymaps  library  playlists  profiles.xml  RssFeeds.xml  Savestates  Thumbnails
pi@raspberrypi:~/.kodi/userdata $ vi advancedsettings.xml
```
`<loglevel hide="false">0</loglevel>`

LOG files could be here
```
pi@raspberrypi:~/.kodi/temp $ pwd
/home/pi/.kodi/temp
pi@raspberrypi:~/.kodi/temp $ ls
archive_cache  kodi.log  kodi.old.log  temp
pi@raspberrypi:~/.kodi/temp $ ls -l
total 48
drwxr-xr-x 2 pi pi  4096 May  8 17:55 archive_cache
-rw-r--r-- 1 pi pi 19236 May  8 17:55 kodi.log
-rw-r--r-- 1 pi pi 19227 May  8 17:47 kodi.old.log
drwxr-xr-x 2 pi pi  4096 May  8 17:38 temp
pi@raspberrypi:~/.kodi/temp $
```
(forum.kodi.tv)[https://forum.kodi.tv/showthread.php?tid=348014]
modiiying `/boot/config.txt`

Display: portrait, Touch: landscape, Kodi: not starting up
```
#dtoverlay=vc4-fkms-v3d
#dtoverlay=vc4-fkms-v3d,cma-256
#display_rotate=1 #1: 90; 2: 180; 3: 270
```

Display: landscape, Touch: landscape, Kodi: not starting up
```
#dtoverlay=vc4-fkms-v3d
#dtoverlay=vc4-fkms-v3d,cma-256
display_rotate=1 #1: 90; 2: 180; 3: 270
```

Display: landscape, Touch: landscape, Kodi: starting up portrait, verzerrt
```
dtoverlay=vc4-fkms-v3d
#dtoverlay=vc4-fkms-v3d,cma-256
display_rotate=1 #1: 90; 2: 180; 3: 270
```

Display: portrait, Touch: portrait aber jetzt scheint horizontal mit vertikal vertauscht, Kodi: starting up portrait, verzerrt
```
dtoverlay=vc4-fkms-v3d
#dtoverlay=vc4-fkms-v3d,cma-256
display_rotate=1 #1: 90; 2: 180; 3: 270
```

Install lib to get touch rotating (https://www.waveshare.com/w/upload/b/be/5.5inch_HDMI_AMOLED_User_Manual_EN.pdf)
`sudo apt-get install xserver-xorg-input-libinput`
was already installed
`sudo vi /etc/X11/xorg.conf.d/40-libinput.conf`
removed `Option "CalibrationMatrix" "0 1 0 -1 0 1 0 0 1"`
now
```
Section "InputClass"
        Identifier "libinput touchscreen catchall"
        Option "CalibrationMatrix" "0 1 0 -1 0 1 0 0 1"
        MatchIsTouchscreen "on"
        MatchDevicePath "/dev/input/event*"
        Driver "libinput"
EndSection
```
`sudo reboot`
Display: portrait, Touch: portrait, Kodi: starting up portrait, verzerrt

### Kodi again
[Kodi Wiki](https://kodi.wiki/view/HOW-TO:Install_Kodi_on_Raspberry_Pi)

`/boot/config.txt`
No effect on KODI being stretched
```
# uncomment this if your display has a black border of unused pixels visible
# and your display can output without overscan
disable_overscan=1
```

No effect
```
# uncomment to force a console size. By default it will be display's size minus
# overscan.
framebuffer_width=1280
framebuffer_height=720
```

## Changing Amoled resolution
[STICKY: HOWTO: Create Custom HDMI modes](https://www.raspberrypi.org/forums/viewtopic.php?t=24679)
```
STICKY: HOWTO: Create Custom HDMI modes

Tue Dec 04, 2012 11:03 pm
We've had a steady stream of requests for supporting HDMI modes that aren't from the official CEA/DMT list of modes.

I've implemented CVT:
http://en.wikipedia.org/wiki/Coordinated_Video_Timings

You specify the new mode in config.txt with the following config string:

hdmi_cvt=<width> <height> <framerate> <aspect> <margins> <interlace> <rb>
width        width in pixels
height       height in pixels
framerate    framerate in Hz
aspect       aspect ratio 1=4:3, 2=14:9, 3=16:9, 4=5:4, 5=16:10, 6=15:9
margins      0=margins disabled, 1=margins enabled
interlace    0=progressive, 1=interlaced
rb           0=normal, 1=reduced blanking

The first three arguments are required. The remaining ones are optional. aspect defaults to 16:9 if not specified.
It should appear if you do

tvservice -m DMT

An example:

hdmi_cvt=1680 1050 60 5 0 0 1
``` 
#### Custom HDMI modes ####
latest "next" firmware has some changes to custom hdmi modes.
CVT now uses a single config string to set up. (First post edited)
You can also play with the raw hdmi timings:

The hdmi_timings parameters are: 
`hdmi_timings=<h_active_pixels> <h_sync_polarity <h_front_porch> <h_sync_pulse> <h_back_porch> <v_active_lines> <v_sync_polarity> <v_front_porch> <v_sync_pulse> <v_back_porch> <v_sync_offset_a> <v_sync_offset_b> <pixel_rep> <frame_rate> <interlaced> <pixel_freq> <aspect_ratio>`

```
hdmi_timings=1080 1 26 4 50 1920 1 8 2 6 0 0 0 60 0 135580000 3
#hdmi_timings=720 1 26 4 50 1280 1 8 2 6 0 0 0 60 0 135580000 3
```

```
HDMI_ASPECT_4_3 = 1
HDMI_ASPECT_14_9 = 2
HDMI_ASPECT_16_9 = 3
HDMI_ASPECT_5_4 = 4
HDMI_ASPECT_16_10 = 5
HDMI_ASPECT_15_9 = 6
HDMI_ASPECT_21_9 = 7
HDMI_ASPECT_64_27 = 8
```

Find out more about hardware (https://raspberrypi.stackexchange.com/questions/67234/raspberry-pi-custom-resolution-issues-for-12-3-1920x720-lcd)
`xrandr -q` in VNC
```
Screen 0: minimum 320 x 200, current 1080 x 1920, maximum 7680 x 7680
HDMI-1 connected primary 1080x1920+0+0 (normal left inverted right x axis y axis) 0mm x 0mm
   FIXED_MODE    60.37*+
```

```
If you happen to be trying this with a Raspberry Pi 4 using Balena to deploy a Docker container, try writing these settings to the config.txt file before first boot. Note the frequency is different to the answer above, 100980000 instead of 109244953.

hdmi_ignore_edid=0xa5000080
hdmi_group=2
hdmi_mode=87
disable_overscan=1
hdmi_timings=1920 0 88 44 148 720 0 4 5 36 0 0 0 60 0 100980000 1

I found that value by running xrandr -q on a stock Raspbian image. (Note Rainer: xrandr -q does not work in ssh 'can not open display')

Screen 0: minimum 320 x 200, current 720 x 1920, maximum 7680 x 7680
HDMI-1 connected primary 720x1920+0+0 left (normal left inverted right x axis y axis) 0mm x 0mm
   1920x1080     60.00  
   1024x768      60.00  
   800x600       60.32    56.25  
   848x480       60.00  
   640x480       59.94  
  1920x720 (0x42) 100.980MHz +HSync +VSync
        h: width  1920 start 2008 end 2052 total 2200 skew    0 clock  45.90KHz
        v: height  720 start  724 end  729 total  765           clock  60.00Hz

Other ways I found to set this without touching the config.txt:

$ xrandr --newmode "1920x720_60.00"  100.98  1920 2008 2052 2200  720 724 729 765  +HSync +Vsync
$ xrandr --addmode HDMI-1 "1920x720_60.00"
$ xrandr --output HDMI-1 --mode "1920x720_60.00"

or

$ vcgencmd hdmi_timings 1920 0 88 44 148 720 0 4 5 36 0 0 0 60 0 100980000 1
$ tvservice -e 'DMT 87
```