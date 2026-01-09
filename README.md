# Wyze to UniFi Protect Bridge

Integrates Wyze cameras into UniFi Protect by converting Wyze cloud streams to RTSP using [docker-wyze-bridge](https://github.com/mrlt8/docker-wyze-bridge) and making them appear as native UniFi cameras using [unifi-cam-proxy](https://github.com/keshavdv/unifi-cam-proxy).

## Features

- Converts Wyze camera streams to RTSP/HLS/WebRTC
- Makes Wyze cameras appear as native UniFi Protect cameras
- Supports audio streaming
- Works with UniFi Protect 6.x
- Supports multiple cameras

## Requirements

- Docker and Docker Compose
- Wyze account with cameras
- UniFi Protect controller (UDM Pro, Cloud Key, etc.)
- Network access between the bridge host and both Wyze cloud and UniFi Protect

## Quick Start

1. Clone this repository:
   ```bash
   git clone https://github.com/yourusername/wyze-unifi-bridge.git
   cd wyze-unifi-bridge
   ```

2. Copy `.env.example` to `.env` and fill in your credentials:
   ```bash
   cp .env.example .env
   nano .env
   ```

3. Start the wyze-bridge first to discover cameras:
   ```bash
   docker-compose up -d wyze-bridge
   ```

4. Check the web UI at `http://your-host-ip:5000` to see discovered cameras and their stream names

5. Update `docker-compose.yml` with your camera details:
   - Change `--name` to your desired camera name in UniFi Protect
   - Change `--mac` to a unique MAC address for each camera
   - Update the RTSP URL with the camera name from the web UI

6. Get an adoption token from UniFi Protect:
   - Go to UniFi Protect web UI
   - Navigate to **Settings > System > Advanced**
   - Find the adoption token or generate one

7. Start all services:
   ```bash
   docker-compose up -d
   ```

## Configuration

### Environment Variables

| Variable | Description |
|----------|-------------|
| `WYZE_EMAIL` | Your Wyze account email |
| `WYZE_PASSWORD` | Your Wyze account password |
| `WYZE_API_ID` | Wyze API ID (required for 2FA) |
| `WYZE_API_KEY` | Wyze API Key (required for 2FA) |
| `WYZE_REFRESH_TOKEN` | Refresh token for Apple SSO users |
| `UNIFI_PROTECT_IP` | IP address of your UniFi Protect controller |
| `UNIFI_ADOPT_TOKEN` | Adoption token from UniFi Protect |
| `HOST_IP` | IP address of the machine running this bridge |

### Getting Wyze API Keys

If you use 2FA or have authentication issues:

1. Go to https://developer-api-console.wyze.com/
2. Create an API key
3. Add `WYZE_API_ID` and `WYZE_API_KEY` to your `.env` file

### Adding Multiple Cameras

For each camera, duplicate the `unifi-cam-proxy-example` service in `docker-compose.yml`:

```yaml
  unifi-cam-proxy-frontdoor:
    image: keshavdv/unifi-cam-proxy:dev
    container_name: unifi-cam-proxy-frontdoor
    restart: unless-stopped
    entrypoint: ["unifi-cam-proxy"]
    command:
      - "--host=${UNIFI_PROTECT_IP}"
      - "--cert=/data/client.pem"
      - "--token=${UNIFI_ADOPT_TOKEN}"
      - "--name=Front Door"
      - "--mac=AA:BB:CC:DD:EE:02"  # Must be unique!
      - "--ip=${HOST_IP}"
      - "--model=UVC G3 Flex"
      - "--verbose"
      - "rtsp"
      - "-s"
      - "rtsp://${HOST_IP}:8554/front-door"
    volumes:
      - ./data/unifi-cam-proxy-frontdoor:/data
    network_mode: host
    depends_on:
      - wyze-bridge
```

## Stream URLs

| Protocol | URL Format |
|----------|------------|
| RTSP | `rtsp://<host>:8554/<camera-name>` |
| HLS | `http://<host>:8888/<camera-name>/stream.m3u8` |
| WebRTC | `http://<host>:8889/<camera-name>` |

## Troubleshooting

### Camera not appearing in wyze-bridge
- Verify Wyze credentials in `.env`
- Check if 2FA is required (you need API keys)
- Ensure camera is online in the Wyze app
- Check logs: `docker logs wyze-bridge`

### Camera not appearing in UniFi Protect
- Verify the adoption token is valid (they expire after ~60 minutes)
- Check that the MAC address is unique for each camera
- Ensure network connectivity to UniFi Protect
- Check logs: `docker logs unifi-cam-proxy-example`

### Broken pipe errors
- Use the `:dev` image tag for unifi-cam-proxy (required for UniFi Protect 6.x)

### Choppy video playback
- Wyze cameras use WiFi 4 (802.11n) chips with limited throughput (~180kb/s)
- Move camera closer to your WiFi access point
- This is a hardware limitation, not a software issue

### 401 Unauthorized on RTSP
- The `WB_AUTH=False` setting in docker-compose.yml disables authentication
- This is required for unifi-cam-proxy to access the streams

## Credits

- [docker-wyze-bridge](https://github.com/mrlt8/docker-wyze-bridge) by mrlt8
- [unifi-cam-proxy](https://github.com/keshavdv/unifi-cam-proxy) by keshavdv

## License

MIT License - See LICENSE file for details
