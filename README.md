# Bluetooth Speaker Reconnector

Automatic Bluetooth speaker connection monitor and Snapcast client for Raspberry Pi.

## Features

- **Bluetooth Reconnector**: Monitors and maintains connection to Bluetooth audio devices
- **Snapcast Client**: Streams audio from Snapcast server through connected Bluetooth speaker
- Fully containerized with Docker
- Automatic reconnection on connection loss
- Audio device filtering (only shows/connects to audio devices)

## Prerequisites

### Network Configuration (Static IP)

Configure a static IP address for reliable network access:

```bash
# List available connections
nmcli connection show

# Configure static IP with router-provided DNS (replace values with your network settings)
sudo nmcli connection modify "netplan-wlan0-francescos-house-5g" \
  ipv4.addresses 192.168.1.105/24 \
  ipv4.gateway 192.168.1.254 \
  ipv4.dns "192.168.1.100 1.1.1.1" \
  ipv4.method manual

sudo nmcli connection up "netplan-wlan0-francescos-house-5g"
```

### PipeWire (Audio System)

Raspberry Pi OS (Bookworm or newer) comes with **PipeWire** pre-installed with Bluetooth support. No additional installation needed.

You can verify PipeWire is running:
```bash
systemctl --user status pipewire pipewire-pulse
```

### Docker

Install Docker on Raspberry Pi:
```bash
# Install Docker
curl -fsSL https://get.docker.com -o get-docker.sh
sudo sh get-docker.sh

# Add your user to the docker group
sudo usermod -aG docker $USER

# Refresh group membership without logging out
newgrp docker

# Verify
docker --version

# Start Portainer Agent (optional, for remote management)
docker run -d \
  -p 9001:9001 \
  --name portainer_agent \
  --restart=always \
  -v /var/run/docker.sock:/var/run/docker.sock \
  -v /var/lib/docker/volumes:/var/lib/docker/volumes \
  portainer/agent:latest
```

## Usage

### Bluetooth Reconnector

The reconnector container monitors and maintains connection to a Bluetooth speaker.

```bash
# List available Bluetooth audio devices
cd bluetooth-reconnector
docker-compose run --rm bluetooth-reconnector python3 /app/bluetooth-speaker-reconnector.py --list --timeout 15

# Run with docker-compose (monitoring mode)
docker-compose up -d
```

Configure the MAC address in `bluetooth-reconnector/docker-compose.yml`:
```yaml
environment:
  - BT_MAC=60:AB:D2:08:0C:32  # Replace with your speaker's MAC
```

### Snapcast Client

The snapclient container plays audio from your Snapcast server through the connected Bluetooth speaker.

```bash
cd snapclient
docker-compose up --build
```

Configure the Snapcast server in `snapclient/docker-compose.yml`:
```yaml
command: ["snapclient", "-h", "snapcast.francescomeli.com", "-s", "pulse", "--hostID", "hallway-main"]
```

## Architecture

1. **Bluetooth Reconnector** pairs and connects to the Bluetooth speaker using `bluetoothctl`
2. **PipeWire** (on host) detects the paired device and creates an audio sink
3. **Snapcast Client** plays audio through PipeWire → Bluetooth speaker

Both containers use `network_mode: host` to access the host's PipeWire/Bluetooth stack.

## Environment Variables

### Bluetooth Reconnector

- `BT_MAC`: MAC address of the Bluetooth speaker (required)
- `BT_CHECK_INTERVAL`: Connection check interval in seconds (default: 30)
- `BT_TIMEOUT`: Bluetooth scan timeout in seconds (default: 15)

## Building Images

The project includes GitHub Actions workflow to automatically build and push multi-platform Docker images (AMD64 and ARM64).

Manual build:
```bash
cd bluetooth-reconnector
docker build -t pinkynrg/bluetooth-speaker-reconnector:latest .

cd ../snapclient
docker build -t pinkynrg/snapclient:latest .
```
