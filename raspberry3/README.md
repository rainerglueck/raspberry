# Raspberry 3

## Setup 5.5'' Amoled Display
1. connect physically
2. start RPi - Display shows not a clear picture, it needs to be configured
3. Follow the instruction from here https://www.waveshare.com/wiki/5.5inch_HDMI_AMOLED connect via SSH, appended to /boot/config.txt
```
max_framebuffer_height=1920
max_usb_current=1
config_hdmi_boost=10
hdmi_group=2
hdmi_force_hotplug=1
hdmi_mode=87
hdmi_timings=1080 1 26 4 50 1920 1 8 2 6 0 0 0 60 0 135580000 3
dtoverlay=vc4-fkms-v3d
```
4. restart - OK

## Network Setup Moosstrasse 11
<li>eth0: 192.168.1.5
<li>wlan: 192.168.1.173