# homeLAB monitoring (Telegraf/Loki/Grafana) stack

## Mapped Ports

```
Host  Container     Service
3000  3000          grafana
514   6514          telegraf (syslog / udp / RFC3164)
601   6601          telegraf (syslog / udp,tcp / RFC5424)
8086  8086          influxDB

3100  3100          loki
3003  3003          grafana
```

## Setup

### Use Loki-Driver as default log driver

- Installing: https://grafana.com/docs/loki/latest/clients/docker-driver/#installing

- Normal-way
  
  Edit docker daemon config 
  - Linux: /etc/docker/daemon.json 
  - macOS: ~/.docker/daemon.json)

```json
{
    "debug" : "false",
    "log-driver": "loki",
    "log-opts": {
      "loki-url": "http://localhost:3100/loki/api/v1/push",
      "loki-external-labels": "job=dockerlog",
      "loki-batch-size": "400",
    }
}
```

````
/etc/init.d/container-station.sh restart
````

- QNAP-way

```js
x-logging: &loki-logging
  driver: loki
  options:
    loki-url: http://localhost:3100/loki/api/v1/push
    loki-external-labels: job=dockerlog
    loki-retries: 3
    mode: non-blocking
    max-buffer-size: 4m

services:
  sample:
    # ....
    logging: *loki-logging

```


```
  logging:
    driver: loki
    options:
      loki-url: http://localhost:3100/loki/api/v1/push
      loki-external-labels: job=dockerlog
      loki-retries: 3
      loki-batch-size": 400
      mode: non-blocking
      max-buffer-size: 4m
```


## Usefull Links

- https://towardsdatascience.com/get-system-metrics-for-5-min-with-docker-telegraf-influxdb-and-grafana-97cfd957f0ac
- https://iosonounrouter.wordpress.com/2020/10/29/quickly-collect-syslogs-with-containerized-tig-stack/
- https://blog.lrvt.de/monitoring-dashboard-with-grafana-telegraf-influxdb-and-docker/
- https://www.blackvoid.club/grafana-8-influxdb-2-telegraf-2021-monitoring-stack/

### Logging

#### influxDB

- https://powersj.io/posts/influxdb-docker-dev/
- https://www.laub-home.de/wiki/InfluxDB_2_Docker_Installation

#### telegraf

#### loki

- http://blog.ruanbekker.com/cheatsheets/loki/
- https://ruanbekker.medium.com/logging-with-docker-promtail-and-grafana-loki-d920fd790ca8

### Metrics

- https://blog.lrvt.de/monitoring-dashboard-with-grafana-telegraf-influxdb-and-docker/

## Test

### stack

````
cd /share/Container/stacks/homelab.monitoring-tig.stack
````

### syslog-ng (linuxserver)

Config:
````
cat /config/syslog-ng.conf
````

Sate:
````
syslog-ng -V

syslog-ng-ctl [cmd] -c /config/syslog-ng.ctl

syslog-ng-ctl stats -c /config/syslog-ng.ctl

syslog-ng-ctl trace --set=on -c /config/syslog-ng.ctl

````

Logs:
````
cat /var/log/messages
cat /var/log/messages-kv.log
````

Send:
```
# TCP with octet framing
echo "57 <13>1 2018-10-01T12:00:00.0Z example.org root - - - test" | nc 192.168.1.20 6514
# UDP
echo "<13>1 2018-10-01T12:00:00.0Z example.org root - - - test" | nc -u 192.168.1.20 6514


nc -w0 -u 192.168.1.20 514 <<< "udp testing again from my home machine"
nc -w0 192.168.1.20 601 <<< "tcp testing again from my home machine"
```
