**Initial Access Vector – Boot Chain Manipulation
Context**

The target system was a Debian 13–based virtual machine that booted successfully but halted at a login prompt. No valid credentials were available, network access was non-functional, and brute force or credential reuse was explicitly out of scope. An offline backup of the server was available. The strategic decision was made to pivot from live interaction to offline disk analysis.

**Strategic Pivot**

Instead of attacking the running VM, the encrypted disk image was attached to a Kali Linux VM as a secondary drive.

**Goals:**

- Identify boot configuration weaknesses
- Analyze encryption handling
- Evaluate initramfs behavior
- Determine whether disk encryption could be bypassed offline

**Disk Layout**

The target disk appeared as:

- /dev/sda1  → Unencrypted /boot partition
- /dev/sda2  → LUKS-encrypted root filesystem

The boot partition contained:

- vmlinuz-*
- initrd.img-*
- grub/

This confirmed that GRUB and initramfs were viable attack surfaces.

**GRUB Parameter Manipulation**

The original kernel boot line resembled:

- linux /vmlinuz-6.12.63+deb13-amd64 root=/dev/mapper/disk ro noresume

It was modified to:

- linux /vmlinuz-6.12.63+deb13-amd64 ro init=/bin/sh console=tty1

Additionally:

- The root= parameter was removed
- Recovery entries were modified consistently

This forced the kernel to:

- Boot into initramfs
- Fail root filesystem mounting
- Trigger fallback shell behavior

Result

On boot, the system dropped into:

- (initramfs)

This shell:

- Ran as uid=0 (root)
- Had access to /dev
- Included cryptsetup

At this stage, the trusted boot boundary was broken.
