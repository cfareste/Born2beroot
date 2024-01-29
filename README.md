# Born2beroot
### A project designed to install and configure a server without any GUI.

If you want to look how to do the partitions part (initial installation and first bonus), or you want to know how to do the project on VMWare, please read the first part of [gemartin's guide](https://github.com/gemartin99/Born2beroot-Tutorial). My own guide differs in some ways with his, as I did this one investigating on my own, and I didn't document the initial installation and the partitions part. However, for the rest of the project you will find good explanations and learn properly how everything works, and more important, you will understand what are we using and why we are using it, and understand more concepts out of the subject to do a good defense of the project.
**⚠ Important ⚠:** If is not tell otherwise, every command should be running as **root** (sometimes it doesn't even work with `su`, it has to be root user!)

## Index
[1. AppArmor]
---


## 1. AppArmor
**What is AppArmor (and / or SELinux)?🛡** AppArmor is a security module of Linux kernel that helps the system administrator to restrict any application's or program's capabilities. To do so, it assigns every app a security profile, some of which have more privileges than others. This helps to ensure that no program can do operations with high privileges without consent.

On Debian, AppArmor comes by default, and is already activated. To be sure that it is, we can use the next command:
    cat /sys/module/apparmor/parameters/enabled
If the output is a `Y`, its activated, otherwise is not.

To activate or deactivate, we can type on the terminal `security=XXXX`, where `XXXX` is the name of the security module (AppArmor, SELinux...).
As the subject says it has to be activated on boot (remember that on Debian it is already activated on boot), we can use the next commands:
1. First, we create a directory for GRUB Bootloader on `/etc/default`:
    mkdir -p /etc/default/grub.d
2. Add on a file `apparmor.cfg` a flag to activate apparmor on boot:
    echo 'GRUB_CMDLINE_LINUX_DEFAULT="$GRUB_CMDLINE_LINUX_DEFAULT apparmor=1 security=apparmor"' | sudo tee /etc/default/grub.d/apparmor.cfg
3. Update GRUB Bootloader:
    update-grub
4. Reboot:
    reboot
