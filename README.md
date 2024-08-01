# PiKVM-webcam
Simple guide for setting up webcam system on your PiKVM system. Tested on PiKVM V4 Plus. Useful for setting up BIOS settings on laptops, or completely reinstalling the OS. Keyboard input function of kvmd works even if the HDMI input is not connected.

# Prepare the PiKVM system

1. `pikvm-update` - update the system
1. `rw` - enable writing to filesystem

# Download the mediamtx app 

Copy download link for latest release of mediamtx (linux_armv7), you can find the download link here: https://github.com/bluenviron/mediamtx/releases

1. `wget <download link>`
1. `tar -xvf <downloaded archive>`
1. `mv mediamtx /usr/local/bin/`
1. `mv mediamtx.yml /usr/local/etc/`

# Create systemd service

`nano /etc/systemd/system/mediamtx.service`

```
[Unit]
Wants=network.target
[Service]
ExecStart=/usr/local/bin/mediamtx /usr/local/etc/mediamtx.yml
[Install]
WantedBy=multi-user.target
```

# Edit mediamtx config file:

`nano /usr/local/etc/mediamtx.yml`

At the end of file modify the "paths" section as:

```
paths:
  cam:
    runOnDemand: ffmpeg -f v4l2 -input_format mjpeg -s 1920x1080 -r 10 -i /dev/webcam -c:v h264_v4l2m2m -pix_fmt yuv420p -b:v 6M -f rtsp rtsp://localhost:$RTSP_PORT/$MTX_PATH
    runOnDemandRestart: yes
  cam-lq:
    runOnDemand: ffmpeg -f v4l2 -input_format mjpeg -s 1280x720 -r 5 -i /dev/webcam -c:v h264_v4l2m2m -pix_fmt yuv420p -b:v 2M -f rtsp rtsp://localhost:$RTSP_PORT/$MTX_PATH
    runOnDemandRestart: yes
  cam-hq:
    runOnDemand: ffmpeg -f v4l2 -input_format mjpeg -s 1920x1080 -r 20 -i /dev/webcam -c:v h264_v4l2m2m -pix_fmt yuv420p -b:v 20M -f rtsp rtsp://localhost:$RTSP_PORT/$MTX_PATH
    runOnDemandRestart: yes
  all_others:
```

# Create udev rule

This is to mount the webcam consistently to a single /dev mapping point. You can find name and index attributes for your exact webcam using `udevadm info -a -p  $(udevadm info -q path -n /dev/videoX)`, where `/dev/videoX` is current mapping of your webcam.

`nano /etc/udev/rules.d/99-webcam.rules`

```
SUBSYSTEM=="video4linux", ATTR{index}=="0", ATTR{name}=="Logitech Webcam C925e", SYMLINK+="webcam"
```

# Enable service and reboot:

1. `sudo systemctl daemon-reload`
1. `sudo systemctl enable mediamtx`
1. `reboot`

# Camera web access:

- `<IP>:8889/cam/` = normal quality (1920x1080, 10fps, 6Mbit max, ~3Mbit avg)
- `<IP>:8889/cam-lq/` = low quality (1280x720, 5fps, 2Mbit max, ~500kbit avg)
- `<IP>:8889/cam-hq/` = high quality (1920x1080, 20fps, 20Mbit max ~10Mbit avg)

Stream quality switch takes ~20 seconds. Multiple streams are possible only in single quality. If webcam USB is disconncted/reconnected just refreshing the stream page should get it running again.
