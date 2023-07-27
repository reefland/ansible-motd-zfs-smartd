# SMARTmon Node Exporter Script

[Back to README.md](../README.md)

Within `defaults/main.yml` if the variable `smartd_node_export_path` is not commented out, then the Smartmon Textfile collector script from [SMARTmon Exporter with Dashboard](https://github.com/reefland/smartmon_nvme) repository will be installed.

A Textfile collector script simply generates a text file of metrics in a format which Prometheus Node Exporter can consume.  

* Node Exporter installation is outside the scope of this process
  * The Node Exporter needs to have the Textfile collector enabled via a command line parameter:

    ```text
    --collector.textfile.directory=/var/lib/node_exporter/textfile_collector
    ```

* The SMARTMon dashboard installation to Grafana is outside the scope of this process

---

Review `defaults/main.yml` file:

* `smartd_node_export_path:` defines the location where Node Exporter Textfile collector is expecting `*.prom` files to be written
  * The value defined needs to match the `--collector.textfile.directory=` setting in Node Exporter Textfile collector

  ```yaml
  # This should match the value defined in node exporter flag "--collector.textfile.directory"
  # If this value does not exist you will need to add node-exporter configuration it allow the
  # Smartmon node exporter scripts output to be consumed.  The path below will be created if it
  # does not already exist.
  #
  smartd_node_export_path: "/var/lib/node_exporter/textfile_collector"
  ```

A Cron job under the `root` account will be created to run the SMARTMon Exporter Script. The `root` account is required as `root` access is needed to interact with `smartctl` utility.  For each script execution, it will create a file named `smartmon.prom` in the directory defined by variable `smartd_node_export_path`.

* `smartd_nvme_script_cron_hour:` specifies which hour the SMARTMon script should execute, the default is all hours

  ```yaml
  # Frequency of running Smartmon Exporter script via Cron
  smartd_nvme_script_cron_hour: "*"  # Run all hours
  ```

* `smartd_nvme_script_cron_minute:` specifies which minute the SMARTMon script should execute, the default is every 5 minutes

  ```yaml
  smartd_nvme_script_cron_minute: "*/5" # Every 5 minutes
  ```

---

Once deployed, if Node Exporter is configured properly you should see Smartmon script metrics being collected within Prometheus:

![Smartmon Node Exporter Metrics in Prometheus](https://github.com/reefland/smartmon_nvme/raw/main/images/prometheus_metrics.png)

See <https://github.com/reefland/smartmon_nvme> for a Dashboard you can import to Grafana:

![Smartmon Grafana Dashboard](https://github.com/reefland/smartmon_nvme/raw/main/images/grafana_dashboard.png)

[Back to README.md](../README.md)
