# Implementation Decisions

## Failover Strategy

### Why Nginx `backup` directive?
- Simple and reliable
- No external dependencies
- Automatic failover without manual intervention
- Zero downtime for clients

### Timeout Configuration
```nginx
proxy_connect_timeout 2s;
proxy_read_timeout 2s;
max_fails=1 fail_timeout=5s;
```

**Rationale:**
- Task requires <10s total request time
- Need to detect failures and retry within single request
- Aggressive timeouts ensure fast failover (typically <3s)

## Docker Compose Design

### Service Dependencies
```yaml
nginx depends_on:
  - nginx_config (completed)
  - app_blue (healthy)
  - app_green (healthy)
```

This ensures:
1. Config is generated before Nginx starts
2. Both apps are healthy before accepting traffic

### Health Checks
- Interval: 5s (frequent monitoring)
- Timeout: 2s (fail fast)
- Start period: 10s (allow app startup)

## Why This Approach Works

1. **Primary/Backup Pattern**: Blue serves all traffic normally, Green is standby
2. **Fast Detection**: 2s timeouts detect failures quickly
3. **Automatic Retry**: Nginx retries on Green within same request
4. **Zero Client Errors**: Clients never see 5xx errors
5. **Simple Configuration**: Pure Nginx config, no custom code

## Alternative Approaches Considered

### 1. Active Health Checks
**Not chosen:** Requires nginx-plus or additional modules

### 2. Manual Switching Script
**Not chosen:** Task requires automatic failover

### 3. Load Balancing Both
**Not chosen:** Not true Blue/Green pattern