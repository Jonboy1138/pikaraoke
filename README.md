# PiKaraoke

Karaoke song search and queueing system that uses Youtube videos as a source. For use with Raspberry Pi devices.

## Features

*  Web interface for multiple users to queue tracks
*  Searching song library via autocomplete
*  Adding new tracks from Youtube
*  Offline storage of video files
*  Pause/Skip/Restart and volume control
*  Now playing and Up Next display
*  Basic editing of downloaded file names
*  Queue editing

## Screenshots

https://imgur.com/a/wgBYeFb

## Supported Devices

This *should* work on all raspberry pi devices, but multi-core models recommended. I did most development on a Pi Zero W and did as much optimization as I could handle, so it will work. However, certain things like concurrent downloads and browsing big song libraries will suffer. All this runs excellently on a Pi 3.

## Setup

If you already have git, you can skip this. If not:

```
sudo apt-get update
sudo apt-get install git
```

Clone this repo. The rest of this guide assumes you install to /home/pi/pikaraoke:
```
cd ~
git clone https://github.com/vicwomg/pikaraoke.git
cd pikaraoke
```

Run the setup script:

```
./setup.sh
```

You will then probably need to reboot since this changes a boot setting (gpu_mem=128). This is to prevent certain videos from showing visual artifacts (green pixel distortion) 

```
sudo reboot
```

## Launch

cd to the pikaraoke directory and run:

`sudo python app.py`

Yes, you must run as sudo since PiKaraoke uses pygame to control the screen buffer.

The app should launch and show the PiKaraoke splash screen and a QR code and a URL. Using a device connected to the same wifi network as the Pi, scan this QR code or enter the URL into a browser. You are now connected! You can start exploring the UI and adding/queuing new songs directly from YouTube.

## Auto-start PiKaraoke

This is optional, but you may want to make your pi a dedicated karaoke device. If so, add the following to your /etc/rc.local file (paths and arguments may vary) to always start pikaraoke on reboot.

```
# start pikaraoke on startup
/usr/bin/python /home/pi/pikaraoke/app.py &
```

Or if you're like me and want some logging for aiding debugging, the following stores output at: /var/log/pikaraoke.log:

```
# start pikaraoke on startup / logging
/usr/bin/python /home/pi/pikaraoke/app.py >> /var/log/pikaraoke.log 2>&1 &
```

If you want to kill the pikaraoke process, you can do so from the PiKaraoke Web UI under: `Info > Quit to console`. Or you can ssh in and run `sudo killall python` or something similar.

Note that if your wifi/network is inactive pikaraoke will error out 10 seconds after being launched. This is to prevent the app from hijacking your ability to login to repair the connection. 

## Usage

Here is the full list of command line arguments:

```
usage: app.py [-h] [-p PORT] [-d DOWNLOAD_PATH] [-o OMXPLAYER_PATH]
              [-y YOUTUBEDL_PATH] [-v VOLUME] [-s SPLASH_DELAY] [-l LOG_LEVEL]
              [--hide-overlay] [--hide-ip] [--hide-splash-screen]

optional arguments:
  -h, --help            show this help message and exit
  -p PORT, --port PORT  Desired http port (default: 5000)
  -d DOWNLOAD_PATH, --download-path DOWNLOAD_PATH
                        Desired path for downloaded songs. (default:
                        /usr/lib/pikaraoke/songs)
  -o OMXPLAYER_PATH, --omxplayer-path OMXPLAYER_PATH
                        Path of omxplayer. (default: /usr/bin/omxplayer)
  -y YOUTUBEDL_PATH, --youtubedl-path YOUTUBEDL_PATH
                        Path of youtube-dl. (default: /usr/local/bin/youtube-
                        dl)
  -v VOLUME, --volume VOLUME
                        Initial player volume in millibels. Negative values
                        ok. (default: 0 , Note: 100 millibels = 1 decibel)
  -s SPLASH_DELAY, --splash-delay SPLASH_DELAY
                        Delay during splash screen between songs (in secs).
                        (default: 4 )
  -l LOG_LEVEL, --log-level LOG_LEVEL
                        Logging level int value (DEBUG: 10, INFO: 20, WARNING:
                        30, ERROR: 40, CRITICAL: 50). (default: 20 )
  --show-overlay        Show overlay in omxplayer with song title and IP.
                        (feature is broken on Pi 4 omxplayer 12/24/2019)
  --hide-ip             Hide IP address from the screen.
  --hide-splash-screen  Hide splash screen before/between songs.
  --alsa-fix            Add this if you are using a USB soundcard or Hifi
                        audio hat and cannot hear audio.
  --dual-screen         Output video to both HDMI ports (raspberry pi 4 only)
```

## Screen UI

Upon launch, the connected monitor/TV should show a splash screen with the IP of PiKaraoke along with a QR code.

Make sure you are connected to the same network/wifi. You can then enter the shown IP or scan the QR code on your smartphone/tablet/computer to open it in a browser. From there you should see the PiKaraoke web interface. It is hopefully pretty self-explanitory, but if you really need some help:

## Web UI

### Home (Microphone Icon)

*  View Now Playing and Next tracks
*  Access controls to repeat, pause, skip and control volume

### Queue

*  Edit the queue/playlist order (up and down arrow icons)
*  Delete from queue ( x icon )
*  Add random songs to the queue
*  Clear the queue

### Songs

*  Add songs to the queue by searching current library on local storage (likely empty at first), search is executed autocomplete-style
*  Add new songs from the internet by using the second search box
*  Click browse to view the full library. From here you can edit files in the library (rename/delete).

### Info

*  Shows the IP and QR code to share with others
*  Shows CPU / Memory / Disk Use stats
*  Allows user to quit to console, shut down, or reboot system. Always shut down from here before you pull the plug on pikaraoke!

## Troubleshooting

### Songs aren't downloading!

Make sure youtube-dl is up to date, old versions have higher failure rates due to security changes in Youtube. You can see your current version installed by navigating to `Info > System Info > Youtube-dl version`. The version number is usually the date it was released. If this is older than a few months, chances are it will need an update.

You can update youtube-dl directly from the web UI. Go to `Info > Update Youtube-dl`

Or, from the CLI:
`sudo pip install --upgrade youtube_dl`

### Downloads are slow!

youtube-dl is very CPU intensive, especially for single-core devices like the pi models zero and less-than 2. The more simultaneous downloads there are, the longer they will take. Try to limit it to 1-2 at a time. Pi 3 can handle quite a bit more.

### I brought my pikaraoke to a friend's house and it can't connect to their network. How do I change wifi connection without ssh?

These are my preferred ways to do it, but they might require either a USB keyboard or a computer with an SD Card reader.

* _Completely Headless_: I can highly recommend this package: https://github.com/jasbur/RaspiWiFi . Install it according to the directions and it will detect when there is no network connection and act as a Wifi AP allowing you to configure the wifi connection from your smartphone, similar to a Chromecast initial setup. You can even wire up a button to GPIO18 and 3.3V to have a manual wifi reset button. This, along with auto-launch in rc.local makes PiKaraoke a standalone appliance!
* _USB Keyboard_: plug in a USB keyboard to the pi. After it boots up, log in and run "sudo raspi-config" and configure wifi through the Network Options section. If the desktop UI is installed, you can also run "startx" and configure wifi from the Raspbian GUI. You can also manually edit /etc/wpa_supplicant/wpa_supplicant.conf as desribed below.
* _SD Card Reader_: Remove the pi's SD card and open it on a computer with an SD card reader. It should mount as a disk drive. On the BOOT partition, add a plaintext file named "wpa_supplicant.conf" and put the following in it:

```
ctrl_interface=DIR=/var/run/wpa_supplicant GROUP=netdev
update_config=1
country=<Your 2-letter country code, ex. US>
network={
  ssid="<the wifi ap ssid name>"
  psk="<the wifi password>"
  key_mgmt=WPA-PSK
}
```

Add the SD card back to the pi and start it up. On boot, Raspbian should automatically add the wpa_supplicant.conf file to the correct location and connect to wifi.

### Can I run PiKaraoke without a wifi/network connection?

Actually, yes! But you can only access your existing library and won't be able to download new songs, obviously. 

If you run your pi as a wifi access point, your browser can connect to that access point, and it should work. See: https://www.raspberrypi.org/documentation/configuration/wireless/access-point.md

Or an even easier approach, if you install this: https://github.com/jasbur/RaspiWiFi (used for configuring wifi connections headless, see above). While it's in AP mode, you can connect to the pi as an AP and connect directly to it at http://10.0.0.1:5000

### Where do I plug in a microphone?

Ideally, you'd have a mixer and amplifier that you could run the line out of the pi to, as well as the microphones.

The pi doesn't have a hardware audio input. Technically, you should be able to run a microphone through it with a USB sound card attached to the pi, but I personally wouldn't bother due to latency and quality issues.

Ideally, you'd have a mixer and amplifier that you could run the line out of the pi to, as well as the microphones. I used this affordable wireless microphone set from amazon: https://www.amazon.com/gp/product/B01N6448Q4/ It has a line in so you can also run PiKaraoke into the mix, and output to an amplifier.

### How do I change song pitch/key?

This is currently not supported due to lack of know-how. As far as I can tell we'd have to pipe omxplayer into some realtime-yet-lightweight audio DSP. Let me know if you have ideas on how to implement it.

In the meantime, you might be able to get away with running the line out through a pitch shift guitar effects pedal or similar device.

### I can't hear audio and I'm getting a blank screen when I try to play a song

If you're using an external USB sound card or hifi audio hat like the hifiberry, you'll need to add the argument --alsa-fix when you launch pikaraoke
