# Blue/Green Deployment with Nginx Auto-Failover

## Quick Start

### Prerequisites
- Docker
- Docker Compose

### Setup

1. Clone the repository:
```bash
git clone https://github.com/YOUR_USERNAME/devops-stage2.git
cd devops-stage2
```

2. Start services:
```bash
docker-compose up -d
```

3. Check status:
```bash
docker-compose ps
```

4. Test baseline:
```bash
curl http://localhost:8080/version
```

### Testing Failover

1. Trigger chaos:
```bash
curl -X POST http://localhost:8081/chaos/start?mode=error
```

2. Test requests (should switch to Green):
```bash
for i in {1..10}; do curl -s http://localhost:8080/version | grep -o '"pool":"[^"]*"'; sleep 0.5; done
```

3. Stop chaos:
```bash
curl -X POST http://localhost:8081/chaos/stop
```

### Architecture
```
Client → Nginx (8080) → Blue (8081) [PRIMARY]
                      → Green (8082) [BACKUP]
```

### Configuration

Edit `.env` to change:
- Docker images
- Release IDs
- Port numbers

### Logs
```bash
docker-compose logs -f
```

### Cleanup
```bash
docker-compose down
```