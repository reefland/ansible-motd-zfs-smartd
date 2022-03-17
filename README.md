# Custom Message of the Day (MOTD), Smartmon Tools, HDDTemp with ZFS Support

Smartmon Tools are used to test and monitor disk devices. HDDTemp is used to monitor temperature of disk devices (SATA, SSD, NVMe all typically have sensors).  This information is gathered in a customized Message of the Day which is presented upon a SSH login along with status of your ZFS Pools and available Patches.

![Sample Login Screen](images/colors_from_dropdown.png)

---

## TL;DR

* Standard Message of the Day (MOTD) output can be disabled or replaced with enhanced versions.
* HDD/SSD Testing will be provided by Smartmon Tools.
* HDD/SSD Testing Schedulers can be specified.
* Each system login will show you the status of the last device test as well as its current temperature

---

## Requirements

* SSH Server configured to display message of the day (enabled by default on Ubuntu)
* Ubuntu 18.04 or 20.04 releases

---

## Packages Installed

The following packages will be installed:

* [smartmontools](https://en.wikipedia.org/wiki/Smartmontools) retrieves the S.M.A.R.T. (Selt-Monitoring, Analysis and Reporting Technology) attributes from the disk devices including temperature and performs periodic testing of the disk devices
* [hddtemp](https://wiki.archlinux.org/index.php/Hddtemp) reads temperature S.M.A.R.T.  attribute from disk devices. This will be run as a daemon
* [git](https://en.wikipedia.org/wiki/Git) needed to retrieve files from [Custom Message of the Day with ZFS Support](https://gitea.rich-durso.us/reefland/motd) Repository
* [update-motd](http://manpages.ubuntu.com/manpages/focal/man5/update-motd.5.html) is a framework by which `motd` is dynamically assembled from a collection of scripts at login
* [figlet](https://en.wikipedia.org/wiki/FIGlet) to generate the hostname as a larger banner font
* [lolcat](http://manpages.ubuntu.com/manpages/focal/man6/lolcat.6.html) provides a colorful rendering of the `figlet` hostname banner
* [bc](https://en.wikipedia.org/wiki/Bc_(programming_language)) a basic calculator used for working with temperature values
* [update-notifier-common](https://packages.ubuntu.com/focal/update-notifier-common) provides some of the MOTD scripts required including notification if a reboot is required
* [bsdmainutils](https://launchpad.net/ubuntu/focal/+package/bsdmainutils) provides text parsing utilities ported over from BSD. Utilities such as `column` are used to present disk information.

---

## How Do I Set It Up

### Edit your inventory document

Add something like the following:

```shell
[motd_group:vars]
enable_custom_motd_entries='["10-hostname-color", "20-sysinfo", "30-zpool-bar", "40-services"]'

[motd_group]
testlinux.example.com more_motd_entries='["60-docker"]'
```

* The `[motd_group:vars]` block defines variables that will be applied to all systems defined in the group and can override variables defined in `defaults/main.yml`.
  * The variable `enable_custom_motd_entries=` is optional and specifies which MOTD messages are to be applied to all systems defined in this group.  If not defined here the value in `defaults/main.yml` will be used.
* The `[motd_group]` block lists the hostname(s) that you intent to apply this script to.
  * The variable `more_motd_entries=` is optional and specifies which MOTD messages are unique to that host and not installed on every host.  If not defined here then nothing else will be added to `enable_custom_motd_entries` list.

---

### Review `defaults/main.yml` to define the defaults

The `defaults/main.yml` can be used to configure which default messages (installed by OS) are disabled and which new ones are to be added and enabled.

#### Default Message Files to Disabled

This block defines existing messages which will be disabled by simply removing the execute bit from them:

```yaml
motd_entries:
  # Define Message of the Day items to disable by removing -x from file
  to_disable:
    - "10-help-text"
    - "80-esm"
    - "80-livepatch"
```

Some older version of Ubuntu used symlinks, this block can unlink them:

```yaml
motd_entries:
  # Define Message of the Day items which need to be unlinked (remove -x does not work)
  to_unlink:
    - "50-landscape-sysinfo"
```

The Ubuntu NEWS is pretty much SPAM:

```yaml
# Display the Ubuntu News Message - 0 = disable, 1 = enable
# value set within: /etc/default/motd-news
show_ubuntu_news_message: '0'
```

### Enhanced Message Files to Enable

The following defines the base set of new message files from the customized MOTD repo which will be enabled.  The selected message files are applied to all systems.  Message files to be enabled on specific systems are defined below.

```yaml
motd_entries:
  # Define Custom Message of the Day files to ENABLE from the GIT Repo to all hosts
  enable_these_for_all_hosts:
  #  - 10-hostname                 # Blah no color hostname
    - 10-hostname-color
    - 20-sysinfo
  #  - 20-uptime                   # Standard uptime message  
    - 30-zpool-bar                # ZFS Usage Graphs
    - 36-diskstatus               # Disks, Temps, Test Results
    - 40-services                 # Status of Services
  #  - 50-fail2ban                 # Fail2ban Summary (standard)
  #  - 50-fail2ban-status          # Fail2ban status per jail
  #  - 60-lxd                      # Status of lxd containers
  #  - 60-docker                   # Status of Deployed Docker Containers
```

### System Specific Message Files to Enable

Some message files don't apply to all servers such as LXD or Docker.  To enable these on specific systems you can define it within the inventory file per host via `more_motd_entries` variable:

#### Specify via Inventory File

```shell
[motd_group:vars]
enable_these_for_all_hosts='["10-hostname-color", "20-sysinfo", "30-zpool-bar", "40-services"]'

[motd_group]
testlinux.example.com more_motd_entries='["60-docker"]'
```

### Report Temperatures in C or F

The temperature readings can be displayed in Celsius or Fahrenheit by adjusting "C" or "F" in the block below:

```yaml
# have hddtemp report units in C or F
hddtemp_temp_unit: "F"
```

---

### Running the Custom Message of the Day (MOTD) with ZFS Support Playbook

This is an example playbook named `motd-zfs-smartd.yml` that can be used to define hosts, the custom MOTD messages to deploy and which MOTD messages are host specific:

```yml
[motd_group:vars]
ansible_user=ansible
ansible_ssh_private_key_file=/home/rich/.ssh/ansible
ansible_python_interpreter=/usr/bin/python3
enable_custom_motd_entries='["10-hostname-color", "20-sysinfo", "30-zpool-bar", "40-services"]'

[motd_group]
k3s01.example.com
testlinux.example.com more_motd_entries='["60-docker"]'
```

```bash
ansible-playbook -i inventory motd-zfs-smartd.yml

# Use Ansible's limit parameter to specify individual hostname to run on:
ansible-playbook -i inventory motd-zfs-smartd.yml -l testlinux.example.com
```

_NOTE: You would want to run this again anytime a disk device is replaced to make sure `smartmontools` has added the device to its testing schedule. Or you can manually update the `/etc/smartd.conf` file._

## Scheduled Drive Testing

When the Customized Message of the Day `disk status:` states a drive is "without error" it is parsing that information from the device's last test result:

```text
disk status:
  Samsung_SSD_840_928H (sda):  73F | without error
```

Various short and long tests are scheduled for each device that `smartmontools` is monitoring.  This schedule and type of test scheduled can be viewed at anytime using:

```text
$ sudo smartd -q showtests

Next scheduled self tests (at most 5 of each type per device):
Device: /dev/disk/by-id/ata-Samsung_SSD_840_Series_S14GNEACB04928H [SAT], will do test 1 of type S at Mon Aug 17 02:10:20 2020 EDT
Device: /dev/disk/by-id/ata-Samsung_SSD_840_Series_S14GNEACB04928H [SAT], will do test 2 of type S at Tue Aug 18 02:10:20 2020 EDT
Device: /dev/disk/by-id/ata-Samsung_SSD_840_Series_S14GNEACB04928H [SAT], will do test 3 of type S at Wed Aug 19 02:10:20 2020 EDT
Device: /dev/disk/by-id/ata-Samsung_SSD_840_Series_S14GNEACB04928H [SAT], will do test 4 of type S at Thu Aug 20 02:10:20 2020 EDT
Device: /dev/disk/by-id/ata-Samsung_SSD_840_Series_S14GNEACB04928H [SAT], will do test 5 of type S at Fri Aug 21 02:10:20 2020 EDT

Device: /dev/disk/by-id/ata-Samsung_SSD_840_Series_S14GNEACB04928H [SAT], will do test 1 of type L at Sat Aug 22 03:10:20 2020 EDT
Device: /dev/disk/by-id/ata-Samsung_SSD_840_Series_S14GNEACB04928H [SAT], will do test 1 of type L at Sat Aug 22 03:10:20 2020 EDT
Device: /dev/disk/by-id/ata-Samsung_SSD_840_Series_S14GNEACB04928H [SAT], will do test 3 of type L at Sat Sep  5 03:10:20 2020 EDT
Device: /dev/disk/by-id/ata-Samsung_SSD_840_Series_S14GNEACB04928H [SAT], will do test 4 of type L at Sat Sep 12 03:10:20 2020 EDT
Device: /dev/disk/by-id/ata-Samsung_SSD_840_Series_S14GNEACB04928H [SAT], will do test 5 of type L at Sat Sep 19 03:10:20 2020 EDT

Totals [Sun Aug 16 11:40:20 2020 EDT - Sat Nov 14 10:40:20 2020 EST]:
Device: /dev/disk/by-id/ata-Samsung_SSD_840_Series_S14GNEACB04928H [SAT], will do  13 tests of type L
Device: /dev/disk/by-id/ata-Samsung_SSD_840_Series_S14GNEACB04928H [SAT], will do  90 tests of type S
Device: /dev/disk/by-id/ata-Samsung_SSD_840_Series_S14GNEACB04928H [SAT], will do   0 tests of type C
Device: /dev/disk/by-id/ata-Samsung_SSD_840_Series_S14GNEACB04928H [SAT], will do   0 tests of type O
```

### Altering the Testing Schedule

The default time configured for short test is roughly 2:00 in the morning and long test roughly at 03:00 in the morning. This can be changed by editing `defaults/main.yml` block:

```yaml
  # Define when smartctl shot and long test will take place. 
  # Format is: MM/DD/d/HH, where:

  # MM = Month of the year, expressed with two decimal digits. The range is from 01 (January)
  #      to 12 (December) inclusive. Do not use a single decimal digit or the match will always fail!
  # DD = Day of the month, expressed with two decimal digits. The range is from 01 to 31 inclusive. 
  #      Do not use a single decimal digit or the match will always fail!
  # d  = Day of the week, expressed with one decimal digit. The range is from 1 (Monday) to 7 (Sunday)
  #      inclusive.
  # HH = Hour of the day, written with two decimal digits, and given in hours after midnight. The 
  #      range is 00 (midnight to just before 1am) to 23 (11pm to just before midnight) inclusive.
  #      Do not use a single decimal digit or the match will always fail!
  
  # Use a ".." as double digit and "." as any single digit.

  smartd_short_test: '../.././02'   # 2AM every day
  smartd_long_test:  '../15/./03'   # 3AM on the 15th of every month
```

Alternatively the scheduled times can be updated directly within `/etc/smartd.conf` for each device.

## Adding Custom Temperature Sensors to HDDTemp Database

The `hddtemp` utility may have trouble finding temperature sensors on more modern SSDs or NVMe disk devices.  This is because the manufactures have changes the S.M.A.R.T. attribute used to hold the information.  The steps below show that `hddtemp` was unable to locate a temperature sensor for a Samsung SSD 840 Series device.

```text
 $ sudo hddtemp /dev/sda

 If it shows: WARNING: Drive /dev/sda doesn't seem to have a temperature sensor.
               /dev/sda: Samsung SSD 840 Series:  no sensor
```

The following can help discover the device attribute number for temperature:

```text
$ sudo smartctl -a /dev/sda | grep -i temp

Finds: 190 Airflow_Temperature_Cel 0x0032   077   060   000    Old_age   Always       -       23

```

Based on the output of above, we know attribute `190` is used for temperature and it will be reported in `Celsius`.  We can use this information to create a custom rule for Ansible to apply to the `hddtemp` database.

The following block in `defaults/main.yml` can be used to define custom temperature sensors:

```yaml
manual_entries_to_hddtemp:
  - '"Samsung SSD (840|860)" 190 C "Temp for Samsung SSDs"'
```

Where:

* Field 1: Use a string or regex matching the drive's display name (as reported by hddtemp output). In this example the regex is `"Samsung SSD (840|860)"` which will match the 840 or 860 Samsung SSD series.
* Field 2: S.M.A.R.T. data attribute number to use. Based on the `grep` search above it found attribute `190` for Samsung SSD.
* Field 3: Specifies if the attribute above temperature unit (C or F)
* Field 4: A simple Label string or comment you define for the entry. In this example the comment is `"Temp for Samsung SSDs"`

_NOTE: Be mindful of all the single and double quotes when adding an entry these are needed for proper yaml parsing._

### Re-run the Smartmon Tools with ZFS Support Playbook

By default this is applied to `hosts: all` defined in the inventory file.  Unsupported platforms are skipped.  The customized `hddtemp` database entries will be added to all hosts.

```text
$ ansible-playbook -i inventory motd-zfs-smartd.yml -l <remote_host_name>
```

Once completed, `hddtemp` should now be able to report the temperature of the device:

```text
$ sudo hddtemp /dev/sda
/dev/sda: Samsung SSD 840 Series: 23 C
```

Now the customized Message of the Day scripts will be able to parse that output and report drive temperature.
