# How to Install and Configure Motion on RaspberryPi
=============

## (OPTIONAL) Configure headless access

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

## Update RaspberryPi firmware

```
sudo apt-get install rpi-update
sudo rpi-update
```

## Update RaspberryPi dependencies

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

## Configure general RaspberryPi settings

Once connected to the RaspberryPi, run the following command to open raspi-config

'''
sudo raspi-config
'''

Main things that need to be done:

1. Change your account password
2. Enable the camera

## Install Motion

```
sudo apt-get install motion
```

## Install Raspberry Pi Camera Dependencies

In order to use to Raspberry Pi Camera after installing motion using apt-get. We need to load the camera module. This is done by typing the following command:

```
sudo modprobe bcm2835-v4l2
```

We can load this by default on Raspberry Pi start-up by modifying the modules file by doing the following:

```
sudo nano /etc/modules
```

#and add the following to the bottom of the file:

```
bcm2835-v4l2
```

## Make Necessary Changes to Motion.conf File

```
sudo nano /etc/motion/motion.conf
```

The notable modifications to this file are the following:

1. The daemon option is set to on
2. Set the stream_quality to 100
3. stream_localhost to off
4. webcontrol_localhost to off
5. Set the quality to 100
6. Set the width to 640 and the height to 480
7. Set pre_capture and post_capture to 2
8. Uncomment mmalcam_name vc.ril.camera if you are using the RaspberryPi camera
9. Change the target_dir parameters to where you want to save your videos/images. My examples uses /home/pi/PiCam
10. Set locate_motion_mode to on
11. Set locate_motion_style to redbox
12. Set text_changes to on

## Enable the motion daemon:

```
sudo nano /etc/default/motion
```

Set 'start_motion_daemon' to yes.

## Restart motion and test!

After we finish editing our configuration, we need to restart the motion service. Use the following command:

```
sudo service motion restart
```

Then use the command:

```
sudo motion
```

## (Optional) Mount a Remote SAMBA Share

This can be useful if you're storing large sized images for simply recording the non-stop. However, it requires that you have a computer or server that runs 24/7 and supports the NFS file sharing protocol. You can potentially use Samba, but I will not include a guide to do that within here.

You can testing mounting the share by running the following command on your RaspberryPi:

Using NFS:

```
sudo mount 192.168.86.2:/mnt/user/Pi_Cam/Camera1 /home/pi/PiCam
```

This will mount the folder mnt/user/PiCam/Camera1 from a server to the folder /home/pi/PiCam on your RaspberryPi.

You can verify this worked by placing a test file in mnt/user/PiCam/Camera1 and navigating to /home/pi/PiCam on the Pi

In order for this to work reliably, we'll want to do this whenever the Pi is restarted.

To do this, we modify the fstab file by doing the following

```
sudo nano /etc/fstab
```

and adding the following line to the bottom of the file

Using NFS:

```
192.168.86.2:/mnt/user/Pi_Cam/Camera1 /home/pi/PiCam nfs auto,noatime,nolock,bg,nfsvers=3,intr,tcp,actimeo=1800 0 0 

```

