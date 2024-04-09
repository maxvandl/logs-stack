# Install stack

## Install LOKI+GRAFANA+GRAFANA-AGENT

```bash
cd compose && docker-compose up -d
```
## Setup Rsyslog “remote” ruleset on the same VM with Loki and Grafana
On Ubuntu, Rsyslog is installed by default. Any .conf file added in /etc/rsyslog.d will be read and included in the configuration.

The config file below creates a “remote” ruleset which does not interfere with the default local logging. It relays whatever comes in via TCP and UDP on port 514 to Promtail listening on TCP port 1514.

/etc/rsyslog.d/00-promtail-relay.conf
```bash
# https://www.rsyslog.com/doc/v8-stable/concepts/multi_ruleset.html#split-local-and-remote-logging
ruleset(name="remote"){
  # https://www.rsyslog.com/doc/v8-stable/configuration/modules/omfwd.html
  # https://grafana.com/docs/loki/latest/clients/promtail/scraping/#rsyslog-output-configuration
  action(type="omfwd" Target="localhost" Port="1514" Protocol="tcp" Template="RSYSLOG_SyslogProtocol23Format" TCP_Framing="octet-counted")
}


# https://www.rsyslog.com/doc/v8-stable/configuration/modules/imudp.html
module(load="imudp")
input(type="imudp" port="514" ruleset="remote")

# https://www.rsyslog.com/doc/v8-stable/configuration/modules/imtcp.html
module(load="imtcp")
input(type="imtcp" port="514" ruleset="remote")
```

onfigure Remote Syslog Server: Ensure that your remote syslog server is properly configured to receive logs from Docker containers. Depending on the syslog server you're using (e.g., rsyslog, syslog-ng), you may need to adjust its configuration to accept logs from remote hosts.
For example, in rsyslog, you might need to modify the configuration file (/etc/rsyslog.conf) to listen for incoming syslog messages on the specified port:
bash
Copy code
# Provides UDP syslog reception
$ModLoad imudp
$UDPServerRun <port>
Restart Syslog Service: After making changes to the syslog server configuration, restart the syslog service to apply the changes.
Verify: Start your Docker containers and verify that logs are being sent to the remote syslog server. You can check the syslog server logs or use tools like tcpdump to inspect incoming syslog messages.
By following these steps, you should be able to push logs from your Docker containers to a remote syslog server. Make sure to adjust the configurations according to your specific environment and requirements.



# Install Grafana Agent in static mode on Linux VM

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
sudo usermod -a -G adm grafana-agent
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

# Install Grafana Agent in static mode on Windows VM

FOLLOW url
https://grafana.com/docs/agent/latest/static/set-up/install/install-agent-on-windows/#standard-install
