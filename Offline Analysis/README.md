**initramfs and LUKS Analysis**

**initramfs Extraction**

The initrd.img was extracted using cpio, revealing:
- cryptroot
- cryptroot-unlock
- cryptsetup
- /scripts/local-top/cryptroot
- /lib/cryptsetup/functions

Root unlocking logic was entirely handled inside initramfs.

A critical observation:

If the root filesystem failed to mount, the system dropped to an interactive shell.

**LUKS Inspection**

From the initramfs shell:

- cryptsetup luksDump /dev/sda2

Confirmed:

- LUKS2
- Argon2id PBKDF
- Valid metadata
  
The encrypted volume was already unlocked as:

- /dev/mapper/disk

Verification:

- cryptsetup status disk

Status showed active read/write mapping.

**Mounting the Root Filesystem**

The root filesystem was mounted manually:

- mount /dev/mapper/disk /mnt

This exposed the full system hierarchy:
- /etc
- /root
- /home
- /var
- /usr
- /lib
- /boot

The system was now fully accessible offline.

**Key Takeaways**
- initramfs is part of the trusted computing base
- GRUB kernel parameters are a critical attack surface
- Disk encryption does not protect against offline boot-chain modification\
- Removing root= is sufficient to trigger recovery logic

This completed the initial compromise phase and enabled full post-mount analysis.
