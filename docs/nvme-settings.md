# NVMe Device Support

[Back to README.md](../README.md)

Please see details in the [Custom Message of the Day with ZFS Support](https://github.com/reefland/motd) Repository for details on NVMe devices.

The sections below highlight support for NVMe devices in `36-diskstatus` where it will include any device that starts with `nvme-`:

```bash
#---[ You can updates these ]--------------------------------------------------
# Types of disks to look for. Used by awk to define disk by-id types to include
# IDE/SATA - you might add another for usb "usb-".
findthese="ata-|scsi-SATA_|nvme-"
```

Then any devices matching `nvme-eui` or `nvme-nvme` are removed from the list.  This should leave just devices with the manufacture name such as Samsung, HP, Sabrent, WD, etc. If you have some odd name detected then you probably need to add another filter here.

```yaml
# This is used by awk to remove unwanted disk devices which matched above.
ignorethese="nvme-(eui|nvme).*"
```

Lastly there is a filter which attempts to pretty up the device names for display. For NVMe devices it removes the `nvme-` prefix with the intent that the device name will now start with your manufacture name.

```yaml
# This is used by sed to remove text from disk device names. This does not alter
# device selection like ones above.  This just helps to make disk device names
# nicer (smaller) or easier to read.  This can be used to rename how ugly device
# names are shown.
sed_filter="s/^scsi-SATA_//; s/^ata-//; s/Series_//; s/^nvme-//;"
```

---

When `36-diskstatus` detects an NVMe device, it will try to get the temperature from `smartctl` by looking for `Temperature:` and parsing the value such as `37 Celsius`.

NVMe Wear Level (Life Expectancy) indicator will also be shown.  Expressed as a percentage starting at 100% and lowers over time towards 0%. Ideally you would replace the device before reaching 0%, however a device may still operate without errors after reaching 0%.

[Back to README.md](../README.md)
