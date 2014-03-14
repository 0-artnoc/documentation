# SSH (Secure Shell)

You can remotely gain access to the command line of a Raspberry Pi from another computer on the same network using `ssh`.

Note you only have access to the command line, not the full desktop environment. For full remote desktop see [VNC](../vnc/README.md).

Firstly, you must enable the SSH Server on your Raspberry Pi. This is done using [raspi-config](../../configuration/raspi-config.md):

Enter `sudo raspi-config` in the Terminal, then navigate to `ssh`, hit `Enter` and select `Enable or disable ssh server`.

SSH is built in to Linux distribtions and Mac OS, and a third party SSH client is available for Windows. See the following guides for using SSH depending on the Operating System used by the computer you are connecting from:

- [Linux & Mac OS](unix.md)
- [Windows](windows.md)
