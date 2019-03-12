## JCook's Installing and Configuring Motion on Raspberry Pi's Guide
=============

## Update Raspberry Firmware

```
sudo apt-get install rpi-update
sudo rpi-update
```

## Update Raspberry Pi Dependencies

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

#Edit the superuser crontab

```
crontab e
```

Add the following lines to the file. This will run the commands at 1PM every Sunday

```
0 13 * * 7 apt update && apt upgrade -y
```

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

bcm2835-v4l2

## Make Necessary Changes to Motion.conf File

See motion.conf

#The notable modifications to this file are the following:

XXXXXXXX

## (Optional) Mount a Remote SAMBA Share

This can be useful if you're storing large sized images for simply recording the non-stop.

You can testing mounting the share by running the following command on your RaspberryPi

```
sudo mount -t cifs -o user=<username>,password=<password> //192.168.86.2/Pi_Cam/Camera1 /home/pi/PiCam
```

#This will mount the folder /PiCam/Camera1 from a server to the folder /home/pi/PiCam on your RaspberryPi
#You can verify this worked by placing a test file in /PiCam/Camera1 and navigating to /home/pi/PiCam on the Pi

#In order for this to work reliably, we'll want to do this whenever the Pi is restarted.

#To do this, we modify the fstab file by doing the following

```
sudo nano /etc/fstab
```

#and adding the following line to the bottom of the file

```
//192.168.86.2/Pi_Cam/Camera1 /home/pi/PiCam cifs uid=0,credentials=/home/pi/.smbcredentials,x-systemd.automount,iocharset=utf8,noperm 0  0
```