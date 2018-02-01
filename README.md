![nokia-logo](https://www.vanillaplus.com/wp-content/uploads/2016/11/Nokia-Logo-720x259.jpg)
# Setup Guide for Using the Raspberry Pi with CDP
In this guide, we will go over configuring a Raspberry Pi with the Leshan LwM2M client and connecting the Pi to a CDP server.
## Requirements
- Raspberry Pi with Internet connection (note: cannot use networks behind a proxy)
- Laptop connected to the same local network
- CDP Instance
### Step 1: Setting up the Pi
In order to work with the Raspberry Pi, you *must* have SSH enabled. To enable this, you can either:
- Boot your Raspberry Pi with a Keyboard/Mouse/Monitor, then access the Raspberry Pi settings by clicking on the Raspberry menu button (top left), then going to Preferences->Raspberry Pi Configuration.
- Remove the SD card from your Pi, insert the SD card in a computer, then place an empty text file called ```ssh``` at the root of the card. **Note:** Ensure there is no file extension on this file. Windows will attempt to automatically append a ```.txt``` to the end of this file. After doing this, insert the SD card in your Pi and boot.

Then, you must have your Pi connected to the same network as your laptop in order to have access to it. In order to do so, you must do one of the following:
- Connect using a wired Ethernet connection
- Boot the Raspberry Pi with a Keyboard/Monitor/Mouse and configure using the Network item in the taskbar (top right).
- Insert the Pi's SD card in your laptop, and copy [this](wpa_supplicant.conf) file to the root of your SD card. Edit the file with Notepad, and set the ```ssid``` field to your network's name and the ```psk``` to your network's password. Insert back into the Pi and boot.

### Step 2: Finding the Pi on the Local Network
Next, we must get access to the Raspberry Pi in order to install the Leshan client. In order to do so, you should use your laptop to open an SSH terminal with the Raspberry Pi.
You will need to know the local IP address of your Raspberry Pi in order to SSH into the device. This can be retrieved from:
- Viewing the Device list on the Router's administration page.
- Using ```nmap``` to ping all devices that are available on the network.

```nmap``` should be already available on most Unix distributions (i.e. macOS, Linux, etc.), however you will have to install it from [here](https://nmap.org/dist/nmap-7.60-setup.exe) if you are using Windows.

First, find the IP Address range of your local network. On a Unix machine, run the following command:
```
ifconfig
```
and look for the adapter that you are using to connect to the internet (Should look like ```en0```, ```wlan0```, etc.). Take the first *three* parts of your IP Address that is listed, as this will be used later with nmap.

For Windows, the command is:
```
ipconfig
```
Again, look for the adapter that you are using to connect to the internet and copy down the first three parts (octets) of your IP address.

Finally, to find the Pi on your network, run the following command:
```
nmap -sP {first}.{three}.{octets}.1-255
```
This will produce a list of live devices on your network. Find the IP Address associated to a MAC Address beginning with ```B8:27:EB```, as this is the OUI of the Raspberry Pi Foundation.

### Step 3: Connecting to the Pi
Now that you have the Raspberry Pi's IP address, you can connect to it in order to install the Leshan client. To do so, you can use the ```ssh``` command on a Unix-based OS, or [PuTTY](https://www.putty.org/) from a Windows machine. The host of the SSH session will be the Local IP address found in Step 2. The default username for the Raspberry Pi is ```pi``` and the password is ```raspberry```.

If successful, you should see a prompt that looks like so:
```
pi@raspberrypi:~ $
```
### Step 4: Java and the Leshan Client
From here, we need to check that Java is installed. Run
```
java -version
```
to see whats available. If the version is less than 1.8, or if the command is not found, install the proper Java version by running:
```
sudo apt-get update
sudo apt-get install -y openjdk-8-jdk-headless
```
Next, we will download the leshan client to the Raspberry Pi:
```
wget -o leshan.jar http://central.maven.org/maven2/org/eclipse/leshan/leshan-client-demo/1.0.0-M5/leshan-client-demo-1.0.0-M5.jar
```
### Step 5: Creating a Device on CDP
Log in to your CDP instance, then click on the 'Devices' tab in the left-hand side, then 'Add Device'. For the Serial Number, specify a name for the device, like ```oliver_pi```. This name will be used later when we start the Leshan client, so don't forget it. Set the Manufacturer to ```Generic``` and the Device to ```LWM2M Generic Device```. Save this, and exit CDP.
### Step 6: Connecting the Raspberry Pi to CDP
For this step, you will need to go back to the SSH terminal you had open from earlier. Also, you will need the *hostname* and *port* for your CDP instance's CoAP/UDP port. By default this is ```5683```, but is often different.

To start the Raspberry Pi's Leshan client, run this command, providing your *hostname*, *port*, and *Device ID* from the previous steps.
```
java -jar leshan.jar -n {YOUR_DEVICE_ID} -u {YOUR_CDP_HOSTNAME}:{YOUR_CDP_PORT}
```
### Step 7: Accessing the Device
Go back to the CDP Console that you had open from earlier, and select the device you made for your Raspberry Pi. Click on it and confirm that the client is connected by checking the Last Modified time. If this is close to current time, then your device is connected to CDP. 

Try running an action against the device. For example, running an ```LWM2M GET``` against the resource path ```/3/0/0``` will return the Manufacturer's name.