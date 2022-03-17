
# Smartmon Drive Testing

[Back to README.md](../README.md)

When the Customized Message of the Day `disk status:` states a drive is `without error` it is parsing that information from the device's last test result:

```text
disk status:
  Samsung_SSD_840_928H (sda):  73F | without error
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
