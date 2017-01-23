# Setting WiFi up via the command line


This method is suitable if you don't have access to the graphical user interface normally used to set up WiFi on the Raspberry Pi. It's especially suitable for use with a serial console cable if you don't have access to a screen or wired Ethernet network. Note also that no additional software is required; everything you need is already included on the Raspberry Pi.   

## Getting WiFi network details  

To scan for WiFi networks, use the command `sudo iwlist wlan0 scan`. This will list all available WiFi networks, along with other useful information. Look out for:

1. `ESSID:"testing"` is the name of the WiFi network.   

2. `IE: IEEE 802.11i/WPA2 Version 1` is the authentication used; in this case it's WPA2, the newer and more secure wireless standard which replaces WPA. This guide should work for WPA or WPA2, but may not work for WPA2 enterprise; for WEP hex keys, see the last example [here](http://www.freebsd.org/cgi/man.cgi?query=wpa_supplicant.conf&sektion=5&apropos=0&manpath=NetBSD+6.1.5). You'll also need the password for the WiFi network. For most home routers this is located on a sticker on the back of the router. The ESSID (ssid) for the network in this case is `testing` and the password (psk) is `testingPassword`.

3. If `security` is a concern entering a password as plain text is not recommended. You can use the `wpa_passphrase` to create a unique token and enter it instead. As arguments the command requires the ESSID and the password. With the example from above calling the command will be `wpa_passphrase "testing" "testingPassword` which will output a ready-to-use wpa-compliant network profile:

  ```
  network={
	  ssid="testing"
	  #psk="testingPassword"
	  psk=131e1e221f6e06e3911a2d11ff2fac9182665c004de85300f9cac208a6a80531
  }
  ```
  
  As you can see the password `testingPassword` is still visible as a comment namely `#psk="testingPassword"`. When pasting the produced profile in your configuration file make sure to remove this line (otherwise the whole procedure of generating the secret token would be pointless). The tool requires a password with at least 8 and up to 63 characters. You can also extract the content of a text file and use it as input of `wpa_passphrase` if the password is stored as plain text inside a file somewhere by calling `wpa_passphrase "testing" << file_where_password_is_stored`.
  
  
## Adding the network details to the Raspberry Pi
   
Open the `wpa-supplicant` configuration file in nano:

`sudo nano /etc/wpa_supplicant/wpa_supplicant.conf`  

Go to the bottom of the file and add the following:   

```
network={
    ssid="The_ESSID_from_earlier"
    psk="Your_wifi_password"
}
```

In the case of the example network, we would enter:  

```
network={
    ssid="testing"
    psk="testingPassword"
}
```

If you are using `wpa_passphrase` you can redirect its output to your configuration file by calling `wpa_passphrase "testing" "testingPassword" >> /etc/wpa_supplicant/wpa_supplicant.conf`. Note that this requires you to change to `root` (by executing `sudo su`) or find another way to redirect the output since the file we write to can be altered only by a user with administrative privileges. Last but not least make sure you use `>>` (which is used to append text to an existing file) since `>` will erase all contents and **then** append the output to the specified file.
   
Now save the file by pressing **Ctrl+X** then **Y**, then finally press **Enter**.  

At this point, `wpa-supplicant` will normally notice a change has occurred within a few seconds, and it will try and connect to the network. If it does not, either manually restart the interface with `sudo ifdown wlan0` and `sudo ifup wlan0`, or reboot your Raspberry Pi with `sudo reboot`.   

You can verify if it has successfully connected using `ifconfig wlan0`. If the `inet addr` field has an address beside it, the Pi has connected to the network. If not, check your password and ESSID are correct.   
