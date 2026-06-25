# Nextcloud Talk Signaling Helm Chart

This Helm chart deploys the **Nextcloud Talk Signaling Server** and **NATS message broker** into a Kubernetes cluster. 

It is designed for a decoupled architecture where **Janus** (WebRTC gateway) and **Coturn** (TURN/STUN server) are hosted on dedicated VMs or separate infrastructure, providing maximum scalability and stability.

## Architecture

- **Signaling Server**: Handles WebSockets and API requests from Nextcloud clients.
- **NATS**: Internal message broker used by the signaling server instances to coordinate states and messages.
- **External Dependencies**: Requires a pre-existing Nextcloud instance, Janus WebRTC server, and Coturn server.

## Prerequisites

- Kubernetes cluster (v1.19+)
- Helm 3.x
- Nextcloud Talk app installed and enabled on your Nextcloud instance
- Dedicated Janus Server (accessible via WebSocket)
- Dedicated Coturn Server
- Ingress controller (e.g., NGINX) configured for WebSocket support

---

## Dedicated VM Setup (Janus & Coturn)

If you have not yet set up your dedicated Janus and Coturn servers, you can follow this quick guide on a fresh Debian/Ubuntu VM.

### 1. Install Packages
```bash
sudo apt update && sudo apt upgrade -y
sudo apt install coturn janus -y
```

### 2. Configure Coturn
Edit the Coturn configuration file:
```bash
sudo nano /etc/turnserver.conf
```
Add or modify the following lines (replace `YOUR VM IP` with your server's actual IP address):
```ini
listening-port=3478
fingerprint
use-auth-secret
static-auth-secret=super-secret-turn-key-123
realm=YOUR VM IP
total-quota=100
bps-capacity=0
stale-nonce
no-multicast-peers
```

Enable the TURN server to run at startup:
```bash
sudo sed -i 's/#TURNSERVER_ENABLED=1/TURNSERVER_ENABLED=1/g' /etc/default/coturn
sudo systemctl restart coturn
sudo systemctl enable coturn
```

### 3. Configure Janus
Configure the core Janus settings:
```bash
sudo nano /etc/janus/janus.jcfg
```
Add the following blocks (replace `YOUR VM IP` with your server's actual IP address):
```ini
nat: {
    nat_1_1_mapping = "YOUR VM IP"
}
media: {
    rtp_port_range = "20000-40000"
}
```

Configure the WebSockets transport for Janus:
```bash
sudo nano /etc/janus/janus.transport.websockets.jcfg
```
Ensure the WebSocket server is enabled and listening on the correct port:
```ini
general: {
    ws = true
    ws_port = 8188
    ws_ip = "0.0.0.0" 
}
```

Restart and enable the Janus service:
```bash
sudo systemctl restart janus
sudo systemctl enable janus
```

---

## Configuration (`values.yaml`)

You must configure the `values.yaml` file with your specific environment details before deploying.

| Parameter | Description | Default |
|-----------|-------------|---------|
| `domain` | Your Nextcloud base URL | `nextcloud.yourdomain.com` |
| `janusUrl` | WebSocket URL for your dedicated Janus server | `ws://janus.yourdomain.com:8188` |
| `turnDomain` | Domain for your Coturn server | `turn.yourdomain.com` |
| `secrets.signalingSecret` | Shared secret between Nextcloud and Signaling Server | `YOUR_SIGNALING_SECRET` |
| `secrets.internalSecret` | Internal secret for signaling server/NATS | `YOUR_INTERNAL_SECRET` |
| `secrets.turnSecret` | Shared secret for Coturn | `YOUR_TURN_SECRET` |
| `ingress.enabled` | Enable ingress resource | `true` |
| `ingress.host` | Domain for the signaling server | `signaling.yourdomain.com` |

## Installation

1. Clone or download this repository.
2. Edit the `values.yaml` file to match your environment.
3. Install the chart using Helm:

```bash
helm install nc-talk ./nextcloud-signaling --namespace nextcloud --create-namespace
```

## Upgrading

To apply changes made to `values.yaml` or update the images:

```bash
helm upgrade nc-talk ./nextcloud-signaling --namespace nextcloud
```

## Health Checks & Troubleshooting

The signaling server uses the `/api/v1/welcome` endpoint for Kubernetes readiness and liveness probes. This ensures the probes bypass IP whitelisting restrictions that apply to the `/api/v1/stats` endpoint, preventing unnecessary `CrashLoopBackOff` errors.

If your pod fails to start, verify the logs:
```bash
kubectl logs -l app=signaling -n nextcloud
```