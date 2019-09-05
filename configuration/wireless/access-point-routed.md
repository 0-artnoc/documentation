# Setting up a Raspberry Pi as a Routed Wireless Access Point

The Raspberry Pi can be used as a wireless access point routing to an existing ethernet network. This will create a new wireless network entirely managed by the Raspberry Pi.

If you wish to extend an existing ethernet network to wireless clients, consider instead setting up a [bridged access point.](./access-point-bridged.md)

```
                                         +- RPi -------+
                                     +---+ 10.10.0.2   |
                                     |   |     WiFi AP +-)))
                                     |   | 192.168.4.1 |         +- Laptop ----+
                                     |   +-------------+     (((-+ WiFi STA    |
                 +- Router ----+     |                           | 192.168.4.2 |
                 | Firewall    |     |   +- PC#2 ------+         +-------------+
(Internet)---WAN-+ DHCP server +-LAN-+---+ 10.10.0.3   |
                 |   10.10.0.1 |     |   +-------------+
                 +-------------+     |
                                     |   +- PC#1 ------+
                                     +---+ 10.10.0.4   |
                                         +-------------+

```

This can be done using the inbuilt wireless features of the Raspberry Pi 3 or Raspberry Pi Zero W, or by using a suitable USB wireless dongle that supports access point mode.
It is possible that some USB dongles may need slight changes to their settings. If you are having trouble with a USB wireless dongle, please check the forums.

This documentation was tested on a Raspberry Pi 3B running a factory installation of Raspbian Buster Lite (Jul. 2019). 

## Before you start

* Ensure you have administrative access to your Raspberry Pi. The network setup will be modified as part of the installation: local access, with screen and keyboard connected to your Raspberry Pi, is recommended.
* Connect your Raspberry Pi to the ethernet network and boot the Raspbian OS.
* Ensure the Raspbian OS on your Raspberry Pi is [up to date](../../raspbian/updating.md) and reboot if packages were installed in the process.
* Take note of the IP configuration of the ethernet network the Raspberry Pi is connected to. Routing is performed at the border of two (or more) separate networks. In this document, we assume IP network `10.10.0.0/24` is configured on the ethernet LAN, and the Raspberry Pi will manage IP network `192.168.4.0/24` for wireless clients.
    * *Note:* Please select another IP network for wireless, e.g. `192.168.10.0/24`, in case IP network `192.168.4.0/24` is already in use by your ethernet LAN.
* Have a wireless client (laptop, smartphone, ...) ready to test your new access point.

<a name="software-install"></a>
## Install the access point and network management software (hostapd, dnsmasq)

In order to work as an access point, the Raspberry Pi needs to have the `hostapd` access point software package installed:

```
sudo apt install hostapd
```
Enable the wireless access point service to start when your Raspberry Pi boots:

```
sudo systemctl unmask hostapd
sudo systemctl enable hostapd
```

In order to provide network management services (DNS, DHCP) to wireless clients, the Raspberry Pi needs to have the `dnsmasq` software package installed:

```
sudo apt install dnsmasq
```

Software installation is complete. We will configure the software packages later on.

<a name="routing"></a>
## Setup the network router

The Raspberry Pi will run and manage a stand-alone wireless network. At your option, the Raspberry Pi will route between the wireless and the ethernet networks, providing Internet access to wireless clients. 

### Define the wireless interface IP configuration (dhcpcd)

In this document, we assume IP network `10.10.0.0/24` is configured on the ethernet LAN, and the Raspberry Pi will manage IP network `192.168.4.0/24` for wireless clients.
*Note:* Please select another IP network for wireless, e.g. `192.168.10.0/24`, in case IP network `192.168.4.0/24` is already in use by your ethernet LAN.

The Raspberry Pi runs a DHCP server for the wireless network, this requires static IP configuration for the wireless interface (`wlan0`) in the Raspberry Pi. 
The Raspberry Pi acts as router on the wireless network, as customary we will give it the first IP address in the network: `192.168.4.1`.

To configure the static IP address, edit the configuration file for `dhcpcd` with:

```
sudo nano /etc/dhcpcd.conf
```

Go to the end of the file and add the following:

```
interface wlan0
    static ip_address=192.168.4.1/24
    nohook wpa_supplicant
```

### Enable routing and IP masquerading

This section configures the Raspberry Pi to let wireless clients access computers on the main network, and from there the Internet.
**NOTE:** If you wish to block wireless clients from accessing the main network and the Internet, skip this section. 

To enable routing, i.e. allow traffic to flow from one network to an other in the Raspberry Pi, edit /etc/sysctl.conf and uncomment this line:

```
net.ipv4.ip_forward=1
```
Enabling routing will allow hosts from network `192.168.4.0/24` to reach the LAN and the main router for Internet access. This is an unknown IP network to that router. 

In order to allow traffic between wireless clients and the Internet without any change to the configuration of the main router, the Raspberry Pi can substitute the IP address of wireless clients with its own IP address on the LAN using a "masquerade" firewall rule.
* The main router will see all outgoing traffic from wireless clients are coming from the Raspberry Pi, allowing communication with the Internet.
* The Raspberry Pi will receive all incoming traffic sollicited by the wireless clients, substitute the IP addresses back and forward the data to the origin wireless client.

This process is configured by adding a single firewall rule in the Raspberry Pi:

```
sudo iptables -t nat -A POSTROUTING -o eth0 -j MASQUERADE
```

Next, save the firewall configuration to file:

```
sudo sh -c "iptables-save > /etc/iptables.ipv4.nat"
```

To reinstate the firewall rule when your Raspberry Pi boots, edit file `/etc/rc.local` and add the following line just above "exit 0":

```
iptables-restore < /etc/iptables.ipv4.nat
```
<a name="dnsmasq-config"></a>
## Configure the DHCP and DNS services for the wireless network (dnsmasq) FIXME DNS?

The DHCP and DNS services are provided by `dnsmasq`. The default configuration file serves as template for all possible configuration options, when we only need a few. It is easier to start from an empty file. 

Rename the default configuration file and edit a new one:

```
sudo mv /etc/dnsmasq.conf /etc/dnsmasq.conf.orig
sudo nano /etc/dnsmasq.conf
```
Add the following to the file and save it:

```
interface=wlan0      # Listening interface
dhcp-range=192.168.4.2,192.168.4.20,255.255.255.0,24h # Pool of IP addresses served via DHCP
```

The Raspberry Pi will deliver IP addresses between `192.168.4.2` and `192.168.4.20`, with a lease time of 24 hours, to wireless DHCP clients.

There are many more options for `dnsmasq`; see the default configuration file or the [online documentation](http://www.thekelleys.org.uk/dnsmasq/doc.html) for more details.



<a name="hostapd-config"></a>
## Configure the access point software (hostapd)

Create the `hostapd` configuration file, located at `/etc/hostapd/hostapd.conf`, to add the various parameters for your wireless network. 

```
sudo nano /etc/hostapd/hostapd.conf
```

Add the information below to the configuration file. This configuration assumes we are using channel 7, with a network name of `NameOfNetwork`, and a password `AardvarkBadgerHedgehog`. Note that the name and password should **not** have quotes around them. The passphrase should be between 8 and 64 characters in length.

To use the 5 GHz band, you can change the operations mode from `hw_mode=g` to `hw_mode=a`. Possible values for `hw_mode` are:
 - a = IEEE 802.11a (5 GHz)
 - b = IEEE 802.11b (2.4 GHz)
 - g = IEEE 802.11g (2.4 GHz)
 - ad = IEEE 802.11ad (60 GHz)

```
interface=wlan0
#driver=nl80211
ssid=NameOfNetwork
hw_mode=g
channel=7
#wmm_enabled=0
macaddr_acl=0
auth_algs=1
ignore_broadcast_ssid=0
wpa=2
wpa_passphrase=AardvarkBadgerHedgehog
wpa_key_mgmt=WPA-PSK
wpa_pairwise=TKIP
rsn_pairwise=CCMP
```

## Run your new wireless access point

It is now time restart your Raspberry Pi and verify the wireless access point becomes automatically available.

```
sudo systemctl reboot
```
Once your Raspberry Pi has restarted, search for WiFi networks with your wireless client. The network SSID you specified in file `/etc/hostapd/hostapd.conf` should now be present, and it should be accessible with the specified password.

If SSH is enabled on the Raspberry Pi, it should be possible to connect to from your wireless client as follows, assuming the `pi` account is present: `ssh pi@192.168.4.1` FIXME or `ssh pi@gw.wlan`

If your wireless client has access to your Raspberry Pi (and the Internet), congratulations on your new access point!

If you encounter difficulties, read the section below.

### Troubleshooting tips TODO
Please follow these steps and verify your configuration:
* *Step 1:* From a computer on the network, does `ping raspberrypi.local` work, and shows an IP address that belongs to the same network as the computer, for example `192.168.1.2`?
    * If ping fails or the address looks different, like `169.254.x.x`, verify [bridge setup](#bridging) (`systemd-networkd` and `dhcpcd`)
    * If ping succeeds, on to Step 2
* *Step 2:* From your test wireless client, do you see the WiFi network name and can successfuly authenticate using the password defined in file `/etc/hostapd/hostapd.conf`?
    * If the wireless client cannot find the network or authentication fails, verify access point software [installation](#hostapd-install) and [configuration.](#hostapd-config)
    * If connecting to the wireless network succeeds, but the wireless client cannot reach machines on the network or the Internet, verify that the DHCP server on the network (often located in the router) answers to the IP address request coming from the wireless client.
    * If the wireless access point and the DHCP server seem to be working, on to Step 3
* *Step 3:* Contact the forums for further assistance. Please refer to this page in your message.

By this point, the Raspberry Pi is acting as a router and access point, and other devices can associate with it. Associated devices can access the Raspberry Pi access point via its IP address for operations such as `rsync`, `scp`, or `ssh`.

