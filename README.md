# gen_initramfs_efibootmgr
Gentoo Initramfs & EFI Stub Generator

"No room for LUKS headers? No problem! I'll just put them in their own partition like a responsible adult. So here we are, ditching GRUB entirely and letting the kernel handle its own boot process like a grown-up."

Welcome to a comedy of errors! I had a problem, or a few - I wanted to implement disk encryption post-Gentoo-install on my ancient laptop, but I had created my partitions with the XFS filesystem which, like my ex, refuses to shrink - no way to make room for LUKS headers on the root partition (sda3). However, I was able to spare 30MB to create a new partition for detached headers (sda4), so I encrypted. Afterwards, I found that Grub didn't support detached headers. In Grub 2.12, that feature was implemented, but I still haven't figured out how to get it to work. Additionally, Grub still (as of this writing) only supports the weaker PBKDF2 algorithm instead of the stronger Argon2id algorithm. 

The scripts:
- build-initramfs: Creates a custom initramfs at /root/initramfs/ that actually understands the concept of detached headers
- update_efibootmgr: Shoves said initramfs into your EFI partition and tells your BIOS "Hey, this is a bootloader now"
- test_modules: Half-baked test script to see if modules would load in a chroot, since I was tired of rebooting and running my recovery script to mount everything. I haven't bothered to make it compatible across distribution and custom kernels (it assumes all modules live under lib/uname -r, which can be a bad assumption to make in a chroot). It sets up a mini-environment to test modprobe commands that the init script will run. It outlived its usefulness, for now
- recovery: A basic script to unlock, bind mount, and chroot in from a LiveUSB since there's no Grub-like rescue kernel selection with plain old EFI
- files/init: The big cheese, a simple busybox-based init script. There's an emergency shell included because let's face it, something will go wrong and you'll need to debug it at 3 AM

Credit to https://github.com/smkent/initramfs for the initial framework - see his repo for neat things like remote unlocking via SSH.

## Architecture Notes
You'll probably need to customize this for your hardware unless you also have a 2012 Asus K55A lying around. The code is commented like a nervous programmer before a code review, but still, 'good luck everybody else' (and future me).

You'll need:
- UEFI-capable system
- Kernel configured with EFI stub and LUKS/device-mapper support
- Root filesystem on a LUKS volume (ideally with detached headers)

The most likely things you'll need to change:
- Module list in build-initramfs for your storage hardware
- Module load order in required_modules array in build-initramfs and files/init modprobe forloop if you have different dependencies
- Device paths for your partition layout
