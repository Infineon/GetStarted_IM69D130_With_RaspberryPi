# Important

**This tutorial is outdated. Please look [here](https://github.com/Infineon/i2s-microphone/wiki/Raspberry-Pi-Getting-Started) for the new tutorial.**

# IM69D130 Stereo Microphone Shield2Go with Raspberry Pi

## Supported Platforms

| Platform | Compatible |
| --- |:---:|
| Raspberry Pi 4 Mod. B | :x: |
| Raspberry Pi 3 Mod. B+ | :heavy_check_mark: |
| Raspberry Pi 3 Mod. B | :heavy_check_mark: |
| Raspberry Pi 3 Mod. A+ | :heavy_check_mark: |
| Raspberry Pi Zero | :heavy_check_mark: |  |
| Raspberry Pi Zero W(H) | :heavy_check_mark: |
| Raspberry Pi 2 Mod. B v1.2 | :heavy_check_mark: |
| Raspberry Pi 2 Mod. B | :heavy_check_mark: |
| Raspberry Pi 1 Mod. B+ | :heavy_check_mark: |
| Raspberry Pi 1 Mod. B | :x: |
| Rapsberry Pi 1 Mod. A+ | :x: |
| Raspberry Pi 1 Mod. A | :x: |

## Quick Start Guide

### Preparation of Raspberry Pi

**You can **[skip](#software-setup)** this step, if you already have Raspbian installed on your compatible Raspberry Pi.**

* Download Raspbian from [here](https://www.raspberrypi.org/downloads/raspbian/)
* Unzip Raspbian and flash the image to an SD card. [Here](https://www.raspberrypi.org/documentation/installation/installing-images/) you find a step-by-step tutorial.

#### Headless setup (optional)

**You need network connection for headless operation.**

##### SSH server

For using your Raspberry Pi without a monitor, you need to create an empty file named `ssh` on the boot partition of the SD card.


##### Automatic WiFi connection (optional, if you have an ethernet connection)
Create a file `wpa_supplicant` according to [this](https://www.raspberrypi.org/documentation/configuration/wireless/wireless-cli.md) guide.

#### First login

Now put the SD card in your compatible Raspberry Pi and power it up.

Log in to your Raspberry Pi with the default credentials (user: *pi*, password: *raspberry*). Change the default password for security reasons.

```
passwd
```

### Software setup

#### Enable I2S (Inter-IC-Sound) on your Raspberry Pi

Open the file `/boot/config.txt` for writing with any text editor of your choice. For simplicity we use *nano*, which is pre-installed on Raspbian:

```
sudo nano /boot/config.txt
```

Scroll down to this line:

```
#dtparam=i2s=on
```

and uncomment it (remove the # from the beginning of the line).

Save your changes and close the file.

Now open the file `/etc/modules`:

```
sudo nano /etc/modules
```

and add the following line to the end of the file:

```
snd-bcm2835
```

Save your changes and close the file.

Now reboot your Raspberry Pi to apply the changes:

```
sudo reboot
```

Let's do an intermediate check, if everything worked so far:

```
lsmod | grep snd_soc_bcm2835_i2s
```

**If the output of this command is not empty, all went well so far!**

If you encounter any problems up to this step, please be sure that you use the newest version of Raspbian on a [supported hardware platform](#supported-platforms). In case it still doesn't work, please consider trying the steps on a fresh Raspbian installation. If you still encounter problems, please feel free to open an [issue](https://github.com/Infineon/GetStarted_IM69D130_With_RaspberryPi/issues).

#### Install I2S module

**Please inform yourself about the tool** [rpi-update](https://github.com/Hexxeh/rpi-update) **and the side effects it might have before executing the following commands.**

**If you have any important data on your Raspberry Pi, we recommend to do make a backup before continuing.**

**Please make sure that your Raspberry Pi has a working Internet connection.**

We now need to install the tool `rpi-update`:

```
sudo apt-get update
sudo apt-get install rpi-update
sudo rpi-update
```

After about one minute the following prompt appears:

```
Would you like to proceed? (y/N)
```

Press *y* to continue the process. Make sure to inform yourself about the potential risks **before** you proceed.

The subsequent updating process takes about 4 minutes.

After the process has finished, please reboot your Raspberry Pi to activate the new firmware:

```
sudo reboot
```

Now install these dependencies for subsequent compliation of a kernel with the I2S module:

```
sudo apt-get install git bc libncurses5-dev bison flex libssl-dev
```

Now we come to the biggest step: Downloading and compling the kernel source - Let's go!

```
sudo wget https://raw.githubusercontent.com/notro/rpi-source/master/rpi-source -O /usr/bin/rpi-source
sudo chmod +x /usr/bin/rpi-source
/usr/bin/rpi-source -q --tag-update
rpi-source --skip-gcc
```

**The last command will take about 10 minutes on a Raspberry Pi 3 Mod. B+, so it's time to grab a coffee :)**

If the following prompt appears (after about 7 minutes), please hit *Enter* to accept the default response:

```
Code coverage for fuzzing (KCOV) [N/y/?] (NEW)
```

#### Compile I2S module

Compile the I2S module:

```
sudo mount -t debugfs debugs /sys/kernel/debug
```

**Don't worry if it says: `mount: /sys/kernel/debug: debugs already mounted or mount point busy.` That's ok.**

Now download the [I2S module](https://github.com/PaulCreaser/rpi-i2s-audio) for the Raspberry Pi, written by [Paul Creaser](https://github.com/PaulCreaser):

```
git clone https://github.com/PaulCreaser/rpi-i2s-audio
```

Load the I2C module statically:

```
cd rpi-i2s-audio
make -C /lib/modules/$(uname -r)/build M=$(pwd) modules
sudo insmod my-loader.ko
```

Now let's check if everything worked so far. Use this command to verify if the module has been loaded correctly:

```
lsmod | grep my_loader
```

Now let's use `dmesg` to read out the latest messages of the Kernel Ring Buffer:

```
dmesg | tail
```

Everything worked so far if you see a line like:

```
[X.XXXXXX] asoc-simple-card asoc-simple-card.0: snd-soc-dummy-dai <-> 3f203000.i2s mapping ok
```

In order to enable autostart for the I2S modules, the following commands have to be executed:

```
sudo cp my_loader.ko /lib/modules/$(uname -r)
echo 'my_loader' | sudo tee --append /etc/modules > /dev/null
sudo depmod -a
sudo modprobe my_loader
```

Now it's time for the hardware part before you can test your new setup!

Before you start working on the hardware connections, shutdown your Raspberry Pi:
```
sudo poweroff
```

and unplug the power cable.

### Hardware setup

* Connect the IM69D130 Microphone Shield2Go to your Raspberry Pi

Pin connection:

| Microphone Shield2Go | Raspberry Pi |
| :---: |:---:|
| 3.3V | 3V3 |
| GND | GND |
| DATA | BCM 20 |
| BCLK | BCM 18 |
| CLK | BCM 19 |

* Power up your Raspberry Pi


### Recording Audio

List available input devices with:

```
arecord -l
```

You should see a *snd_rpi_simple_card*.

You can record a wav file with this command:

```
arecord -D plughw:1 -c2 -r 48000 -f S32_LE -t wav -V stereo test.wav
```

By default, the sound level of the recording is very low, but so is the noise, hence you can easily amplify the output file e.g. using [sox](http://sox.sourceforge.net/):

```
sudo apt-get install sox
sox test.wav test_norm.wav --norm=0 trim 10
```

`--norm=0` normalizes the output file to 0 dB and `trim 10` cuts the first 10 seconds of the audio file. This is recommended if you get a clipping peak in the beginning of the recording, because otherwise the normalization does not work correctly.

To sum this up; for recording a sequence of 10 seconds, normalizing it and playing it back you can use the following command:

```
arecord -D plughw:1 -c2 -r 48000 -f S32_LE -t wav -V stereo --duration 20 test.wav && sox test.wav test_norm.wav --norm=0 trim 10 && aplay test_norm.wav
```
