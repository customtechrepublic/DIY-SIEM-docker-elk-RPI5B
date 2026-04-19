# DIY SIEM - ELK Stack on Raspberry Pi 5

A home-made Security Information and Event Management (SIEM) solution using the Elastic stack (Elasticsearch, Logstash, Kibana) running on a Raspberry Pi 5 with Debian Trixie.

## Overview

This project provides a ready-to-use ELK stack configured and optimized for the Raspberry Pi 5 ARM64 architecture. Perfect for home lab security monitoring, log aggregation, and learning SIEM concepts.

## Requirements

### Hardware
- Raspberry Pi 5 (4GB or 8GB RAM)
- SD card or SSD (SSD highly recommended)
- Stable power supply (official Pi 5 power adapter)

### Software
- Debian Trixie / Raspberry Pi OS 64-bit
- Docker Engine 18.06.0+
- Docker Compose 2.0.0+

## Quick Start

### 1. Install Docker on Raspberry Pi

```bash
# Update system
sudo apt update && sudo apt upgrade -y

# Install dependencies
sudo apt install -y ca-certificates curl gnupg lsb-release

# Add Docker GPG key and repository
sudo install -m 0755 -d /etc/apt/keyrings
curl -fsSL https://download.docker.com/linux/debian/gpg | sudo gpg --dearmor -o /etc/apt/keyrings/docker.gpg
sudo chmod a+r /etc/apt/keyrings/docker.gpg
echo "deb [arch=$(dpkg --print-architecture) signed-by=/etc/apt/keyrings/docker.gpg] https://download.docker.com/linux/debian $(lsb_release -cs) stable" | sudo tee /etc/apt/sources.list.d/docker.list > /dev/null

# Install Docker
sudo apt update
sudo apt install -y docker-ce docker-ce-cli containerd.io docker-buildx-plugin docker-compose-plugin

# Add user to docker group
sudo usermod -aG docker $USER
newgrp docker
```

### 2. Clone and Run

```bash
# Clone this repository
git clone https://github.com/yourusername/DIY-SIEM-docker-elk-RPI5B.git
cd DIY-SIEM-docker-elk-RPI5B

# Initialize users and passwords
docker compose up setup

# Start the stack
docker compose up -d
```

### 3. Access Kibana

Wait 2-3 minutes for initialization, then open:
- **URL**: `http://<raspberry-pi-ip>:5601`
- **Username**: `elastic`
- **Password**: `changeme`

## Configuration

### Environment Variables

Edit `.env` file to customize:

```env
ELASTIC_VERSION=9.2.2
ELASTIC_PASSWORD=changeme
LOGSTASH_INTERNAL_PASSWORD=changeme
KIBANA_SYSTEM_PASSWORD=changeme
```

### Memory Tuning

Default settings are optimized for Raspberry Pi 5:

| Service | Default Heap | Notes |
|---------|--------------|-------|
| Elasticsearch | 256-512MB | Adjust based on available RAM |
| Logstash | 128MB | Sufficient for light load |
| Kibana | Auto | Uses ~512MB max |

To modify, edit `docker-compose.yml`:

```yaml
environment:
  ES_JAVA_OPTS: -Xms512m -Xmx1024m
```

### Ports

| Service | Port | Description |
|---------|------|-------------|
| Elasticsearch | 9200 | HTTP API |
| Elasticsearch | 9300 | Transport |
| Kibana | 5601 | Web UI |
| Logstash | 5044 | Beats input |
| Logstash | 50000 | TCP input |
| Logstash | 9600 | Monitoring API |

## Project Structure

```
DIY-SIEM-docker-elk-RPI5B/
├── docker-compose.yml     # Main compose file
├── .env                   # Environment variables
├── elasticsearch/         # Elasticsearch config
│   ├── config/
│   │   └── elasticsearch.yml
│   └── Dockerfile
├── kibana/               # Kibana config
│   ├── config/
│   │   └── kibana.yml
│   └── Dockerfile
├── logstash/             # Logstash config
│   ├── config/
│   │   └── logstash.yml
│   ├── pipeline/
│   │   └── logstash.conf
│   └── Dockerfile
├── setup/                # Initialization scripts
│   ├── entrypoint.sh
│   ├── lib.sh
│   └── roles/
├── extensions/           # Optional extensions
│   ├── filebeat/
│   ├── metricbeat/
│   ├── heartbeat/
│   └── curator/
└── RPI5-SETUP.md         # Detailed setup guide
```

## Usage

### Sending Logs to Logstash

```bash
# Send logs via TCP port 50000
cat /path/to/logfile.log | nc -q0 <pi-ip> 50000

# Or use Beats (Filebeat, etc.)
```

### Managing the Stack

```bash
# View all logs
docker compose logs -f

# View specific service
docker compose logs -f elasticsearch
docker compose logs -f kibana
docker compose logs -f logstash

# Stop the stack
docker compose down

# Remove all data
docker compose down -v

# Restart a service
docker compose restart elasticsearch
```

### Changing Passwords

```bash
# Reset elastic user password
docker compose exec elasticsearch bin/elasticsearch-reset-password --batch --user elastic

# Reset logstash_internal password
docker compose exec elasticsearch bin/elasticsearch-reset-password --batch --user logstash_internal

# Reset kibana_system password
docker compose exec elasticsearch bin/elasticsearch-reset-password --batch --user kibana_system
```

Then update passwords in `.env` and restart:
```bash
docker compose up -d logstash kibana
```

## Extensions

### Enable Filebeat

```bash
cd extensions/filebeat
docker compose up -d
```

### Enable Metricbeat

```bash
cd extensions/metricbeat
docker compose up -d
```

### Enable Heartbeat

```bash
cd extensions/heartbeat
docker compose up -d
```

## Performance Tips

1. **Use SSD** - Dramatically improves Elasticsearch I/O
2. **Monitor Temperature** - Use `vcgencmd measure_temp` to check CPU temp
3. **Reduce Shard Count** - Limit primary shards to 1-2 for small datasets
4. **Disable Trial Features** - Set `xpack.license.self_generated.type: basic` in `elasticsearch.yml`
5. **Adjust JVM Heap** - Match to available RAM (leave 1GB for OS on 4GB Pi)

## Troubleshooting

### Elasticsearch Won't Start
```bash
# Check logs
docker compose logs elasticsearch

# Common fixes:
# - Reduce heap size in docker-compose.yml
# - Ensure bootstrap.memory_lock is disabled
# - Check disk space
```

### Kibana Shows Blank Page
- Wait 2-3 minutes for initialization
- Check logs: `docker compose logs kibana`
- Verify Elasticsearch is running: `curl http://localhost:9200`

### Slow Performance
- Use faster storage (NVMe SSD > SD card)
- Reduce JVM heap if swapping
- Check for CPU throttling

### Port Already in Use
```bash
# Find process using port
sudo lsof -i :5601

# Change port in docker-compose.yml if needed
```

## Security Notes

- **Change default passwords** immediately after first run
- **Enable firewall** on Raspberry Pi
- **Use SSL/TLS** for production (see tls branch)
- **Limit network exposure** - only expose necessary ports

## Useful Links

- [Elasticsearch Docker Docs](https://www.elastic.co/docs/deploy-manage/deploy/self-managed/install-elasticsearch-with-docker)
- [Kibana Docker Docs](https://www.elastic.co/docs/deploy-manage/deploy/self-managed/install-kibana-with-docker)
- [Logstash Docker Docs](https://www.elastic.co/docs/reference/logstash/docker-config)
- [Elastic Docker Registry](https://www.docker.elastic.co)

## Credits

Based on [docker-elk](https://github.com/deviantony/docker-elk) by Tony.

## License

MIT License - See LICENSE file for details.