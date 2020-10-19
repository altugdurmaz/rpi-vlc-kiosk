# rpi-vlc-kiosk 

1- play inline commands urls videos

2- play downloaded videos systemctl job

### Installing the OS

Install a version of Raspbian Lite (Minimal image based on Debian Buster). You can head over to : https://www.raspberrypi.org/downloads/raspbian/ and follow the instructions.

Once the image is downloaded, you can burn it to your SD card with tools like Etcher (https://www.balena.io/etcher/)

1- You need to connect the pi to an external monitor and keyboard during the setup phase.   
2- insert your micro sd card to RPI and plug-in power cable

### Connect your raspberry pi

First login
```sh
user: pi
password: raspberry
```

### Configuration

```sh
sudo raspi-config
```

- change pi password

- change hostname

- set wifi ssid and pass

- set timezone

- inferface opt -> SSH enable (you need to set a new password to open)

- Boot Options -> Console Autologin

- finish -> reboot

### Updates
Applying latest versions
```sh
sudo apt-get update && sudo apt-get upgrade -y
reboot
```
### Installing VLC Player
```sh
sudo apt-get install vlc
```
## Play inline commands urls 
Download the video you want to play on your booth. Since we will be using vlc (https://www.videolan.org/vlc/), your video should have a format that is supported by vlc.

Before we start, let's know the options that will work for us
important: to use VLC from the command line without the default interface, then you can do so by swapping out the `vlc` command with `cvlc`.
```sh
--fullscreen: full screen video
--loop, -L: loop videos
--no-osd: hide video title
--no-audio: muted video
--no-mouse-events or --no-cursor: disable cursor
.
.
.
```
You can to reach more: https://wiki.videolan.org/Documentation:Command_line/#Receiving_a_network_stream

### Let's try what we learned

in terminal:

```sh
cvlc --fullscreen --loop --no-osd --no-audio http://www.example.org/your_file.mpg http://www.example.org/your_file2.mpg
```

You should be seeing videos on the screen you are connected to.

We are done it !

## Play downloaded videos

Download the video you want to play on your booth. Since we will be using vlc (https://www.videolan.org/vlc/), your video should have a format that is supported by vlc.
Open up a terminal, and you can copy the video to your Pi with the following command :
`scp path_to_your_video/your_video.mp4 pi@raspberrypi.local:~`
This will copy your video to the home directory of your Raspberry Pi.

### Create a background job that will start playing your video when booting your Pi

We are going to create a background job that your operating system will launch at each boot of your Pi. To do so, we are creating a systemd unit file that will have this content :
`
[Unit]
Description=rpiKiosk

[Service]
User=pi
Environment="DISPLAY=:0"
ExecStart=/usr/bin/vlc -Rf path_to_your_video.mp4
WorkingDirectory=/home/pi
Restart=always

[Install]
WantedBy=multi-user.target
`
We created a systemd service that we call `rpiKiosk`. 

- We add a `User` field as VLC is not supposed to be started as the root user. 
- We add an environment variable called `DISPLAY` so that the video plays on the HDMI output of the Pi. 
- We use the `-Rf` flag in the `vlc` command so that the video plays in full screen and on repeat. 

Connect to your raspberry Pi over ssh : 
```
ssh pi@raspberrypi.local
```

Use your favorite text editor to create the unit file : 
```
sudo nano /etc/systemd/system/rpiKiosk.service
```

Copy/Paste the unit file we described earlier. Enable the service to auto-start on boot. 

```
sudo systemctl enable rpiKiosk.service 
```

To start the service : 
```
sudo systemctl start rpiKiosk.service 
```

To view the latest logs, use `sudo journalctl -xfu rpiKiosk.service`

To view the current service status, use `sudo systemctl status rpiKiosk.service`

We are done it !
