# Meraki API Exporter for Prometheus
Prometheus exporter to collect some data from Meraki dashboard via API

### Exported metrics
Not all devices exports all metrics.

| metric | unit | description |
| --- | --- | --- |
| meraki_device_latency | seconds |
| meraki_device_loss_percent | % | 0 - 100%
| meraki_device_status | int | 0 - Offline <br> 1 - Online
|meraki_device_using_cellular_failover| int | 1 - using cellular <br> 0 - using main Uplink
| meraki_device_uplink_status | int | 'active': 0 <br> 'ready': 1 <br> 'connecting': 2 <br> 'not connected': 3 <br> 'failed': 4   
| request_processing_seconds | sec | tottal processing time for all hosts, exported once |

### Labels
All metrics but __request_processing_seconds__ has following Labels
| . | . | . |
| --- | --- | --- |
| serial | string | Serial Number
| name | string | Device name or MAC address if name is not defined
| networkId | string | Network ID to which device belongs too  
| orgName  | string | Organisation Name
| orgId | integer | Organisation Id

**meraki_device_uplink_status** also carry "uplink" label containing uplink name.

### How to Use
```
pip install -r requirements.txt
```
You need to provide API Key from meraki portal as argument when starting exporter.<br>
**DO NOT USE KEYS WITH FULL ADMIN PRIVILEGES**<br>
Exporter is listening on port 9822 on all interfaces by default

```
  -h, --help     show this help message and exit
  -k API_KEY     API Key (Required, can also be specified using `MERAKI_API_KEY` environment variable)
  -p http_port   HTTP port to listen for Prometheus scrapper, default 9822
  -i bind_to_ip  IP address where HTTP server will listen, default all interfaces
```
GET request for **/?target=\<Organization Id\>** returns data expected for Prometheus Exporter

GET request for **/organizations** returns YAML formatted list of Organisation Id API key has access to. You can use to automatically populate list of targets in Prometheus configuration.  
```
/usr/bin/curl --silent --output /etc/prometheus/meraki-targets.yml http:/127.0.0.1:9822/organizations
```

**prometheus.yml**
```
scrape_configs:
  - job_name: 'Meraki'
    scrape_interval: 120s
    scrape_timeout: 40s
    metrics_path: /
    file_sd_configs:
      - files:
        - /etc/prometheus/meraki-targets.yml
```
Please check **/systemd** folder for systemd services and timers configuration files, if your system uses it.

### Docker
You can run like this
docker build -t meraki-dashboard-prometheus-exporter:latest
`docker run -p 9822:9822 -e MERAKI_API_KEY=<api key> meraki-dashboard-prometheus-exporter:latest`

Updated:
Update dockerfile for running on amd64 CPU devices
resolving prometheus pass "/?target" as "/%3Ftarget"
