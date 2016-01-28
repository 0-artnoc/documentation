## Connecting to a Pi over VNC using Mac OS

For Mac OS you will not need any extra software. Just select ``Go -> Connect to server ...`` (&#8984; K) from the Finder menu and enter
``vnc://raspberrypi.local:1`` as the Server Address and click ``Connect``. Here ``:1`` must correspond to the display on which you started the vnc server on your pi. In case of problems you could also try to replace ``raspberrypi.local`` by the IP address of your pi.

Alternatively you can use a program called RealVNC which is known to work with the Raspberry Pi VNC server, it can be downloaded from [realvnc.com](http://www.realvnc.com/download/vnc/latest)

Download the package file and open it. During the setup you'll be offered a choice of the type of installation. You only need the VNC viewer on your Mac and not the server so select custom and then *uncheck* VNC Server (see below).

![VNC OSX installation](images/osx/vnc-osx-install.png)

Click `Continue` and go ahead with the rest of the installation. Once the installation is complete open the finder, then select Applications on the left and enter vnc into the search box. VNC Viewer should then be shown. Perhaps create a shortcut to it in your Dock for future use.

![VNC connection dialog](images/osx/vnc-osx-connect.png)

When you run it you'll be presented with the dialog above. You will need to enter the IP address of the Raspberry Pi followed by the screen number (:0 or :1). For example: 192.168.0.165:1

Click the Connect button and you'll be given an unencrypted connection warning.

![Unencrypted connection warning](images/osx/vnc-osx-warning.png)

Generally speaking this warning only needs consideration if the connection between your Mac and the Pi is going over the Internet. If you're using a Local Area Network or a School network then you don't really need to worry. Click *Continue* and you'll be prompted for the password that was specified when configuring the VNC server on the Raspberry Pi earlier. You should then find yourself at the Raspberry Pi desktop.

![Raspberry Pi desktop](images/osx/vnc-osx-connected.png)

Don't use the *logout* menu as you would on the Raspberry Pi desktop when you want to close down. Just close the RealVNC window itself and then use the kill command on the Raspberry Pi, described above, to shut down the VNC server.

For further documentation on RealVNC please visit this page: http://www.realvnc.com/products/vnc/documentation/latest/
