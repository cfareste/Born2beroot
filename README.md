# Born2beroot
### A project designed to install and configure a server without any GUI.

If you want to look how to do the partitions part (initial installation and first bonus), or you want to know how to do the project on VMWare, please read the first part of [gemartin's guide](https://github.com/gemartin99/Born2beroot-Tutorial). My own guide differs in some ways with his, as I did this one investigating on my own, and I didn't document the initial installation and the partitions part. However, for the rest of the project you will find good explanations and learn properly how everything works, and more important, you will understand what are we using and why we are using it, and understand more concepts out of the subject to do a good defense of the project.  

**⚠️ Important ⚠️:** If is not tell otherwise, every command should be running as **root** (sometimes it doesn't even work with `su`, it has to be root user!)  

**🗒️ Note 🗒️:** If you see this emoji "⭕" it means that the information you are reading is worth knowing for the defense, as one question of the defense is based on that knowledge.


## Index
[1. AppArmor](#1-AppArmor) 

[2. User](#2-User)  

[3. Sudo](#3-Sudo)  

&ensp;&ensp;[3.1 Sudo policies](#31-Sudo-policies)  

---


## 1. AppArmor
**What is AppArmor (and / or SELinux)?🛡** AppArmor is a security module of Linux kernel that helps the system administrator to restrict any application's or program's capabilities. To do so, it assigns every app a security profile, some of which have more privileges than others. This helps to ensure that no program can do operations with high privileges without consent.

On Debian, AppArmor comes by default, and is already activated. To be sure that it is, we can use the next command:  
~~~
⭕cat /sys/module/apparmor/parameters/enabled
~~~
If the output is a `Y`, its activated, otherwise is not.
![image](https://github.com/ChristianFidalgoAreste/Born2beroot/assets/113194238/782f3e67-c7b3-4d70-bfd2-64216c0c9e09)


To activate or deactivate, we can type on the terminal `security=XXXX`, where `XXXX` is the name of the security module (AppArmor, SELinux...).
As the subject says it has to be activated on boot (remember that on Debian it is already activated on boot), we can use the next commands:
1. First, we create a directory for GRUB Bootloader on `/etc/default`:
~~~
mkdir -p /etc/default/grub.d
~~~
2. Add on a file `apparmor.cfg` a flag to activate apparmor on boot:
~~~
echo 'GRUB_CMDLINE_LINUX_DEFAULT="$GRUB_CMDLINE_LINUX_DEFAULT apparmor=1 security=apparmor"' | sudo tee /etc/default/grub.d/apparmor.cfg
~~~
3. Update GRUB Bootloader:
~~~
update-grub
~~~
4. Reboot:
~~~
reboot
~~~


## 2. User
Before we install anything, we need to create the group `user42`:
~~~
addgroup user42
~~~
Now add the user to this group:
~~~
adduser cfidalgo user42
~~~
⭕ To check if the group was created succesfully we can use:
~~~
cat /etc/group | grep user42
~~~
![image](https://github.com/ChristianFidalgoAreste/Born2beroot/assets/113194238/50cbe33f-bcb8-47b4-88e1-c3455d1fcfe8)


## 3. Sudo
Install sudo with `apt install sudo` and reboot just in case (`reboot`).
Repeat the steps of [point 2.](#2.-User) and add the user to the group `sudo`:
~~~
adduser cfidalgo sudo
⭕cat /etc/group | grep sudo
~~~
![image](https://github.com/ChristianFidalgoAreste/Born2beroot/assets/113194238/6d29080a-a969-4378-b10f-8be352055791)

### 3.1 Sudo policies
You can modify the sudo policies (password retries, message, etc.) modifying the main `sudoers` file (sudo configuration file) with `sudo visudo`.  
However, is more practical and better practice to create an own small configuration file on the `/etc/sudoers.d` directory, as it's more structured, well-organized, and directly imported in the main file.  
🗒️ `visudo` is a text editor really similar to nano, but it checks for sudo configuration syntax, to prevent errors if the file is wrong configured.  

To create an own sudo configuration file, type:
~~~
nano /etc/sudoers.d/sudoconf
~~~
Note that you can change the name `sudoconf` to whatever you like, as it's imported directly just to be in `sudoers.d` directory.  
Inside this file, we will add the next configuration:  
 - Limit password's tries to 3: `Defaults    passwd_tries=3`
 - Personal message for bad password: `Defaults    badpass_message="Wrong password, try again"`
 - Save input and output of the message in the directory set (next point): `Defaults    log_input, log_output`
 - Set the directory for sudo logs: `Defaults    iolog_dir="/var/log/sudo"`
 - Require to be in a tty to use sudo: `Defaults    requiretty"`
 - Limit the paths that sudo can use: `Defaults    secure_path="/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/sbin:/bin:/snap/bin"`

Result (make sure you use tabs):
~~~
Defaults    passwd_tries=3
Defaults    badpass_message="Wrong password, try again"
Defaults    log_input, log_output
Defaults    iolog_dir="/var/log/sudo"
Defaults    requiretty
Defaults    secure_path="/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/sbin:/bin:/snap/bin"
~~~
![image](https://github.com/ChristianFidalgoAreste/Born2beroot/assets/113194238/a47829c1-ad6d-4932-a292-a64f06a6fcf8)

![image](https://github.com/ChristianFidalgoAreste/Born2beroot/assets/113194238/1106d53f-6d10-49bc-aae6-d4b75fb3c8df)


## 4. Password
