# BIQU-B1-Klipper-Settings
printer.conf and other notes for installing Klipper on a stock BIQU B1 with a Pi Zero 2 W using UART

## Raspberry Pi Zero 2 W Setup
### OS Imaging
Follow [MainsailOS Installation Guide](https://docs.mainsail.xyz/setup/mainsailos/pi-imager) to image the SD card.

### Network Configuring
Before moving SD card to the PI, edit the following files using a proper text editor (VSCode, Vim, Notepad++, nano, etc)

``mainsailos-wpa-supplicant.txt`` - update your network ID and password:

    network={
     ssid="MyNetworkSSID"
     psk="Pa55w0rd1234"
    }

``config.txt`` - replace the last section with:

    [pi0w]
    enable_uart=1
    dtoverlay=pi3-miniuart-bt
    
### Initial Boot and Updates
With the Pi still disconnected from the printer, insert the SD and power it up using a USB adapter for now (eventually it will be powered using the 5V from the UART connection).

After a minute or two, you should be able to point your browser on any computer on the same network to `http://mainsailos.local` and see the Mainsail dashboard. There's probably a handful of warnings, but we're not done yet so that's to be expected.

In the **Machine** tab, use the **Update Manager** panel to update klipper, moonraker, and mainsail if available. The System OS-Packages update likely won't work yet, we'll address that below.

For the next few steps, you'll need to SSH into the Pi (using PuTTY on Windows, terminal on Mac or Linux), the IP should be `mainsailos.local`, default username and password are `pi` and `raspberry`.

It's a good idea to update all the packages on the Pi manually at first, though in the next step we'll enable Moonraker to do this via the dashboard (this may take a while):
    
    sudo apt update
    sudo apt upgrade
    
To give Moonraker the permissions is needs in order to perform some tasks like updating OS packages, run the following:

    cd ~/moonraker/scripts
    ./set-policykit-rules.sh
    sudo service moonraker restart
    
### Create printer.cfg
Last step before moving on to the printer is creating the config file that Klipper will use to define your machine. You can type this via SSH by creating a file in `~/klipper_config` called `printer.cfg`, though it's likely easier through the Mainsail dashboard on your computer. 

In the **Config Files** panel of the **Machine** tab, ensure the **Root** dropdown is set to *"config"*, click **Create File** and name it `printer.cfg`.

Copy/Paste my [config file](https://github.com/JoeTodes/BIQU-B1-Klipper-Settings/blob/main/printer.cfg) into the editor, then click **Save and Restart**

## Flash SKR V1.4 Firmware
Before we can connect the Pi, we have to flash the Klipper firmware to the printer's mainboard which in our case is an SKR V1.4

SSH into the Pi and run the following commands to bring up the firmware configuration menu:

    cd ~/klipper/
    make menuconfig
    
This should bring up a settings menu you can navigate with the arrow keys and space bar. Here are the settings we need for our hardware:

![BIQU B1 UART Firmware Settings](https://github.com/JoeTodes/BIQU-B1-Klipper-Settings/blob/main/klipper_menuconfig.png)

Note the specific UART port (UART0 P0.3/P0.2) and that we are using a "Smoothiware bootloader"

Hit **Q** to save and quit, the run

    make
    
to generate the firmware file, which will be located at `~/klipper/out/klipper.bin`

We need to get that file off of the Pi and on to a separate SD card to flash the mainboard. Use something like WinSCP or Filezilla on PC or `scp` in the terminal on Mac or Linux to move `klipper.bin` to your computer and on to the other SD card. You also **need** to rename it to `firmware.bin` in order for the printer to recognize it.

Stick the SD card into the printer (in the **TF Card** slot closer to the gantry, not the one by the click-wheel which is part of the touch screen), power on the printer, and wait a few minutes.

To verify that the firmware was flashed, remove the SD card, plug it into your computer and check that `firmware.bin` was renamed to `firmware.cur`

## Wiring
With the **printer** and **Pi** powered off and unplugged, remove the bottom panel to access the mainboard. We have 3 tasks in here:

1. Remove the ribbon cable connecting the mainboard to the touch screen. It won't work with Klipper (yet) and we need one of the ports for the UART connection to the Pi 
2. Move the filament runout sensor connector (labelled **FLD**) from the touchscreen to the E0DET port on the mainboard (see [SKR V1.4 ReferenceDiagram](https://github.com/JoeTodes/BIQU-B1-Klipper-Settings/blob/main/SKR%201-4%20Pinout.jpg), about halfway up the right side).
3. Construct and connect the cable assembly you will use for the Pi to the UART connection labelled TFT (where the touchscreen used to be connected). See [Nero 3D's excellent video](https://youtu.be/AtW3GqkKUz8?t=229) for an example cable assembly. The pin connections should be as follows:

| Pi GPIO Pin | Purpose | SKR V1.4 TFT Pin |
|---|---|---|
| 4 | 5V | NPWR |
| 6 | GROUND | GND |
| 8 | TX->RX | RX0 |
| 10 | RX<-TX | TX0 |

Because we are using the 5V from this connection, you will no longer need to power the Pi with the USB adapter.

## First Full Boot
After you've triple checked your wiring, it's time to power on the printer again. This should also power on the Pi and after a few moments you should be able to reconnect your browser to `mainsailos.local` to verify that everything is up and running and configured correctly.

Before you start printing, it's definitely recommended to go through and test that everything is set up correctly. Follow the steps in the [Klipper docs](https://www.klipper3d.org/Config_checks.html) or in [Nero 3D's walkthrough](https://www.youtube.com/watch?v=T-knWbh1Gg8).

Note that `pressure_advance` and `input_shaper` settings have been commented out in my [printer.cfg](https://github.com/JoeTodes/BIQU-B1-Klipper-Settings/blob/main/printer.cfg), you should calibrate those settings for your own printer. Again, the official Klipper docs and Nero 3D are both great resources for these processes.

Let me know in an **Issue** if you run into any problems following these steps, we're all in this together. Keep your stick on the ice, I'm pulling for you. 
