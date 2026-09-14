# Load Balancer App

A Docker-based load balancing demonstration using Nginx that distributes traffic across two backend web servers.

## Project Overview

This is an educational load balancing application that showcases how Nginx can be used as a reverse proxy to distribute incoming requests across multiple backend servers. The architecture demonstrates:

- **Load Balancer**: Nginx container serving as a reverse proxy
- **Backend Servers**: Two Nginx containers serving static content
- **Request Distribution**: Round-robin load balancing between backends

## Architecture

```
Client Requests
    ↓
Load Balancer (Nginx on port 80)
    ↓
    ├─→ Web Server 1 (port 80, serves ./one/)
    └─→ Web Server 2 (port 80, serves ./two/)
```

The load balancer is configured via `/conf/default.conf` and routes all incoming requests to an upstream backend group containing both web servers.

## Project Structure

```
/app
├── docker-compose.yml    # Docker services configuration
├── conf/
│   └── default.conf      # Nginx load balancer configuration
├── one/
│   └── index.html        # Frontend for container 1
├── two/
│   └── index.html        # Frontend for container 2
└── README.md             # This file
```

## Setup Instructions

### Prerequisites

- Docker
- Docker Compose

### Installation & Running

1. Navigate to the project directory:
   ```bash
   cd /app
   ```

2. Start the containers:
   ```bash
   docker-compose up -d
   ```

3. Access the load balancer:
   ```bash
   curl http://localhost
   ```

   Or open `http://localhost` in your browser. Each request will be distributed to either Container 1 or Container 2.

4. Stop the containers:
   ```bash
   docker-compose down
   ```

## How It Works

### Load Balancer Configuration

The Nginx load balancer (`conf/default.conf`) is configured with:

- **Upstream backend group**: Defines `web1:80` and `web2:80` as backend servers
- **Round-robin distribution**: Requests are distributed sequentially to each backend
- **Error handling**: `proxy_next_upstream` directive retries on timeout/error
- **Proxy pass**: All requests to `/` are forwarded to the upstream backend

### Container Details

| Service | Image | Purpose | Volume Mount |
|---------|-------|---------|--------------|
| `lb` | nginx:latest | Reverse proxy/load balancer | `conf/default.conf` → `/etc/nginx/conf.d/default.conf` |
| `web1` | nginx:latest | Backend server 1 | `one/` → `/usr/share/nginx/html` |
| `web2` | nginx:latest | Backend server 2 | `two/` → `/usr/share/nginx/html` |

## Testing Load Balancing

To verify load balancing is working, send multiple requests and observe which container responds:

```bash
# Send multiple requests
for i in {1..10}; do curl http://localhost; echo ""; done
```

You should see alternating responses from "Container 1" and "Container 2", demonstrating successful round-robin distribution.

## Customization

### Adding More Backends

To add additional backend servers:

1. Add a new service in `docker-compose.yml`:
   ```yaml
   web3:
     image: nginx:latest
     volumes:
       - "./three:/usr/share/nginx/html"
   ```

2. Update `conf/default.conf` upstream block:
   ```nginx
   upstream backend {
       server web1:80;
       server web2:80;
       server web3:80;
   }
   ```

3. Restart containers: `docker-compose restart lb`

### Modifying Backend Content

Edit `one/index.html` or `two/index.html` to change what each container serves. Changes are reflected immediately in running containers.

### Load Balancing Algorithm

The current configuration uses round-robin (default). Other algorithms supported by Nginx include:
- `least_conn` - Least connections
- `ip_hash` - Based on client IP
- `random` - Random server selection
- `weighted` - With `weight` parameter per server

Example (least connections):
```nginx
upstream backend {
    least_conn;
    server web1:80;
    server web2:80;
}
```

## Troubleshooting

### Containers not connecting
Ensure Docker network allows container-to-container communication. Docker Compose creates a default network automatically.

### Slow responses
Check if both backend servers are running: `docker-compose ps`

### Configuration not updating
After modifying `conf/default.conf`, reload the load balancer:
```bash
docker-compose restart lb
```
