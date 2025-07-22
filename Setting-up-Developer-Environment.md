If you only wish to develop modules, you can refer to the [Module Development Wiki](https://github.com/bitfocus/companion-module-base/wiki) for a simpler and more minimal setup.  
The following will also work for developing modules, if you prefer the more manual route

## Platform notes

### Installing WSL

Many AV users work on Windows, setting up a developers environment is a bit harder then on OSX or Linux. The key is combining linux on Windows. You are installing Linux side-by-side on windows.

First install Windows Subsystem for Linux version 2 (WSL2),   
WSL is included with Windows but needs to be activated. Follow [Microsoft’s installation instructions](https://learn.microsoft.com/en-us/windows/wsl/setup/environment). 

Basically, open a command or powershell window and type  
~~~  
   wsl --install  
~~~  
After this completes, reboot your computer and WSL will automatically install the latest LTS version of Ubuntu. (24.04 at this time of writing).  Once this is done continue through Microsoft’s instructions, especially  
    `sudo apt update && sudo apt upgrade`, 
also the instructions to:	
 - Set up Windows terminal,  
 - Set up VS Code, and   
 - Set up Git.

If you develop with Visual Studio Code, which we currently recommend, then you can do remote developing. That means VSC is running on Windows but it opens a connection to the virtual machine and actually you edit the files directly on the WSL machine. VSC also has an integrated terminal so you can work on the WSLOS. For that you should install the extension [WSL](https://marketplace.visualstudio.com/items?itemName=ms-vscode-remote.remote-wsl) and follow [Microsoft’s instructions here](https://learn.microsoft.com/en-us/windows/wsl/tutorials/wsl-vscode). You will see the connection info on the bottom left of each window and you can connect there or by the command palette. You can debug using breakpoints etc. either by starting automatic debugging from the terminal or by attaching to the running node.js process (which should be the first one in the list when you type ctrl-shift-P and “Debug: Attach to Node Process”.

You will need to install all the prerequisite tools on the WSL machine just like you would do on Linux. Look below what these tools are.

When you are running Companion from a VSC terminal, VSC will know the network ports and create automatic port forwards for you. That means although Companion runs on the virtual machine, you'll be able to access the admin page from your Windows host by accessing localhost or 127.0.0.1 and the used port.  
That makes it quite convenient but you always have to remember that Companion runs on a virtual machine inside of your computer and that virtual machine has a different IP-address. When you want to access the API of software running on your Windows host, you can't use 127.0.0.1 like you would without WSL. 127.0.0.1 is the localhost of the WSL. WSL sets up a virtual network interface for you and to access your host OS, you have to use 172.28.96.1. If you want to verify this use the command `ip route` in your Linux shell.

If you want to use USB-devices at WSL, it gets complicated because WSL doesn't have built-in USB passthrough features. The solution is USB over IP software. That means you capture the USB packets at the host, transmit it to the virtual machine and then inject it to the USB system there. Here is how it's done:

- On Windows install the [USB-IP driver](https://github.com/dorssel/usbipd-win/releases)  
- Optional but highly recommended install a GUI to control the USB connections: [WSL USB Manager](https://gitlab.com/alelec/wsl-usb-gui/-/releases)  
- Make sure your WSL kernel is at least version 5.15.150.1 (released Mar 2024) (`uname -a`). If not, update your kernel.  
- Linux distributions need extra rules to make USB devices accessible to users:
- Copy the following code into a bash shell (terminal):
~~~
sudo cat <<EOF |sudo tee /etc/udev/rules.d/50-companion-WSL.rules
SUBSYSTEM=="usb|hidraw", GROUP="plugdev", MODE:="0666"
EOF

~~~
- Now reboot the Computer (It may be sufficient to type: `sudo udevadm control --reload-rules`.)
- Open the WSL USB Manager in Windows
- Attach your USB device, you should see it popping up in the top pane of the manager.  
- Check the "bound" checkbox next to your USB device (usbip will now remember this permission; you won't have to do it again.) 
- Select the USB device by clicking on it and then press the "Attach" button. The device should now move from the top pane to the forwarded devices pane.  
- Optional: on the linux shell use `lsusb` to check if you find your device and remember the bus and device number.  
- Optional: In the linux shell use `cat /dev/bus/usb/001/003` to check if you have access to the device. Replace the numbers (/001/003) with the bus number and the device number that you have found with lsusb  
- *Note: most Companions USB surfaces are handled as HID devices. WSL presents the HID interface through a separate “device” than shown by* `lsusb` *Instead when an HID device is added via the usb-ip driver, it will set up* `/dev/hidraw#` where # is an integer starting from 1. If permissions are incorrectly set, Companion will output an error message in the terminal. *You can also test it directly, as above.*  

### Using homebrew on macos

On macos, the typically way to install git and other tools is with [Homebrew](https://brew.sh/)

## The process

### 1) Install node.js

There are many ways of doing this. We recommend using a version manager, to allow for easily updating and switching between versions.

In the past, we recommended using `n`, but that requires a version node.js to be installed first.  
Our recommendation is to use [fnm](https://github.com/Schniz/fnm#installation) It is fast and cross-platform. (If a dependency is missing, use `sudo apt install` to add it.) You are free to install any other way you wish, but you will need to figure out the correct commands or ensure you have the right version at each point.

Once you have installed fnm execute the following in a terminal, to install node.js v22 and make it be the default.

```
fnm install 22
fnm use 22
fnm default 22
corepack enable
yarn init -2
```

> When using Git Bash on windows, you can get into trouble with line endings (Windows uses CRLF while Linux uses LF). `git config core.autocrlf true` converts this for you

### 2) Other tools

To edit the source code or write new code you can use any text editor you like, but there are many editors which are made especially for developing computer code or even better especially for JavaScript.
If you have no idea you should try the [Visual Studio Code](https://code.visualstudio.com/) editor.

You will also need to install [git](https://git-scm.com/), which is what we use to manage, upload and download the source code with [GitHub](https://github.com/bitfocus). On macos this is available from homebrew.  
If you have never used git or github before, have a read of our [Git crashcourse](https://github.com/bitfocus/companion-module-base/wiki/Git-crashcourse).

For a simple windowing editor, you can try gedit. Install it with `sudo apt install gedit`

TODO - we should recommend a free git gui tool  [Perhaps SmartGit?]

### 2a) Linux dependencies

If you are using linux, you should follow the dependencies and udev rules steps as described in the README included in the release builds https://github.com/bitfocus/companion/tree/main/assets/linux.

For WSL, you should follow the dependencies portion since you already installed the udev rules, above.

You may also need to install python, which on Ubuntu can be achieved with: `sudo apt install python`

### 3) Companion preparation

Using your git client, you can clone Companion.

Once you have done this, in a terminal (the console window inside vscode is perfect for this) run  `yarn install` to prepare your clone for being run. You will need to do this every time you update your clone as we are often updating dependencies or changing the webui code.

You can now run Companion with `yarn dev`

By default this will serve the prebuilt version of the webui, which will not update as you make changes. If you wish to run the webui in development mode in a second terminal window/tab run`yarn dev:webui`. (or `yarn dist:webui` to just compile it)This will launch the development version of the webui on a different port, typically [http://localhost:5173](http://localhost:5173)
***Important: you still need to run `yarn dev` separately for this to work***

This will run the 'headless' version of Companion, without the red launcher window. If you want to run with that, you can cd into the `launcher` folder and run `yarn dev`. On WSL this may require additional dependencies to be installed.

TODO: How to compile a distribution for Windows from WSL (note that links and packages mentioned in the Wiki “Build for Another Device” are broken)