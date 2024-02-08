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

[6. UFW Firewall](#6-UFW-Firewall)

[7. Script](#7-Script)

[8. Bonus](#8-Bonus)

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
  
**⚠️ Warning ⚠️:** The users that were created before the installation of the password policies are **NOT** following all the policies (they are not following the `PASS_MAX_DAYS`, `PASS_MIN_DAYS` and `PASS_WARN_AGE`), including the *root* user. To make sure all the previously existing users are following these policies, you can check it with `chage -l user`.

![image](https://github.com/ChristianFidalgoAreste/Born2beroot/assets/113194238/a4b3c4bf-b587-4dd6-bbfe-4e908d194e1d)

The image shows that the *root* user is not following the current policies. You can change it with the command `chage -m 2 -M 30 user`, where the flag `-m` is the minimum days policy and the `-M` the maximum days policy.ç

![image](https://github.com/ChristianFidalgoAreste/Born2beroot/assets/113194238/fffc74e0-b1cd-4bc1-8976-7684c2a6655a)
![image](https://github.com/ChristianFidalgoAreste/Born2beroot/assets/113194238/79567802-2db7-4678-9065-404bd46f6bb0)


## 5. SSH Server
To be able to connect to the server via SSH we need to install OpenSSH-server and modify the configuration files with small changes to change the port and prohibit the root connection.

**What is SSH (Secure Shell)? 🐢:** Is a protocol to connect to a server in a secure channel. It works with private and public keys combination, where the server store the private key and the client has the public key. When a secure channel opens, the server compares both keys and if the "puzzle" resolves, it accepts the connection. All messages are encrypted with this combination, so is secure to send sensible data, as long as the public key is correct (otherwise the decryption won't work). Nonetheless, the public key can't be anywhere if you **don't have another security middleware**, like user and password. If not, anybody with the public key (probably a lot of people) could join your server!  

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

![image](https://github.com/ChristianFidalgoAreste/Born2beroot/assets/113194238/ffc1006a-50b7-4272-9701-fd3920531c32)



## 7. Script
For the script, we will separate it in 2 ways. First, we will create the script, explaining every command used and how it works. It's important that you understand every command used, as it's asked on the defense. Then, we will configure **Crontab**, a **cron** configuration file, a tool that allows us to program repetitive tasks along the server in background.

1. Script:
First, create a `monitoring.sh` script with nano in the `/root` directory (it can be elsewhere, but this directory is exclusive for root, so you prevent not-allowed modifications):
~~~
nano /root/monitoring.sh
~~~

2. Add the content of the script, creating variables for every command and showing it's value in the `wall` command (A command to show in every terminal of the server the same formatted message):
 - **Arch**: With the command `uname -a` we can see all the information of the architecture. The parameter `-a` lists ALL the information, while other parameters like `-v` just lists the version (Debian).
 ![image](https://github.com/ChristianFidalgoAreste/Born2beroot/assets/113194238/b97330e8-1e06-44d5-9e32-06d7d44ee35a)

 - **CPU Physical**: The `/proc/cpuinfo` file contains information about the CPU. The `physical id` refers to the id of every physical CPU in the system, starting by 0. That means that we can count how many times it appears to know how many physicals CPU there are. We use `grep "physical id" | wc -l` for that.
 ![image](https://github.com/ChristianFidalgoAreste/Born2beroot/assets/113194238/a061cc15-8629-48b2-b5f1-fa6f138eadd4)

 - **Virtual CPU**: The same as before, but this time counting the `processor` line. We count how many times the line `processor` appears.
 ![image](https://github.com/ChristianFidalgoAreste/Born2beroot/assets/113194238/677fa1e1-cfe9-471b-b75f-475844faee92)

 - **Memory usage**: The command `free` lists information about the memory. With the parameter `--mega` (not -m) we can list this information in MB (-m is for Mebibytes, not Mega). As the information appears by columns, we need to use the command `awk` to filter this info. With the `FNR` we can access to the nth row (index of the row), and with the `{print $x}` we can print the `$x` column (index of column). The `$3` col is the memory used, the `$2` col is the total memory, and to print the percentage, we use `printf()` to print formatted (`%.2f` stands for 2 decimals) text, and we calculate it with **memUsed * 100 / memTotal**.
![image](https://github.com/ChristianFidalgoAreste/Born2beroot/assets/113194238/e5edc8e7-736f-4977-a91a-2bdf89beb9ab)

 - **Disk usage**: The comand `df` sum up information about the disk (disk filesystem), and we use the flag `-m` to list it in MB. We filter the info only to `/dev/` and exclude (with the flag **-v**) the `/boot`. Then, for every item in the `$x` column (`$3` for usage and `$2` for total), you add the value to a variable and then print the result with `awk` (and for the percentage we use the same formula as in "Memory usage"):
![image](https://github.com/ChristianFidalgoAreste/Born2beroot/assets/113194238/3bf24c0c-b4cb-4f6e-8719-2478978fface)

 - **CPU Load**: With the command `vmstat`, we can see different system statistics, like CPU activity, memory used, etc. Then, with `tail -1` we can get only the last output. Finally, we print the 15th column, that contains the information of usage of CPU. We need to substract this result to 100 to get the actual use of CPU, and format it to 1 decimal with `printf` and `expr`:
![image](https://github.com/ChristianFidalgoAreste/Born2beroot/assets/113194238/d26116bf-1032-4b1f-a2ee-932b2022d90f)

 - **Last boot**: With `who -b` we can see the information of the last boot. To parse the information, we extract from the frist row (`FNR`) the 3rd and 4rth column, separate with space (`$3 " " $4`):
![image](https://github.com/ChristianFidalgoAreste/Born2beroot/assets/113194238/21c066c9-ce9a-4d01-a408-2079031dc4e8)

 - **LVM Use**: We extract how many lines are with "lvm" on the output of `lsblk`, that lists information about the volumes and partitions. Using and `if` statement, we look if the number of lines is greater than 0. If it is, we `echo` "yes", otherwise we `echo` "no":
![image](https://github.com/ChristianFidalgoAreste/Born2beroot/assets/113194238/211c2605-4ee0-4fed-9074-de8b64bfc545)

 - **TCP Connections**: With the command `ss` we can list information about the network and connections (as using `netstat`). With the flag `-ta`, we can filter for TCP only. Then, we `grep` the established only ("ESTAB"), and we count the lines.
![image](https://github.com/ChristianFidalgoAreste/Born2beroot/assets/113194238/11fe0ebf-b262-4d96-802a-d54e0651c64e)

 - **User log**: The `users` command lists all the connected users. Using `wc -w` we can count the words of the output. This way we are counting the different users.
![image](https://github.com/ChristianFidalgoAreste/Born2beroot/assets/113194238/3fe763f8-a2fb-4182-a86f-1cdc812b14be)

 - **Network**: To get the IP, we can simply use `hostname -I`. To get the MAC address, we can use `ip link`, to show the network interfaces. We `grep` the "link/ether" and print the second column with `awk`:
![image](https://github.com/ChristianFidalgoAreste/Born2beroot/assets/113194238/6619e8fc-54ce-41d7-9afa-1199bc55b02b)

 - **Sudo**: To get the number of sudo commands executions we can use `journalctl`, a tool that lists all the system's registers. With `_COMM=sudo` we filter with executable scripts that match with sudo. Then, we `grep` "COMMAND" to filter the command one's, and finally we count how many lines there are with `wc -l`:
![image](https://github.com/ChristianFidalgoAreste/Born2beroot/assets/113194238/b6b1a390-3701-49cd-ae57-15aa453d72d9)

Script:
~~~
#!/bin/bash

#Arch
architecture=$(uname -a)

#CPU Physical
cpuPhys=$(grep "physical id" /proc/cpuinfo | wc -l)

#vCPU
vCPU=$(grep "processor" /proc/cpuinfo | wc -l)

#Memory usage
memUsed=$(free --mega | awk 'FNR == 2 {print $3}')
memTotal=$(free --mega | awk 'FNR == 2 {print $2}')
memPerc=$(free --mega | awk 'FNR == 2 {printf("%.2f"), $3*100/$2}')

#Disk Usage
diskUsed=$(df -m | grep "/dev/" | grep -v "/boot" | awk '{disk_us += $3} END {print disk_us}')
diskTotal=$(df -m | grep "/dev/" | grep -v "/boot" | awk '{disk_tot += $2} END {printf("%.1fGb"), disk_tot/1024}')
diskPerc=$(df -m | grep "/dev/" | grep -v "/boot" | awk '{disk_us += $3} {disk_tot += $2} END {printf("%d"), disk_us*100/disk_tot}')

#CPU load
cpuLoad=$(vmstat 1 2 | tail -1 | awk '{print $15}')
cpuParsed=$(printf "%.1f" $(expr 100 - $cpuLoad))

#Last boot
lastBoot=$(who -b | awk 'FNR == 1 {print $3 " " $4}')

#LVM use
lvmUse=$(if [ $(lsblk | grep "lvm" | wc -l) -gt 0 ]; then echo yes; else echo no; fi)

#TCP Connections
tcpConn=$(ss -ta | grep "ESTAB" | wc -l)

#User log
userLog=$(users | wc -w)

#Network
ip=$(hostname -I)
mac=$(ip a | grep "link/ether" | awk '{print $2}')

#Sudo
sudo=$(journalctl _COMM=sudo | grep COMMAND | wc -l)

wall " Architecture: $architecture
CPU Physical: $cpuPhys
vCPU: $vCPU
Memory Usage: $memUsed/${memTotal}MB ($memPerc%)
Disk Usage: $diskUsed/${diskTotal} ($diskPerc%)
CPU Load: $cpuParsed%
Last boot: $lastBoot
LVM use: $lvmUse
TCP Connections: $tcpConn Established
User log: $userLog
Network: IP $ip ($mac)
Sudo: $sudo cmd"
~~~

3. Create the temporized task with crontab. Edit the crontab file with the next command:
~~~
crontab -e -u root 
~~~

And add the next line (maybe you need to change the script path):
~~~
*/10 * * * * sh /root/monitoring.sh
~~~

This line follows a specific syntax:
 - Minutes (0 - 59): The task repeats every 'X' minute. `*` represents no specific minute, and `*/X` represents every X minutes.
 - Hour (0 - 23): The task repeats every 'X' hour. `*` represents no specific hour, and `*/X` represents every X hours.
 - Day of the month (1 - 31): The task repeats every 'X' day. `*` represents no specific day, and `*/X` represents every X days.
 - Month of the year (1 - 12): The task repeats every 'X' month. `*` represents no specific month, and `*/X` represents every X months.
 - Day of the week (0 - 7): The task repeats every 'X' day of the week (monday, tuesday...).

4. Finally, we give permissions to the script so crontab can execute it (change the path of the script to your needs):
~~~
chmod ugo+x /root/monitoring.sh
~~~

![image](https://github.com/ChristianFidalgoAreste/Born2beroot/assets/113194238/e33cfeec-31c0-479a-9ed7-5536b9c95d4f)

## 8. Bonus





