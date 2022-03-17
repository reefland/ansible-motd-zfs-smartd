
# HDD Temp Settings

[Back to README.md](../README.md)

## Report Temperatures in C or F

The temperature readings can be displayed in Celsius or Fahrenheit by adjusting "C" or "F" in the block below:

```yaml
# have hddtemp report units in C or F
hddtemp_temp_unit: "F"
```

---

## Adding Custom Temperature Sensors to HDDTemp Database

The `hddtemp` utility may have trouble finding temperature sensors on more modern SSDs or NVMe disk devices.  This is because the manufactures have changes the S.M.A.R.T. attribute used to hold the information.  The steps below show that `hddtemp` was unable to locate a temperature sensor for a Samsung SSD 840 Series device.

```shell
 $ sudo hddtemp /dev/sda

 If it shows: WARNING: Drive /dev/sda doesn't seem to have a temperature sensor.
               /dev/sda: Samsung SSD 840 Series:  no sensor
```

The following can help discover the device attribute number for temperature:

```shell
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

By default this is applied to `[motd_group]` defined in the inventory file.  Unsupported platforms are skipped.  The customized `hddtemp` database entries will be added to all hosts defined in this block.

```text
# To limit to a specific host:
$ ansible-playbook -i inventory motd-zfs-smartd.yml -l <remote_host_name>
```

Once completed, `hddtemp` should now be able to report the temperature of the device:

```shell
$ sudo hddtemp /dev/sda
/dev/sda: Samsung SSD 840 Series: 23 C
```

Now the customized Message of the Day scripts will be able to parse that output and report drive temperature.

[Back to README.md](../README.md)
