# Syslog Forwarder

A lightweight syslog collector and forwarder using rsyslog that collects logs from network devices and forwards them to the SIEM (Logstash).

## Quick Start

### 1. Start the ELK Stack

First, ensure the ELK stack is running:

```bash
cd ..
docker compose up -d
```

### 2. Start Syslog Forwarder

```bash
docker compose up -d
```

## Configuration

The forwarder listens on:
- **TCP** port 514
- **UDP** port 514

It forwards all received logs to Logstash on port 514.

## Configure Devices to Send Syslog

### Network Devices (Router, Switch, Firewall)

```bash
# Cisco IOS
logging host <PI-IP>
logging trap informational
logging source interface <interface>

# Ubiquiti Edgerouter
set system syslog host <PI-IP>
set system syslog facility all

# pfSense
# Go to Status > System Logs > Settings
# Enable Remote Logging -> enter Pi IP
```

### Linux Systems

```bash
# Add to /etc/rsyslog.conf
*.* @@<PI-IP>:514

# Or use journalctl
journalctl -t <service> | nc -u <PI-IP> 514
```

### Windows Systems

Use NXLog, WinSyslog, or the Windows Event Forwarding feature.

### Test Sending Logs

```bash
# From any Linux system
logger -n <PI-IP> -P 514 "Test message from syslog-forwarder"
```

## Architecture

```
[Network Devices] --syslog--> [syslog-forwarder:514] --> [Logstash:514] --> [Elasticsearch]
```

## Environment Variables

| Variable | Description | Default |
|----------|-------------|---------|
| RSYSLOG_CONFIG_DEBUG | Enable debug mode | 0 |

## Volumes

| Path | Description |
|------|-------------|
| /var/log/remote | Local log storage |
| /etc/rsyslog.conf | Rsyslog configuration |

## Troubleshooting

### Check if syslog forwarder is running

```bash
docker compose ps
docker compose logs syslog-forwarder
```

### Test connectivity

```bash
# From Pi
nc -l 514

# From remote host
logger -n <PI-IP> -P 514 "Test"
```

### Check logs in Kibana

Search for index pattern `logstash-*` and filter by `type:syslog`