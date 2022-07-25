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

## Configure Sysctl to Secure Linux Kernel

Edit `/etc/sysctl.conf` to:
- Restrict network-transmitted configuration for IPv4
- Restrict network-transmitted configuration for IPv6
- Turn on execshield protection
- Prevent syn flood attack
- Turn on source IP address verification
- Prevent spoofing attack against the IP address of the server
- Logs various suspicious packets (spoofed packets, source-routed packets, and
  redirects).

Example of `/etc/sysctl.conf`

```conf
# The following is suitable for dedicated web server, mail, ftp server etc.
# ---------------------------------------
# BOOLEAN Values:
# a) 0 (zero) - disabled / no / false
# b) Non zero - enabled / yes / true
# --------------------------------------
# Controls IP packet forwarding
net.ipv4.ip_forward = 0

# Do not accept source routing
net.ipv4.conf.default.accept_source_route = 0

# Controls the System Request debugging functionality of the kernel
kernel.sysrq = 0

# Controls whether core dumps will append the PID to the core filename
# Useful for debugging multi-threaded applications
kernel.core_uses_pid = 1

# Controls the use of TCP syncookies
# Turn on SYN-flood protections
net.ipv4.tcp_syncookies = 1
net.ipv4.tcp_synack_retries = 5

########## IPv4 networking start ##############
# Send redirects, if router, but this is just server
# So no routing allowed
net.ipv4.conf.all.send_redirects = 0
net.ipv4.conf.default.send_redirects = 0

# Accept packets with SRR option? No
net.ipv4.conf.all.accept_source_route = 0

# Accept Redirects? No, this is not router
net.ipv4.conf.all.accept_redirects = 0
net.ipv4.conf.all.secure_redirects = 0

# Log packets with impossible addresses to kernel log? yes
net.ipv4.conf.all.log_martians = 1
net.ipv4.conf.default.accept_source_route = 0
net.ipv4.conf.default.accept_redirects = 0
net.ipv4.conf.default.secure_redirects = 0

# Ignore all ICMP ECHO and TIMESTAMP requests sent to it via broadcast/multicast
net.ipv4.icmp_echo_ignore_broadcasts = 1

# Prevent against the common 'syn flood attack'
net.ipv4.tcp_syncookies = 1

# Enable source validation by reversed path, as specified in RFC1812
net.ipv4.conf.all.rp_filter = 1

# Controls source route verification
net.ipv4.conf.default.rp_filter = 1

########## IPv6 networking start ##############
# Number of Router Solicitations to send until assuming no routers are present.
# This is host and not router
net.ipv6.conf.default.router_solicitations = 0

# Accept Router Preference in RA?
net.ipv6.conf.default.accept_ra_rtr_pref = 0

# Learn Prefix Information in Router Advertisement
net.ipv6.conf.default.accept_ra_pinfo = 0

# Setting controls whether the system will accept Hop Limit settings from a router advertisement
net.ipv6.conf.default.accept_ra_defrtr = 0

#router advertisements can cause the system to assign a global unicast address to an interface
net.ipv6.conf.default.autoconf = 0

#how many neighbor solicitations to send out per address?
net.ipv6.conf.default.dad_transmits = 0

# How many global unicast IPv6 addresses can be assigned to each interface?
net.ipv6.conf.default.max_addresses = 1

########## IPv6 networking ends ##############

#Enable ExecShield protection
#Set value to 1 or 2 (recommended)
#kernel.exec-shield = 2
#kernel.randomize_va_space=2

# TCP and memory optimization
# increase TCP max buffer size setable using setsockopt()
#net.ipv4.tcp_rmem = 4096 87380 8388608
#net.ipv4.tcp_wmem = 4096 87380 8388608

# increase Linux auto tuning TCP buffer limits
#net.core.rmem_max = 8388608
#net.core.wmem_max = 8388608
#net.core.netdev_max_backlog = 5000
#net.ipv4.tcp_window_scaling = 1

# increase system file descriptor limit
fs.file-max = 65535

#Allow for more PIDs
kernel.pid_max = 65536

#Increase system IP port limits
net.ipv4.ip_local_port_range = 2000 65000

# RFC 1337 fix
net.ipv4.tcp_rfc1337=1
```
