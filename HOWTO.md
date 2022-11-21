# Setup Pizzabox

## Materials:

- Raspberry Pi 3B+ or 2B with [Pimoroni On/Off SHIM](https://shop.pimoroni.com/products/onoff-shim)

- Raspberry Pi OS Lite 2022-09-22 bullseye-armhf-lite

- Teensy 4.1 with Arduino framework

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
        
  **Raspberry 3B+ only** Uncomment/edit the following lines:
  
        profile static_eth0
        static ip_address=10.10.0.23/24
        
        interface eth0
        fallback static_eth0
  
  **Raspberry 2B only** Uncomment/edit the following lines:
  
        interface eth0
        static ip_address=10.42.0.23/24
        static routers=10.42.0.1

        
- **Since April 2022, there is no default user/password!** To set up headless Pis, it is now necessary to set up a `userconf.txt` file on the `boot` partition of the sd card.
  See the [manual](https://www.raspberrypi.com/documentation/computers/configuration.html#configuring-a-user)
  
  To set up a user named `pi` with password `pizzabox`, use this line in the config file:
  
        pi:$6$oitbxzwpQ7h3hEOt$VRT9QCd.vvdJqyyjWsb3E7XnbGEeshqnovi8JFHvBf4oMj8mMCSuHXfQ8.x4BeTT3L.5w7eBFmRocuuxezcRD0
        
- **For Raspberry Pi 2B** Share internet over ethernet. See [here](https://www.tecmint.com/share-internet-in-linux/):

  * Use wired networking profile with option "IPV4->shared to other computers"
  
  * Default IP is 10.42.0.1
  



### First Boot

- Hook up pi to configuration system via ethernet. Insert SD-card, plug in OnOff SHIM

- Connect via ssh:

        ssh pi@10.10.0.23

- Use raspi-config tool to

  * **Raspberry 3/3B+ only** set up WiFi and connect (for updates, ...)

  * **With Raspberry Pi OS > 2022** enable legacy camera
  
  * enable serial:
  
        Disable Login shell on serial
        Enable serial hardware

  * **Not necessary with Raspberry Pi OS > 2022** change password to `pizzabox`
  
  Apply changes and reboot.
  
### Install Software

- Update system

        sudo apt update
        sudo apt full-upgrade

- Install python libraries:

        sudo apt install git python3-picamera python3-gpiozero python3-pip python3-setuptools python3-serial
        
  **Caution:** *The "old" `picamera` library is now considered "legacy" and will be replaced in future versions with `picamera2`.* However this requires changes to the software.
        
- Update NumPy:

        pip3 install numpy --upgrade
        
  To fix bug with NumPy dependencies on Raspberry Pi install libatlas [source](https://numpy.org/devdocs/user/troubleshooting-importerror.html):
  
        sudo apt-get install libatlas-base-dev
        
- Install dependencies for python `sounddevice` library:

        sudo apt install libportaudio2
        
- Install `ffmpeg` for video conversion

        sudo apt install ffmpeg
        
- Install `pydub` python library for mp3 support:

        pip3 install pydub
        
- Install `scipy` for Raspberry Pi:

        sudo apt install python3-scipy
        
#### Install pygame for sound playback

- Install latest version of pygame using pip:

        pip3 install pygame
        
- Additional library is needed

        sudo apt install libsdl2-mixer-2.0-0
        sudo apt install pulseaudio
        
### Install Soundcard and Make Default

- Set external soundcard as default [how-to](https://www.raspberrypi-spy.co.uk/2019/06/using-a-usb-audio-device-with-the-raspberry-pi/)

- When hotplugged, the sound card may not be activated. If true, reboot.

- To see if the sound card was registered, run

        aplay -l
        
  Produces output similar to
  
        **** List of PLAYBACK Hardware Devices ****
        card 0: Headphones [bcm2835 Headphones], device 0: bcm2835 Headphones [bcm2835 Headphones]
          Subdevices: 8/8
          Subdevice #0: subdevice #0
          Subdevice #1: subdevice #1
          Subdevice #2: subdevice #2
          Subdevice #3: subdevice #3
          Subdevice #4: subdevice #4
          Subdevice #5: subdevice #5
          Subdevice #6: subdevice #6
          Subdevice #7: subdevice #7
        card 1: Device [USB PnP Sound Device], device 0: USB Audio [USB Audio]
          Subdevices: 1/1
          Subdevice #0: subdevice #0

          
  `card 1` is the usb sound card.

- Edit the file `/usr/share/alsa/alsa.conf`:

  Change the following lines (change the `0` to `1`)
  
        defaults.ctl.card 1
        defaults.pcm.card 1
        
- Test the speakers:

        speaker-test -c2
        
  Should play white noise on the left and right channel.
  
- Test the mic:

        arecord -d 10 -f cd -t wav test.wav
        aplay test.wav

- Setting microphone volume lower with softcontrol:
  
  see: https://superuser.com/questions/1542023/alsa-setting-microphone-levels
  
  * created file `~/.asoundrc`:
  
        pcm.!default
        {
            type asym
            playback.pcm
            {
                type plug
                slave.pcm "dmix"
                slave.rate 48000
            }
            capture.pcm
            {
                type plug
                slave.pcm "mic_control"
            }
        }

        pcm.mic_control {
            type            softvol
            slave {
                pcm         "hw:1,0"
            }
            control {
                name        "Softmute Capture Volume"
                card        1
            }
            max_dB 10.0
            min_dB -100.0
        }
        
  * softvol needs to be enabled by using the microphone (sic), see [alsa manual](https://alsa.opensrc.org/Softvol). 
    Use `arecord` to enable the control in `alsamixer`:
    
        arecord -d 10 -f cd -t wav test.wav
        
  * with `alsamixer` set:
  
        PCM gain:   60, [dB gain: -8.13, -8.13] 
        Mic gain:    0, [dB gain: 0.00]
        Softmute:   25, [dB gain: -25.61, -25.61]

- To set gain levels etc. run:

        alsamixer
        
## Set Up Camera

- Plug in camera with ribbon cable (requires reboot)

- **This step needs legacy camera installed** Test camera (with screen plugged in):
  
        raspistill -t 1 -o test.jpg
        
  Download test.jpg using sftp. On macos use filezilla for sftp.
  
## Set Up OnOff Shim:

- Use the one-line installer:

        curl https://get.pimoroni.com/onoffshim | bash
        
## Set Up usb stick automount

**After this step, the Pi will only boot with an inserted USB drive with the name PIZZAFILES. Proceed with caution**

- USB drive needs to be named `PIZZAFILES` and formated with vfat

- Create mountpoint:

        cd ~
        mkdir pizzafiles

- Add line to `/etc/fstab`:

        LABEL=PIZZAFILES /home/pi/pizzafiles vfat rw,relatime,exec,uid=pi,gid=pi 0 2
        
- Insert usb drive and check with

        sudo mount -a
        
  The drive should be mounted to `/home/pi/pizzafiles`. 

        
## Python Development on Raspi (pizzabox-main)

- Clone github repository `https://git.theater.digital/johannes.payr/pizzabox-main.git` to your computer

- To deploy development version, copy files to Pi from outside the project directory:

        scp -rpv pizzabox-main pi@10.10.0.23:/home/pi/pizzabox-main
        
- On the Pi, install in editing mode using pip:

        cd ~/pizzabox-main
        pip3 install -e .
        
- To test if everything works run `python3`, type:

        from pizzactrl.statemachine_test import *
        sm.run()
        exit()
        
- For work-in-progress updates, run (run in project root directory `pizzabox-main`)

        scp pizzactrl/*.py  pi@10.10.0.23:/home/pi/pizzabox-main/pizzactrl/
        
## Running as a Service

- See the [Documentation](https://docs.fedoraproject.org/en-US/quick-docs/understanding-and-administering-systemd/index.html) on systemd services

- On the Pi, copy the unit file to the right location:

        sudo cp pizza.service /etc/systemd/system/

- Reload the daemon and test that the service works:

        sudo systemctl daemon-reload
        sudo systemctl start pizza
        
- To enable the service on startup:

        sudo systemctl enable pizza

- While testing, disable the service:

        sudo systemctl disable pizza



