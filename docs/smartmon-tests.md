
# Smartmon Drive Testing

[Back to README.md](../README.md)

When the Customized Message of the Day `disk status:` states a drive is `without error` it is parsing that information from the device's last test result stored in the system logs.

```text
disk status:
  Samsung_SSD_840_928H (sda):  73F  without error
```

For devices able to present a wear level indicator, the `without error` is changed to an all lowercase `passed` to make room to display the indicator:

```text
disk status:
  Samsung_SSD_860_EVO_1TB_249Y (sde):  84F  passed [86%]
```

When test results for the device cannot be obtained from the system logs, it will fallback to checking `smartctl`.  This will be reported as a simple all uppercase `PASSED` or `FAILED`. Seeing all uppercase result should be investigated to determine why test results cannot be found in the system logs.

```text
disk status:
  Crucial_MX500_1TB_SSD_3619 (sda):  104F  PASSED
```

## Drive Testing Schedules

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

### Testing Schedule

The default test schedule is configured for short test is roughly 02:00 each morning and long test roughly at 03:00 in the morning on the 15th of each month. This can be changed by editing `defaults/main.yml` block:

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

Alternatively the scheduled times can be updated directly within `/etc/smartd.conf`.

### Testing Directives

`smartctl` supports many testing and reporting options. The `DEFAULT` is defined which will be applied to all detected devices:

```yaml
# Smartmon DEFAULT testing directives
# -a = Equivalent to turning on all of the following Directives:
#     '-H' to check the SMART health status
#     '-f' to report failures of Usage (rather than Prefail) Attributes
#     '-t' to track changes in both Prefailure and Usage Attributes
#     '-l error' to report increases in the number of ATA errors
#     '-l selftest' to report increases in the number of Self-Test Log errors
#     '-l selfteststs' to report changes of Self-Test execution status
#     '-C 197' to report nonzero values of the current pending sector count
#     '-U 198' to report nonzero values of the offline pending sector count
# -s (S/xxx|Lxxx) = Self-Test schedules
# -m = Send a warning email to one or more email address
# -M exec = run the executable PATH instead of the default mail command, when smartd needs to send email
smartd_conf_values: >
  DEFAULT -a -s (S/{{ smartd_short_test }}|L/{{ smartd_long_test }}) -m {{ smartd_email_notification }} -M exec /usr/share/smartmontools/smartd-runner

```

* Variables `smartd_short_test` and `smartd_long_test` are test schedules defined in previous section.
* Variable `smartd_email_notification` defines where the email notifications will be sent (see section below).

### Device Warning / Failure Email Alerts

`smartctl` can send email notifications if a local SMTP client is available and configured for system use.  By default notification is just sent to the `root` account.  You can specify one or multiple email addresses for alert notification:

```yaml
# If smtp client is available to allow external email, set notification email address. To send email
# to more than one user, use a "comma separated" form for the address (no spaces). Reminder this
# variable can be set uniquely per host or groups of hosts using ansible inventory file or other
# ansible methods:  user1@address1,user2@address2,...,userN@addressN (with no spaces)
smartd_email_notification: "root"
```

* NOTE: like any ansible variable, this can be overridden and defined at host or host group levels using inventory file or other ansible methods.

### Device Detection

Whenever a device is added or replaced, this playbook should be run again to update the `/etc/smartd.conf` file to refresh the list of devices to be monitored.

Be aware that testing with `smartctl` feature `DEVICESCAN` was tried. Several problems were identified with `DEVICESCAN`:

* It uses names like `/dev/sda`, `/dev/sdb`, which are not consistent. These names are assigned in the order the devices responded at boot and these name can point to different devices at each boot.
* These names are also not unique. If a device failed or was replaced, you would still get test results from previous device until next test run as the device name was not unique.
* The script `36-diskstatus` looks for the unique names to determine test status from `journalctl`, it will not find test results created by `DEVICESCAN` using names like `/dev/sda`. It will fall back to simple `PASSED` or `FAILED` indicators based on `smartctl` status.
