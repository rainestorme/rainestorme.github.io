# Installing MrChromebox's RW_LEGACY payload on `kindred`

> Heads up! This post is mainly for my own reference, and I only put it here incase someone else can benefit from it if their board isn't supported officially by MrChromebox's script.

There I was. I had just bricked my Chromebook for the third time after installing a developer version of murkmod, and I was just about fed up. I wanted to use nmap, but that was only acessible through Crouton and I had just wiped my installation. Fuuuck. After some consideration, I decided to just boot an Arch Linux USB that I had sitting around, and thus I opened a terminal and ran MrChromebox's script.

![image](https://github.com/rainestorme/rainestorme.github.io/assets/115757568/353cd0c0-cd9b-47d6-bc91-ee69bcf13a0d)

![image](https://github.com/rainestorme/rainestorme.github.io/assets/115757568/810e85a8-7bfb-4259-8045-41a951964127)

Huh, that's weird. Other `hatch` devices are supported, why isn't this one? Maybe it's just a bug...

![image](https://github.com/rainestorme/rainestorme.github.io/assets/115757568/1296f070-b728-494b-994d-6a30444ad9c8)

Nope, it's unsupported. Let me look at the source...

![image](https://github.com/rainestorme/rainestorme.github.io/assets/115757568/29e8b74f-3769-43ad-8b79-76d39de861b7)

Huh?!? It's right there! Why wouldn't that work?

![image](https://github.com/rainestorme/rainestorme.github.io/assets/115757568/40244146-47d6-40c2-959d-caf140e012a0)

Ahh... In the logic, it only works with Chromeboxes. I'm not entirely sure if this is a bug, but oh well. Let's download and verify the firmware...

```bash
curl -LOk https://mrchromebox.tech/files/firmware/rw_legacy/rwl_altfw_cml-mrchromebox_20210415.bin
curl -LOk https://mrchromebox.tech/files/firmware/rw_legacy/rwl_altfw_cml-mrchromebox_20210415.bin.md5
md5sum rwl_altfw_cml-mrchromebox_20210415.bin
cat rwl_altfw_cml-mrchromebox_20210415.bin.md5
```

As long as the outputs of the last 2 commands match, then you're good to go. Now, we just flash it to the RW_LEGACY section...

```bash
flashrom -w -i RW_LEGACY:rwl_altfw_cml-mrchromebox_20210415.bin
```

If that command outputs success, then all you should need to do is reboot the system and everything will work!
