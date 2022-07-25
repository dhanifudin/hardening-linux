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
7. Finally eddit the /etc/apt/listchanges.conf and set email  ID:
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
