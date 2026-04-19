# ELK Stack on Raspberry Pi 5 - Debian Trixie

This guide explains how to run the Elastic stack (ELK) on a Raspberry Pi 5 running Debian Trixie (bookworm) using Docker.

## Requirements

### Hardware
- Raspberry Pi 5 (4GB or 8GB RAM recommended)
- SD card or SSD (SSD highly recommended for performance)
- Power supply

### Software
- Debian Trixie (or Raspberry Pi OS 64-bit) installed
- Docker Engine
- Docker Compose

## Quick Start

### 1. Install Docker on Raspberry Pi 5

```bash
# Update system
sudo apt update && sudo apt upgrade -y

# Install dependencies
sudo apt install -y ca-certificates curl gnupg lsb-release

# Add Docker GPG key
sudo install -m 0755 -d /etc/apt/keyrings
curl -fsSL https://download.docker.com/linux/debian/gpg | sudo gpg --dearmor -o /etc/apt/keyrings/docker.gpg
sudo chmod a+r /etc/apt/keyrings/docker.gpg

# Add Docker repository
echo "deb [arch=$(dpkg --print-architecture) signed-by=/etc/apt/keyrings/docker.gpg] https://download.docker.com/linux/debian $(lsb_release -cs) stable" | sudo tee /etc/apt/sources.list.d/docker.list > /dev/null

# Install Docker
sudo apt update
sudo apt install -y docker-ce docker-ce-cli containerd.io docker-buildx-plugin docker-compose-plugin

# Add user to docker group
sudo usermod -aG docker $USER
newgrp docker
```

### 2. Verify ARM64 Architecture

```bash
uname -m
# Should output: aarch64
```

### 3. Clone and Configure

```bash
# Clone repository
git clone https://github.com/deviantony/docker-elk.git
cd docker-elk

# For Raspberry Pi 5, update .env to use ARM64-compatible version
# Elastic 8.x+ supports ARM64 natively
```

### 4. Optimize for Raspberry Pi 5

Edit `docker-compose.yml` and reduce memory settings:

```yaml
elasticsearch:
  environment:
    ES_JAVA_OPTS: -Xms256m -Xmx512m  # Reduced for Pi 5

logstash:
  environment:
    LS_JAVA_OPTS: -Xmx128m -Xms128m  # Reduced for Pi 5
```

### 5. Start the Stack

```bash
# Initialize users and passwords
docker compose up setup

# Start the stack
docker compose up -d
```

### 6. Access Kibana

Wait ~2-3 minutes for initialization, then access:
- URL: http://<raspberry-pi-ip>:5601
- Username: elastic
- Password: changeme

## Important Notes

### Performance Considerations
- Use SSD instead of SD card for better I/O performance
- Elasticsearch runs slower on ARM64 - expect slower indexing
- Monitor CPU temperature to prevent throttling
- Consider using 8GB Pi 5 model for better performance

### Recommended Optimizations

For better performance on Pi 5, add to `elasticsearch/config/elasticsearch.yml`:

```yaml
thread_pool.search.queue_size: 100
thread_pool.write.queue_size: 200
indices.memory.index_buffer_size: 10%
```

### Memory Limits

Adjust based on your Pi 5 model:
- 4GB Pi 5: ES=256-512MB, LS=128MB
- 8GB Pi 5: ES=512-1024MB, LS=256MB

### Disable Paid Features (Optional)

To avoid trial license:

```bash
# Before initial setup, edit elasticsearch/config/elasticsearch.yml
xpack.license.self_generated.type: basic
```

## Troubleshooting

### Elasticsearch Won't Start
- Check logs: `docker compose logs elasticsearch`
- Ensure sufficient memory
- Try disabling memory lock: `bootstrap.memory_lock: false`

### Slow Performance
- Use SSD instead of SD card
- Reduce JVM heap size
- Check CPU throttling with `vcgencmd measure_temp`

### Port Conflicts
If ports are already in use, modify ports in `docker-compose.yml`:
- Elasticsearch: 9200, 9300
- Kibana: 5601
- Logstash: 5044, 50000, 9600

## Useful Commands

```bash
# View logs
docker compose logs -f elasticsearch
docker compose logs -f kibana

# Stop stack
docker compose down

# Remove data volumes
docker compose down -v

# Restart specific service
docker compose restart elasticsearch
```

## Tested Versions

- Elasticsearch: 8.x, 9.x (ARM64 supported)
- Kibana: 8.x, 9.x (ARM64 supported)
- Logstash: 8.x, 9.x (ARM64 supported)

For latest ARM64 images, see: https://www.docker.elastic.co