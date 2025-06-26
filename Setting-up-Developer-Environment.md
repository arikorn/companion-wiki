If you only wish to develop modules, you can refer to the [Module Development Wiki](https://github.com/bitfocus/companion-module-base/wiki) for a simpler and more minimal setup.  
The following will also work for developing modules, if you prefer the more manual route

## Platform notes

### Installing WSL

> Many AV users work on Windows, setting up a developers environment is a bit harder then on OSX or Linux. The key is combining linux on Windows. You are installing Linux side-by-side on windows.

First install Windows Subsystem for Linux version 2 (WSL2), then reboot;

`Enable-WindowsOptionalFeature -Online -FeatureName Microsoft-Windows-Subsystem-Linux`

Download from the windows store a Linux distribution (I used Ubuntu LTS search for linux) and install/start it (just follow the steps). This will install a command line interface for Ubuntu. It will start automatically.

If you develop with Visual Studio Code, which we currently recommend, then you can do remote developping. That means VSC is running on Windows but it opens a connection to the virtual machine and actually you edit the files directly on the WSL machine. VSC also has an integrated terminal so you can work on the guest OS. For that you should install the extension [WSL](https://marketplace.visualstudio.com/items?itemName=ms-vscode-remote.remote-wsl). You will see the connection info on the bottom left of each window and you can connect there or by the command palette.

You will need to install all the prerequisite tools on the WSL machine just like you would do on Linux. Look below what these tools are.

When you are running Companion from a VSC terminal, VSC will know the network ports and create automatic port forwards for you. That means although Companion runs on the virtual machine, you'll be able to access the admin page from your Windows host by accessing localhost or 127.0.0.1 and the used port.  
That makes is quite convenient but you always have to remember that Companion runs on a virtual machine inside of your computer and that virtual machine has a different IP-address. When you want to access the API of software running on your Windows host, you can't use 127.0.0.1 like you would without WSL. 127.0.0.1 is the localhost of the WSL. WSL sets up a virtual network interface for you and to access your host OS, you have to use 172.28.96.1. If you want to verify this use the command `ip route` on your guest Linux.

If you want to use USB-devices at WSL, it gets really complicated because they are attached to the host OS and WSL is a very lightweight virtual machine, so it doesn't have USB passthrough features. The solution is an USB over IP software. That means you capture the USB packets at the host, transmit it to the virtual machine and then inject it to the USB system there. Here is how it's done:

- On Windows install the [USB-IP driver](https://github.com/dorssel/usbipd-win/releases)
- Optional but highly recommended install a GUI to control the USB connections: [WSL USB Manager](https://gitlab.com/alelec/wsl-usb-gui/-/releases)
- Make sure your WSL kernel is at least version 5.10.60.1 (`uname -a`). If not update your kernel.
- Install the software on your guest Linux to receive the USB data and some dependencies. Assuming Ubuntu 20 or 22 run:

```
sudo apt install linux-tools-virtual hwdata
sudo update-alternatives --install /usr/local/bin/usbip usbip /usr/lib/linux-tools/*/usbip 20
```

- Like on other Linux distributions the virtual USB devices will only be accessible from the root user. To make them accessible to regular users, you need to add new udev rules. `sudo nano /etc/udev/rules.d/50-companion.rules` will open an editor.
- add `SUBSYSTEM=="usb", GROUP="plugdev", MODE="0666"` to the file. This means that all USB devices will get read/write permissions for the owner and for the group "plugdev". Save with CTRL-O and exit with CTRL-X
- make sure the group plugdev exists with `sudo groupadd plugdev`. Usually it exists already.
- add your user to the group plugdev `sudo usermod -aG plugdev $USER`.
- Now reboot the Computer
- Open the WSL USB Manager
- Attach your USB device, you should see it popping up in the top pane of the manager.
- Check the "bound" checkbox next to your USB device
- Select the USB device by clicking on it and then press the "Attach" button. The device should now move from the top pane to the forwarded devices pane.
- on the linux guest use `lsusb` to check if you find your device and remember the bus and device number.
- on the linux guest use `cat \dev\bus\usb\001\003` to check if you have access to the device. Replace the numbers (\001\003) with the bus number and the device number that you have found with lsusb
- WSL kernel does not have OS features for HID protocol, you will be able to access HID devices like a regular USB device, but the kernel does not provide HID wrapping.

### Using homebrew on macos

On macos, the typically way to install git and other tools is with [Homebrew](https://brew.sh/)

## The process

### 1) Install node.js

There are many ways of doing this. We recommend using a version manager, to allow for easily updating and switching between versions.

In the past, we recommended using `n`, but that requires a version node.js to be installed first.  
Our recommendation is to use [fnm](https://github.com/Schniz/fnm#installation) It is fast and cross-platform. You are free to install any other way you wish, but you will need to figure out the correct commands or ensure you have the right version at each point.

Once you have installed fnm execute the following in a terminal, to install node.js v18 and make it be the default.

```
fnm install 22
fnm use 22
fnm default 22
corepack enable
```

> When using Git Bash on windows, you can get into trouble with line endings (Windows uses CRLF while Linux uses LF). `git config core.autocrlf true` converts this for you

### 2) Other tools

To edit the source code or write new code you can use any text editor you like, but there are many editors which are made especially for developing computer code or even better especially for JavaScript.
If you have no idea you should try the [Visual Studio Code](https://code.visualstudio.com/) editor.

You will also need to install [git](https://git-scm.com/), which is what we use with to manage, upload and download the sourcecode with [GitHub](https://github.com/bitfocus). On macos this available from homebrew.  
If you have never used git or github before, have a read of our [Git crashcourse](Git-crashcourse).

TODO - we should recommend a free git gui tool

### 2a) Linux dependencies

If you are using linux, you should follow the dependencies and udev rules steps as described in the README included in the release builds https://github.com/bitfocus/companion/tree/main/assets/linux.

For WSL, you should follow the dependencies portion.

You may also need to install python, which on Ubuntu can be achieved with: `sudo apt install python`

### 3) Companion preperation

Using your git client, you can clone Companion.

Once you have done this, in a terminal (the console window inside vscode is perfect for this) run `yarn update` to prepare your clone for being run. You will need to do this every time you update your clone as we are often updating dependencies or changing the webui code.

You can now run Companion with `yarn dev`

By default this will serve the prebuilt version of the webui, which will not update as you make changes. If you wish to run the webui in development mode in a second terminal window/tab run `yarn dev-webui`. This will launch the development version of the webui on a different port, typically http://localhost:5173

This will run the 'headless' version of Companion, without the red launcher window. If you want to run with that, you can cd into the `launcher` folder and run `yarn dev`. On WSL this may require additional dependencies to be installed.
