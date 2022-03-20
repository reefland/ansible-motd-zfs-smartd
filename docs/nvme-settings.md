# Limited NVMe Device Support

[Back to README.md](../README.md)

Please see details in the [Custom Message of the Day with ZFS Support](https://github.com/reefland/motd) Repository for details on NVMe devices.

Essentially the 3 sections below shows how support for NVMe devices is provided in `36-diskstatus` where it will include any device that starts with `nvme-`:

```bash
#---[ You can updates these ]--------------------------------------------------
# Types of disks to look for. Used by awk to define disk by-id types to include
# IDE/SATA - you might add another for usb "usb-".
findthese="ata-|scsi-SATA_|nvme-"
```

Then any devices matching `nvme-eui` or `nvme-nvme` are removed from the list.  This should leave just devices with the manufacture name such as Samsung, HP, Sabrent, WD, etc. If you have some odd name detected then you probably need to add another filter here.

```yaml
# This is used by awk to remove unwanted disk devices which matched above.
ignorethese="ata-Samsung|nvme-(eui|nvme).*"
```

Lastly there is a filter which attempts to pretty up the device names for display. For NVMe devices it removes the `nvme-` prefix with the intent that the device name will now start with your manufacture name.

```yaml
# This is used by sed to remove text from disk device names. This does not alter
# device selection like ones above.  This just helps to make disk device names
# nicer (smaller).
sed_filter="s/^scsi-SATA_//; s/^ata-//; s/Series_//; s/^nvme-//;"
```

---

The HDD Temp utility does not support NVMe devices.  If `36-diskstatus` script detects a NVMe device, it will try to get the temperature from `smartctl` by looking for `Temperature:` and parsing the value such as `37 Celsius`.

If your device reports temperature differently, then open an issue in the [Custom Message of the Day with ZFS Support](https://github.com/reefland/motd) Repository.

[Back to README.md](../README)
