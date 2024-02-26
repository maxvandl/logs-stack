# Install stack

## Install LOKI+GRAFANA+GRAFANA-AGENT

```bash
cd compose && docker-compose up -d
```

## Install Grafana Agent in static mode on Linux

You can install Grafana Agent in static mode on Linux.

## Install on Debian or Ubuntu

To install Grafana Agent in static mode on Debian or Ubuntu, run the following commands in a terminal window.

Import the GPG key and add the Grafana package repository:

```bash
sudo mkdir -p /etc/apt/keyrings/
wget -q -O - https://apt.grafana.com/gpg.key | gpg --dearmor | sudo tee /etc/apt/keyrings/grafana.gpg > /dev/null
echo "deb [signed-by=/etc/apt/keyrings/grafana.gpg] https://apt.grafana.com stable main" | sudo tee /etc/apt/sources.list.d/grafana.list
```

Update the repositories:

```bash
sudo apt-get update
```

## Install Grafana Agent:

```bash
sudo apt-get install grafana-agent
```

## Configure Grafana Agent:

edit file

```
/etc/grafana-agent.yaml
```

add inside

```yml
---
logs:
  configs:
    - name: default
      clients:
        - url: http://{LOKI_URL}:3100/loki/api/v1/push
      positions:
        filename: /tmp/positions.yaml
      scrape_configs:
        - job_name: system
          static_configs:
            - targets:
                - localhost
              labels:
                job: varlogs
                __path__: /var/log/*log
```

## Uninstall on Debian or Ubuntu

To uninstall Grafana Agent on Debian or Ubuntu, run the following commands in a terminal window.

Stop the systemd service for Grafana Agent:

```bash
sudo systemctl stop grafana-agent
sudo systemctl disable grafana-agent
```

## Uninstall Grafana Agent:

```bash
sudo apt-get remove grafana-agent
```

## Optional: Remove the Grafana repository:

```bash
sudo rm -i /etc/apt/sources.list.d/grafana.list
sudo rm -i /etc/apt/keyrings/grafana.gpg
```

## Install Grafana Agent in static mode on Windows

FOLLOW url
https://grafana.com/docs/agent/latest/static/set-up/install/install-agent-on-windows/#standard-install
