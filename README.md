<p align="center">
	<img src="https://ptpimg.me/399pk8.png" alt="misterjtc logo" width="450">
</p>
<h3 align="center">Raspberry Pi Home Security Camera<!-- Serve Confidently --></h3>
<p align="center">This is a custom 3D printed raspberry pi camera solution</p>

---

This project uses **open-source** software to deliver a cost effective home security solution with motion detection.

The pieces of software used in this project are highly flexible and can likely be adapted to your configuration although it may deviate from what I have outlined below.

# How to Configure the RaspberryPi
=============

## (OPTIONAL) Configure Headless Access

To allow remote access by default, we need to enable ssh. This can be done by the following:

In the boot folder of the sd card, create a new file with the following name:

```
ssh
```

If you are planning to use Wifi like me, then you will need to create a file with your Wifi credentials in the boot directory of the sd card. This can be done as follow:

Create a file called:

```
wpa_supplicant.conf
```

Place the following details in it:

```
country=CA
ctrl_interface=DIR=/var/run/wpa_supplicant GROUP=netdev
update_config=1
network={
        ssid="yournetworkssid"
        psk="younetworkpassword"
        key_mgmt=WPA-PSK
}
```

including the quotes!

## Update RaspberryPi Firmware

```
sudo apt-get install rpi-update
sudo rpi-update
```

## Update RaspberryPi Dependencies

```
sudo apt-get update
sudo apt-get upgrade
```

AND

Setup regular automatic updates, run the following commands:

Obtain superuser privlidges

```
sudo su
```

Edit the superuser crontab

```
crontab -e
```

Add the following lines to the file. This will run the commands at 1PM every Sunday

```
0 13 * * 7 apt update && apt upgrade -y
```

For more information see here: [Crontab Ref](https://www.raspberrypi.org/documentation/linux/usage/cron.md)

## Configure General RaspberryPi Settings

Once connected to the RaspberryPi, run the following command to open raspi-config

```
sudo raspi-config
```

Main things that need to be done:

1. Change your account password
2. Enable the camera

You can confirm that your camera is connected and enabled by running the following command:

```
v4l2-ctl --list-formats
```

You should see something like this:

```
ioctl: VIDIOC_ENUM_FMT
        Type: Video Capture

        [0]: 'YU12' (Planar YUV 4:2:0)
        [1]: 'YUYV' (YUYV 4:2:2)
        [2]: 'RGB3' (24-bit RGB 8-8-8)
        [3]: 'JPEG' (JFIF JPEG, compressed)
        [4]: 'H264' (H.264, compressed)
        [5]: 'MJPG' (Motion-JPEG, compressed)
        [6]: 'YVYU' (YVYU 4:2:2)
        [7]: 'VYUY' (VYUY 4:2:2)
        [8]: 'UYVY' (UYVY 4:2:2)
        [9]: 'NV12' (Y/CbCr 4:2:0)
        [10]: 'BGR3' (24-bit BGR 8-8-8)
        [11]: 'YV12' (Planar YVU 4:2:0)
        [12]: 'NV21' (Y/CrCb 4:2:0)
        [13]: 'BGR4' (32-bit BGRA/X 8-8-8-8)
```

## Install v4l2rtspserver

```
sudo apt-get install cmake liblog4cpp5-dev libv4l-dev git
```

```
git clone https://github.com/mpromonet/v4l2rtspserver.git
cd v4l2rtspserver/
cmake .
make
sudo make install
```

## Test v4l2rtspserver:

```
v4l2rtspserver -H 972 -W 1296 -F 15 -P 8554 /dev/video0
```

This command provides you with an image of 1296x972 with 15 fps

## View the stream:

The video stream will become available at the following address:

```
rtsp://<raspberry-pi-ip>:8554/unicast
```

You can view the stream by using the above link in VLC player. Media --> Open Network Stream ...

## Launch v4l2rtspserver on startup:

To start v4l2rtspserver on boot we need to modify the v4l2rtspserver.service with our stream configuration.

```
sudo nano /lib/systemd/system/v4l2rtspserver.service
```

We need to add our run command 

```
v4l2rtspserver -H 972 -W 1296 -F 15 -P 8554 /dev/video0
```

The v4l2rtspserver.service file should look like this:

```
[Unit]
Description=V4L2 RTSP server
After=network.target

[Service]
Type=simple
Restart=always
RestartSec=1
User=pi
ExecStart=/usr/local/bin/v4l2rtspserver -F15 -H 972 -W1296 -P 8554 /dev/video0
WorkingDirectory=/usr/local/share/v4l2rtspserver
StartLimitIntervalSec=0

[Install]
WantedBy=multi-user.target
```

## Activate v4l2rtspserver on boot:

```
sudo systemctl enable v4l2rtspserver
```

## Rotate camera:

If you need to rotate the camera output, you can use something like this:

```
v4l2-ctl --set-ctrl=rotate=180
```