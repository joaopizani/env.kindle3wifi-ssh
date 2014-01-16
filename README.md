env.kindle3wifi-ssh
===================

Everything needed (software and confs) in order to access your Kindle 3 WIFI through SSH and sync files.

I got a Kindle 3 (WiFi only version) and wanted to achieve what every linux user would:
Get SSH access to it.
Also, with SSH arranged, I would like to be able to synchronize a `text` directory from my computer to the kindle,
so that all of my reading material would be ready to be used, whenever I wanted.

So, after some hours of diving deep into forums, I found that I could achieve my goals by taking some complicated steps.
I had to jailbreak the kindle, then install a USBNetwork hack, then fiddle with network configs and SSH server configs in the Kindle.

This repository is intended to provide you with the needed tools and "pre-baked" config files, relieving you of much stress.
By applying the steps described below, your end result will be a kindle which is easily accessible via SSH (WiFi) using your favorite private key,
and a kindle into which you can synchronize a defined directory in your computer using the provided `kindle-rsync` script.

The steps to achieve the desired scenario
-----------------------------------------

### Getting SSH access to the kindle via WiFi

  1. Jailbreak your kindle. Use the file `kindle-jailbreak-<version>.*` in this repo.
     Carefully read and follow the instructions in that file.
       + Restart your kindle some 2 times after the jailbreak. Just to be sure :)
  2. Apply the _USBNet_ hack to the kindle, with the file `kindle-usbnetwork-<version>.*` in this repo.
     Carefully read and follow the instructions in that file.
       + Restart your kindle some 2 times after applying the _USBNetwork_ hack. Just to be sure :)
  3. Let's now create a connection between the kindle and your PC using USB and copy all "pre-baked" config files over.
     This should be the last time that you need to connect to the kindle via USB.
       + On the kindle's home screen, press the `<delete>` key to show the search bar.
           - In the search bar, type `;debugOn` and hit enter.
           - Invoke search , then type `~help`, hit enter, and see if a help screen appears. If it does, GREAT!
           - Invoke search, type `~usbNetwork`, hit enter; invoke search, type `;debugOff` and enter again.
       + The kindle is ready. Now for the connection with your PC:
           - Connect your kindle to the PC with a USB cable
           - In your PC there should be a new network device showing up in the last lines of `dmesg | tail`.
           - Disable all "regular" (wired and/or wireless) networking in your PC
           - Then type into a terminal (assuming the new device is `usb0`):
             `sudo ifconfig usb0 192.168.2.1`. Your PC assumed address `192.168.2.1`.
           - To test, probe kindle's SSH port: `telnet 192.168.2.2 22`. Quit by `<Ctrl>+]` then `<Ctrl>+D`.
       + Copying all "pre-baked" config files to the kindle network configuration directory:
           - To remove all existing net config files in the kindle, do:
             `ssh root@192.168.2.2 'rm -rf /mnt/us/usbnet/etc'`.
           - Append your favorite SSH public key to `<this-repo>/etc/dot.ssh/authorized_keys`.
           - Copy "pre-baked" config files with: `scp -r <this-repo>/etc root@192.168.2.2:/mnt/us/usbnet/`
           - Enable on boot: `ssh root@192.168.2.2 'mv /mnt/us/usbnet/DISABLE_auto /mnt/us/usbnet/auto'`.
  4. Gently terminate the ethernet connection between your PC and the kindle: `sudo ifconfig usb0 down`.
  5. Disconnect the USB cable.
  6. Restart the kindle some 2 times.
  7. DONE. You should now be able to access your kindle via SSH via WiFi with your favorite SSH private key.

### Easy synchronization of reading material between PC and kindle via SSH/WiFi

Now that you have SSH access to the kindle, it would be a good idea to somehow setup a fixed IP (in your LAN) for the kindle.
There are several ways to do it; if you are in a typical home environment with a wireless router, probably you will need to do one of the following:

  * Setup your kindle to get a **static IP** from the WiFi access point.
    Remember, this IP for the kindle must be out of the DHCP range of your router, otherwise there could be IP conflicts.
  * In your router, create a DHCP rule with the MAC address of your kindle.
    Then your kindle can just connect to the WiFi with default settings, but will **always obtain the same address**.

Having the kindle's IP setup, just link the script `<this-repo>/kindle-rsync` into `${HOME}/bin` or somewhere else in your PATH:

    ln -s -f "<this-repo>/kindle-rsync" "~/bin/kindle-rsync"

The script takes **1 parameter**, which is the **directory whose contents are to be synchronized with the Kindle**.
An environment variable called `KINDLE_ADDR` should be defined with the kindle's IP address whenever executing the script,
otherwise a "bogus" default address (the address of the kindle on MY LAN) will be used.
So, a typical execution of the script would look similar to this:

    $ export KINDLE_ADDR=<kindle_ip>
    $ kindle-rsync "~/Text/portable"

Of course, you can include `KINDLE_ADDR` in your `~/.profile` and have even more convenience!

