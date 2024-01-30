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

[4. Password policies](#4-Password-policies)

[5. SSH Server](#5-SSH-Server)  

[6. UFW Firewall](#6-UFW-Firwall)

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
Repeat the steps of [point 2.](#2-User) and add the user to the group `sudo`:
~~~
adduser cfidalgo sudo
⭕ cat /etc/group | grep sudo
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


## 4. Password policies
To modify the configuration of the password policies we need to take two steps: First modify the `/etc/login.defs` file, and second of all install the `libpam-pwquality` package and modify it's configuration files.  

**What is /etc/login.defs? 🔑:** Is a file that contains the configuration control definitions for the login package, that is to say, the password configuration of all users in the system.  
**What is libpam-pwquality? 🔐:** Is a package that helps you configure hard policies for password in your system. It adds more policies and rules to make sure no user has weak passwords.

1. Modify /etc/login.defs and modify the password aging controls:  
   - Change the expiration day of the password: `PASS_MAX_DAYS 30`
   - Minimum days before changing your new password: `PASS_MIN_DAYS 2`
   - Warn the user 7 days before his password expires: `PASS_WARN_AGE 7`
~~~
PASS_MAX_DAYS 30
PASS_MIN_DAYS 2
PASS_WARN_AGE 7
~~~

2.1 Install `libpam-pwquality`:
~~~
apt install libpam-pwquality
~~~

2.2 Modify `/etc/security/pwquality.conf`:
   - Number of chars that differs from old password: `difok = 7`
   - Minimum length: `minlen = 10`
   - Minimum one uppercase letter: `ucredit = -1`
   - Minimum one digit: `dcredit = -1`
   - Maximum number of repeated chars: `maxrepeat = 3`
   - Force the root to follow this policies: `enforce_for_root`
   - If a password is invalid, don't accept it (make sure is not just a warning): `enforcing = 1`
~~~
difok = 7
minlen = 10
ucredit = -1
dcredit = -1
maxrepeat = 3
enforce_for_root
enforcing = 1
~~~

![image](https://github.com/ChristianFidalgoAreste/Born2beroot/assets/113194238/11ed4012-7715-4e9b-bd98-2cf47763cdd6)
![image](https://github.com/ChristianFidalgoAreste/Born2beroot/assets/113194238/827e1da1-54ec-4f76-a159-b407c206535f)
![image](https://github.com/ChristianFidalgoAreste/Born2beroot/assets/113194238/5ee6277e-0801-4483-ac57-ec109caab442)
![image](https://github.com/ChristianFidalgoAreste/Born2beroot/assets/113194238/4e436736-736c-466c-8405-995a9498e414)


## 5. SSH Server
To be able to connect to the server via SSH we need to install OpenSSH-server and modify the configuration files with small changes to change the port and prohibit the root connection.

**What is SSH (Secure Shell)? 🐢:** Is a protocol to connect to a server in a secure channel. It works with private and public keys combination, where the server store the private key and the client has the public key. When a secure channel opens, the server compares both keys and if the "puzzle" resolves, it accepts the connection. All messages are encrypted with this combination, so is secure to send sensible data, as long as the public key is correct (otherwise the decryption won't work). Nonetheless, the public key can't be anywhere if you **dont't have another security middleware**, like user and password. If not, anybody with the public key (probably a lot of people) could join your server!  

1. Install OpenSSH-server:
~~~
apt install openssh-server
~~~

2. Modify the `port` in `/etc/ssh/ssh_config` (client configuration):
~~~
Port 4242
~~~

3. Modify the `port` and deny root access in `/etc/ssh/sshd_config` (server configuration):
~~~
Port 4242
PermitRootLogin no
~~~

4. Restart SSH service to apply changes:
~~~
sercice ssh restart
~~~

![image](https://github.com/ChristianFidalgoAreste/Born2beroot/assets/113194238/f16961c9-425f-47c1-bede-cd3802c8839a)
![image](https://github.com/ChristianFidalgoAreste/Born2beroot/assets/113194238/db521238-c38f-4e81-87cf-232f43440427)
![image](https://github.com/ChristianFidalgoAreste/Born2beroot/assets/113194238/3b5bf0c3-d234-4d64-9263-0afbc555c5e1)


## 6. UFW Firewall  
**What is a Firewall? 🔥🧱:** Is a security system of the network of a computer or server. This software allows and denies explicit connections packages and ports, so the system can allow exact connections to other services (only SSH, only web, only web and SSH...), and deny the others. This way you make sure only the necessary connections and services are allowed to the system, and no other ports / connections are allowed, so no one can connect wherever they want.  
UFW is a Firewall (Uncomplicated Firewall) with a really easy API and really easy to use and configure.

1. Install UFW:
~~~
apt install ufw
~~~

2. Enable UFW:
~~~
ufw enable
~~~

3. Add a rule to allow port 4242:
~~~
ufw allow 4242
~~~

4. Verify that UFW is enabled and with the rule active (2 ways):
~~~
ufw status
ufw status verbose
~~~

If the status is `status: active` and the rule is `4242 ALLOW Anywhere` it's correct.




















