# Setup Pizzabox

## Materials:

- Raspberry Pi 3B+ with [Pimoroni On/Off SHIM](https://shop.pimoroni.com/products/onoff-shim)

- Raspberry Pi OS Lite 2021-05-07

- *(later)* Raspberry Pi Pico, connected via UART and power

- USB Soundcard

## Resources:

- [official raspberry pi](https://www.raspberrypi.org/documentation/usage/gpio/)

- [interactive pinout documentation](https://pinout.xyz/pinout/onoff_shim#)

## Initial Configuration:

### Before First Boot

- Enable ssh:

        cd <media/user>/boot
        touch ssh
        
- Set fixed fallback ip on eth0:

        cd <media/user>/rootfs/etc
        sudo nano dhcpcd.conf
        
  Uncomment the following lines:
  
        profile static_eth0
        static ip_address=10.10.0.23/24
        
        interface eth0
        fallback static_eth0

### First Boot

- Hook up pi to configuration system via ethernet, screen is optional. Insert SD-card, plug in OnOff SHIM

- Connect via ssh:

        ssh pi@10.10.0.23

- Use raspi-config tool to

  * *(optional)* set up WiFi and connect (for updates, ...)

  * enable camera
  
  * enable serial

  * change password to `pizzabox`
  
  Apply changes and reboot.
  
### Install Software

- Update system

        sudo apt update
        sudo apt full-upgrade

- Install python libraries:

        sudo apt install git python3-picamera python3-gpiozero python3-pip python3-setuptools python3-serial
        
- Update NumPy:

        pip3 install numpy --upgrade
        
  To fix bug with NumPy dependencies on Raspberry Pi install libatlas [source](https://numpy.org/devdocs/user/troubleshooting-importerror.html):
  
        sudo apt-get install libatlas-base-dev
        
- Install dependencies for python `sounddevice` library:

        sudo apt install libportaudio2
        
- Install `gpac` for the video convertion tool *MP4Box*:

        sudo apt install gpac
        
- Install `pydub` python library for mp3 support:

        pip3 install pydub
        
#### Install pygame for sound playback

- Install latest version of pygame using pip:

        pip3 install pygame
        
- Additional library is needed

        sudo apt install libsdl2-mixer-2.0-0
        
### Install Soundcard and Make Default

- Set external soundcard as default [how-to](https://www.raspberrypi-spy.co.uk/2019/06/using-a-usb-audio-device-with-the-raspberry-pi/)

- When hotplugged, the sound card may not be activated. If true, reboot.

- To see if the sound card was registered, run

        aplay -l
        
  Produces output similar to
  
        **** List of PLAYBACK Hardware Devices ****
        card 0: b1 [bcm2835 HDMI 1], device 0: bcm2835 HDMI 1 [bcm2835 HDMI 1]
          Subdevices: 4/4
          Subdevice #0: subdevice #0
          Subdevice #1: subdevice #1
          Subdevice #2: subdevice #2
          Subdevice #3: subdevice #3
        card 1: Headphones [bcm2835 Headphones], device 0: bcm2835 Headphones [bcm2835 Headphones]
          Subdevices: 4/4
          Subdevice #0: subdevice #0
          Subdevice #1: subdevice #1
          Subdevice #2: subdevice #2
          Subdevice #3: subdevice #3
        card 2: Device [USB PnP Sound Device], device 0: USB Audio [USB Audio]
          Subdevices: 1/1
          Subdevice #0: subdevice #0
          
  `card 2` is the usb sound card.

- Edit the file `/usr/share/alsa/alsa.conf`:

  Change the following lines (change the `0` to `2`)
  
        defaults.ctl.card 2
        defaults.pcm.card 2
        
- Test the speakers:

        speaker-test -c2
        
  Should play white noise on the left and right channel.
  
- Test the mic:

        arecord -d 10 -f cd -t wav test.wav
        aplay test.wav

- To set gain levels etc. run:

        alsamixer
        
## Set Up Camera

- Plug in camera with ribbon cable (requires reboot)

- Test camera (with screen plugged in):
  
        raspistill -t 1 -o test.jpg
        
  Download test.jpg using sftp. On macos use filezilla for sftp.
  
## Set Up OnOff Shim:

- Use the one-line installer:

        curl https://get.pimoroni.com/onoffshim | bash
        
## Python Development on Raspi (pizzabox-main)

- clone github repository `https://git.theater.digital/johannes.payr/pizzabox-main.git`

- *(For now)* Use Pycharm. Set up remote python interpreter on raspberry pi:

        SSH Server: 10.10.0.23
        User Name:  pi
        Path:       /usr/bin/python3
        
- install missing packages: `'click', 'sounddevice', 'soundfile', 'scipy'`
        


