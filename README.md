env.kindle3wifi-ssh
===================

Everything needed (software and configurations) to access your Kindle 3 WiFi through SSH and sync files to/from it.

I got a Kindle 3 (WiFi-only version) and wanted to achieve what every linux user would: Get SSH access to it.
Also, with SSH arranged, I would like to be able to synchronize a `text` directory from my computer to the kindle,
so that all of my reading material would be ready to be used, whenever I wanted.

So, after some hours of diving deep into forums, I found that I could achieve my goals by taking some complicated steps.
I had to jailbreak the kindle, then install the `USBNetwork` hack, fiddle with network configuration and SSH server parameters in the Kindle.

This repository is intended to provide you with the needed tools and "pre-baked" configuration files, relieving you of much stress.
By applying the steps described below, your end result will be a kindle which is easily accessible via SSH (WiFi) using your favorite private key,
and a kindle into which you can synchronize a defined directory in your computer using the provided `kindle-rsync` script.

The steps to achieve the desired scenario
-----------------------------------------

### Getting SSH access to the kindle via WiFi

  1. Jailbreak your kindle. Use the file `kindle-jailbreak-<version>.*` in this repository.
     Carefully read and follow the instructions in that file.
       + Restart your kindle some 2 times after the jailbreak. Just to be sure ☺
  2. Apply the `USBNet` hack to the kindle, with the file `kindle-usbnetwork-<version>.*` in this repository.
     Carefully read and follow the instructions in that file.
       + Restart your kindle some 2 times after applying the `USBNetwork` hack. Just to be sure ☺
  3. Let's now create a connection between the kindle and your PC using USB and copy all "pre-baked" configurations to the kindle.
     This should be the last time that you need to connect to the kindle via USB.
       + On the kindle's home screen, press the `<delete>` key to invoke the search bar.
           - In the search bar, type `;debugOn` and hit enter.
           - Invoke search again, then type `~help`, hit enter, and check whether a help screen appears. If it does, proceed!
           - Invoke search again, type `~usbNetwork`, hit enter; invoke search yet again, type `;debugOff` and enter.
       + The kindle is ready to connect to your PC via USB. Let's do it:
           - Plug your kindle to the PC with a USB cable
           - In your PC there should be a new network device showing up in the last lines of `dmesg | tail`.
           - Disable all "regular" (wired and/or wireless) networking in your PC
           - Then type into a terminal (assuming the new device is `usb0`):
             `sudo ifconfig usb0 192.168.2.1`. Your PC then has assumed address `192.168.2.1`.
           - To test the connection, probe kindle's SSH port: `telnet 192.168.2.2 22`. Quit by `<Ctrl>+]` then `<Ctrl>+D`.
       + Copying all "pre-baked" config files to the kindle network configuration directory:
           - To remove all existing net config files in the kindle, do:
             `ssh root@192.168.2.2 'rm -rf /mnt/us/usbnet/etc'`.
           - Write the SSH public key with which you'll access the kindle to `<this-repo>/etc/dot.ssh/authorized_keys`.
             `cp <your-desired-ssh-public-key> <this-repo>/etc/dot.ssh/authorized_keys`
           - Copy "pre-baked" configuration files with: `scp -r <this-repo>/etc root@192.168.2.2:/mnt/us/usbnet/`
           - Enable networking module on kindle's boot: `ssh root@192.168.2.2 'mv /mnt/us/usbnet/DISABLE_auto /mnt/us/usbnet/auto'`.
  4. Gently terminate the ethernet connection between your PC and the kindle: `sudo ifconfig usb0 down`.
  5. Disconnect the USB cable.
  6. Restart the kindle some 2 times.
  7. DONE. You should now be able to access your kindle via SSH via WiFi with your favorite SSH private key.

### Easy synchronization of reading material between PC and kindle via SSH/WiFi

Now that you have SSH access to the kindle, it would be a good idea to setup a fixed IP (in your LAN) for the kindle.
There are several ways to do it; if you are in a typical home environment with a wireless router, you will need to do one of the following:

 1. In your router, create a DHCP rule with the MAC address of your kindle.
    Then your kindle can just connect to the WiFi with default settings, but will **always obtain the same address**.
 2. Setup your kindle to get a **static IP** from the WiFi access point.
    Remember, this IP for the kindle must be out of the DHCP range of your router, otherwise there could be IP conflicts.

Alternative 1 is almost surely the easiest.

Having the kindle's IP fixed, just link the script `<this-repo>/kindle-rsync` into `${HOME}/bin` or somewhere else in your PATH:

```bash
    ln -s -f "<this-repo>/kindle-rsync" "~/bin/kindle-rsync"
```

The script takes **1 parameter**, which is the **directory whose contents are to be synchronized with the Kindle**.
An environment variable called `KINDLE_ADDR` should be defined with the kindle's IP address whenever executing the script,
otherwise a "bogus" default address will be used.
So, you can define the variable in your `~/.profile` and execute the script like this:

```bash
    $ cat 'export KINDLE_ADDR=<kindle_ip>' >> ~/.profile
    $ export KINDLE_ADDR=<kindle_ip>
    $ kindle-rsync "~/Text/portable"
```

