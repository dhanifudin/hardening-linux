# Hardening Linux

## Linux Hardening Checklist: System Installation and  Patching

- Set sticky bit on all world writable directories

  Setting the sticky bit on world writable directories prevents users from
  deleting or renaming files in that directory that are not owned by them. This
  feature prevents the ability to delete or rename files in world writable
  directories (such as /tmp) that are owned by another user.

  ```
  chmod 1777 /tmp
  ```
- Automatic Updates

1. Update the server, run:
  ```
  sudo apt update && sudo apt  upgrade
  ```
2. Install unattended upgrades on Ubuntu.  Type the following command
  ```
  sudo apt install unattended-upgrades apt-listchanges bsd-mailx
  ```
3. Turn on unattended  security updates, run then select **Yes**
  ```
  sudo dpkg-reconfigure  -plow unattended-upgrades
  ```
4. Configure automatic updates, enter:
  ```
  sudo  vi /etc/apt/apt.conf.d/50unattended-upgrades
  ```
5. Set up  alert email ID:
  ```
  Unatended-Upgrade::Mail  "sysadmin@yourcompany.com";
  ```
6. Automatically reboot Ubuntu box  WITHOUT CONFIRMATION  for kernel updates
  ```
  Unatended-Upgrade::Automatic-Reboot "true";
  ```
7. Finally edit the /etc/apt/listchanges.conf and set email  ID:
   ```
   email_address=sysadmin@yourcompany.com
   ```
8. Verify that it is working  by running the following command:
  ```
  sudo unattended-upgrades  --dry-run
  ```

## Disabling Unnecessary Services

Command to stop a service
  ```
  sudo systemctl stop [service]
  ```

Command to disable [service]
  ```
  sudo systemctl disable [service]
  ```

Command to kill a process
  ```
  sudo kill -9 [process_id]
  ```

## OS Hardening

- Restrict Core Dumps

1. Open the terminal app and login using the ssh command for remote cloud server
2. Then edit the `/etc/security/limits.conf`
3. Append the following lines
  ```
  hard core 0
  soft core 0
  ```
4. Make sure the Linux prevents setuid and setgid programs from dumping core to.
   Edit the following file: `/etc/sysctl.d/9999-disable-core-dump.conf` or
   `/etc/sysctl.conf` Then append:
   ```
   fs.suid_dumpable=0
   kernel.core_pattern=!/bin/false
   ```
5. Save and close the file. Finally run the `sudo sysctl -p /etc/sysctl.d/9999-disable-core-dump.conf`

- Remove legacy services
- Disable any services ans applications starteddd by `xinetd` or `inetd` that
  are not being utilizedd. Remove xinetd
- Disable or remove server services (FTP, DNS, LDAP, NFS, SNMP etc) that are not
  used
- Ensure syslog (rsyslog, syslog, syslogng) service is running
- Enable a network time protocol (NTP) service to ensure clock accuracy
- Restrict the use of the cron and at services

## Enforce Strong Password Management
## Restrict User From Using Previous Passwords
## Ensure No Accounts Have Empty Passwords
  ```
  awk -F: '($2 == "") {print}' /etc/shadow
  ```
## Disable Unnecessary Accounts
  Command to view users who have been inactive from past 90 days
  ```
  lastlog -b 90 | tail -n+2 | grep -v 'Never logged in'
  ```

  Command to disable a user
  ```
  usermod -L [user]
  ```

## Create Separate Disk Partitions for Linux System
- Separate OS files from user files for higher data security
- Ensure that the following file systems are mounted on separate partitions
  - /usr
  - /home
  - /var and /var/tmp
  - /tmp
- Create separate partitions for **Apache** and **FTP** server roots
- Edit and update following configuration settings in /etc/fstab file
  - noexec: Do not set execution of any binaries on this partition (prevents
    execution of binaries but allows scripts).
  - nodev: Do not allow character or special devices on this partition (prevents
    use of device files such as zero, sda, etc)
  - nosuid: Do not set SUID/SGID access on this partition (prevent the setuid bit)

  Example of `/etc/fstab`

  ```
  /dev/sda04    /ftpdata ext3   defaults,nosuid,nodev,noexec 1 2
  ```
## Remove or Rectify Permissions for World Writable Files
- View World Writable Files Without Sticky Bit
  ```
  find /home/alice -xdev -type d \( -perm -0002 -a ! -perm -1000 \) -print
  ```
- View no owner files
  ```
  find /home/alice -xdev \( -nouser -o -nogroup \) -print
  ```

## Disable USB Storage
- Disable USB Storage using the system BIOS Configuration option
- Disable kernel support for USB via GRUB
- In Debian based distribution, block usb-storage module from loading into Linux
  kernel
  ```
  cd /lib/modules/<kernel version/kernel/drivers/usb/storage
  sudo mv usb-storage.ko usb-storage.ko.blacklist
  ```
- Red Hat debian based distribution, disable USB storage with using fake install
  or blacklist usb-storage
  ```
  # modify file at /etc/modprobe.d/blacklist.conf
  vi /etc/modprobe.d/blacklist.conf
  # put blacklist statement
  blacklist usb-storage
  ```
