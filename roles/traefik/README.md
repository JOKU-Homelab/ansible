
# Traefik Reverse Proxy Role for Trackify

This Ansible role installs and configures Traefik as a reverse proxy and load balancer for the Trackify application infrastructure.

## Requirements

- AlmaLinux 9 or compatible RHEL-based system
- Ansible 2.9+
- DNS records pointing to the Traefik server
- Firewall access for HTTP (80) and HTTPS (443) ports

## Features

- **SSL Termination**: Automatic SSL certificate management with Let's Encrypt
- **Load Balancing**: Distributes traffic across backend services
- **Service Discovery**: Dynamic routing configuration
- **Health Checks**: Monitors backend service health
- **Dashboard**: Web-based management interface
- **Security Headers**: Comprehensive security header configuration
- **Rate Limiting**: Protects against abuse and DDoS
- **Monitoring**: Prometheus metrics integration
- **Logging**: Structured access and application logging

## Role Variables

### Basic Configuration
```yaml
traefik_version: "3.0.4"
traefik_user: traefik
traefik_group: traefik
traefik_domain: "jokulab.ch"
```

### SSL/TLS Configuration
```yaml
traefik_ssl_enabled: true
traefik_letsencrypt_enabled: true
traefik_letsencrypt_staging: false
traefik_letsencrypt_email: "admin@jokulab.ch"
```

### Dashboard Configuration
```yaml
traefik_dashboard_enabled: true
traefik_dashboard_port: 8080
traefik_dashboard_auth_enabled: true
traefik_dashboard_user: admin
traefik_dashboard_password: "{{ vault_traefik_dashboard_password }}"
```

### Service Routing
```yaml
traefik_backend_services:
  trackify-frontend:
    domain: "trackify.jokulab.ch"
    backend_url: "http://prdweb01:80"
    health_check: "/health"
  
  trackify-api:
    domain: "trackify.jokulab.ch"
    backend_url: "http://prdapp01:8080"
    path_prefix: "/api"
    health_check: "/api/health"
```

## Directory Structure

```
/etc/traefik/                  # Configuration files
├── traefik.yml               # Main configuration
├── dynamic/                  # Dynamic configuration
│   └── dynamic.yml          # Service definitions
├── ssl/                     # SSL certificates
└── .htpasswd               # Dashboard authentication

/var/lib/traefik/            # Data directory
├── acme.json               # Let's Encrypt certificates
└── ...

/var/log/traefik/           # Log files
├── traefik.log            # Application logs
└── access.log             # Access logs
```

## Example Playbook

### Basic Usage
```yaml
---
- hosts: reverseproxy
  become: yes
  roles:
    - role: traefik
      vars:
        traefik_domain: "jokulab.ch"
        traefik_letsencrypt_email: "admin@jokulab.ch"
```

### Production Configuration
```yaml
---
- hosts: prdrpx01
  become: yes
  roles:
    - role: traefik
      vars:
        traefik_domain: "jokulab.ch"
        traefik_letsencrypt_enabled: true
        traefik_letsencrypt_staging: false
        traefik_dashboard_password: "{{ vault_traefik_dashboard_password }}"
```

### Development Configuration
```yaml
---
- hosts: devrpx01
  become: yes
  roles:
    - role: traefik
      vars:
        traefik_domain: "jokulab.ch"
        traefik_letsencrypt_staging: true
        traefik_log_level: DEBUG
```

## Service Routing Configuration

Traefik automatically routes traffic based on domain and path:

### Frontend Routes
- `https://trackify.jokulab.ch/` → `http://prdweb01:80/`
- `https://trackify-dev.jokulab.ch/` → `http://devweb01:80/`

### API Routes
- `https://trackify.jokulab.ch/api/*` → `http://prdapp01:8080/api/*`
- `https://trackify-dev.jokulab.ch/api/*` → `http://devapp01:8080/api/*`

### Dashboard
- `https://traefik.jokulab.ch/` → Traefik dashboard

## SSL Certificate Management

### Let's Encrypt (Production)
```yaml
traefik_letsencrypt_enabled: true
traefik_letsencrypt_staging: false
traefik_letsencrypt_email: "admin@jokulab.ch"
```

### Let's Encrypt Staging (Testing)
```yaml
traefik_letsencrypt_enabled: true
traefik_letsencrypt_staging: true
```

### Self-Signed Certificates (Development)
```yaml
traefik_letsencrypt_enabled: false
traefik_generate_selfsigned_cert: true
```

## Security Features

### Security Headers
- X-Frame-Options: DENY
- X-Content-Type-Options: nosniff
- X-XSS-Protection: 1; mode=block
- Referrer-Policy: strict-origin-when-cross-origin
- HSTS with preload (for HTTPS)

### Rate Limiting
```yaml
traefik_rate_limit_enabled: true
traefik_rate_limit_burst: 100
traefik_rate_limit_average: 50
```

### CORS Support
Automatic CORS headers for API endpoints:
- Access-Control-Allow-Origin
- Access-Control-Allow-Methods
- Access-Control-Max-Age

## Monitoring

### Prometheus Metrics
```yaml
traefik_metrics_enabled: true
traefik_metrics_prometheus_enabled: true
```

Metrics available at: `http://traefik-server:8080/metrics`

### Health Checks
Automatic health checking for backend services:
- Interval: 30 seconds
- Timeout: 5 seconds
- Configurable per service

## Maintenance

### Check Traefik Status
```bash
systemctl status traefik
journalctl -u traefik -f
```

### View Logs
```bash
# Application logs
tail -f /var/log/traefik/traefik.log

# Access logs
tail -f /var/log/traefik/access.log

# Systemd logs
journalctl -u traefik -f
```

### Update Configuration
```bash
# Test configuration
/usr/local/bin/traefik validate --configfile=/etc/traefik/traefik.yml

# Reload configuration (for dynamic changes)
systemctl reload traefik

# Restart service (for static configuration changes)
systemctl restart traefik
```

### Certificate Management
```bash
# Check Let's Encrypt certificates
cat /var/lib/traefik/acme.json | jq '.letsencrypt.Certificates[] | {main: .domain.main, sans: .domain.sans}'

# Force certificate renewal (if needed)
systemctl stop traefik
rm /var/lib/traefik/acme.json
systemctl start traefik
```

## Troubleshooting

### Common Issues

1. **Certificate Generation Fails**
   ```bash
   # Check DNS resolution
   nslookup trackify.jokulab.ch
   
   # Verify domain points to this server
   dig +short trackify.jokulab.ch
   
   # Check Let's Encrypt rate limits
   tail -f /var/log/traefik/traefik.log | grep -i acme
   ```

2. **Backend Service Unreachable**
   ```bash
   # Test backend connectivity
   curl -v http://prdweb01:80/health
   curl -v http://prdapp01:8080/api/health
   
   # Check Traefik routing
   curl -H "Host: trackify.jokulab.ch" http://localhost/health
   ```

3. **Dashboard Access Issues**
   ```bash
   # Check dashboard authentication
   htpasswd -v /etc/traefik/.htpasswd admin
   
   # Test dashboard locally
   curl -u admin:password http://localhost:8080/api/overview
   ```

### Performance Tuning

#### High Traffic Optimization
```yaml
traefik_rate_limit_burst: 500
traefik_rate_limit_average: 200

# In traefik.yml template, add:
# serversTransport:
#   maxIdleConnsPerHost: 200
#   dialTimeout: 30s
#   responseHeaderTimeout: 0s
```

#### Memory Usage
```bash
# Monitor Traefik resource usage
systemctl status traefik
ps aux | grep traefik
```

## Security Hardening

### Network Security
```yaml
# Restrict dashboard access
traefik_firewall_zone: internal  # Only internal access
traefik_dashboard_auth_enabled: true
```

### SSL/TLS Hardening
```yaml
# Force modern TLS only
traefik_ssl_min_version: "VersionTLS13"
traefik_ssl_cipher_suites:
  - "TLS_AES_256_GCM_SHA384"
  - "TLS_CHACHA20_POLY1305_SHA256"
  - "TLS_AES_128_GCM_SHA256"
```

## Integration with Monitoring

### Prometheus Configuration
```yaml
# prometheus.yml
scrape_configs:
  - job_name: 'traefik'
    static_configs:
      - targets: ['prdrpx01:8080']
    metrics_path: '/metrics'
```

### Grafana Dashboards
Import Traefik dashboard ID: `11462` or `4475`

### Alerting Rules
```yaml
groups:
  - name: traefik
    rules:
      - alert: TraefikDown
        expr: up{job="traefik"} == 0
        for: 1m
        labels:
          severity: critical
        annotations:
          summary: "Traefik is down"
          
      - alert: TraefikHighErrorRate
        expr: rate(traefik_service_requests_total{code=~"5.."}[5m]) > 0.1
        for: 2m
        labels:
          severity: warning
        annotations:
          summary: "High error rate in Traefik"
```

## Advanced Configuration

### Custom Middleware
```yaml
# Add to dynamic.yml
middlewares:
  custom-headers:
    headers:
      customRequestHeaders:
        X-Custom-Header: "Trackify"
      customResponseHeaders:
        X-Version: "1.0"
```

### Path-based Routing
```yaml
traefik_backend_services:
  trackify-admin:
    domain: "trackify.jokulab.ch"
    backend_url: "http://prdapp01:8080"
    path_prefix: "/admin"
    strip_prefix: true
```

### TCP/UDP Load Balancing
```yaml
# For non-HTTP services (databases, etc.)
entryPoints:
  postgres:
    address: ":5432"

# Add TCP routing in dynamic configuration
```

## Backup and Recovery

### Backup Important Files
```bash
# Create backup script
tar -czf traefik-backup-$(date +%Y%m%d).tar.gz \
  /etc/traefik/ \
  /var/lib/traefik/acme.json \
  /var/log/traefik/
```

### Disaster Recovery
```bash
# Restore configuration
tar -xzf traefik-backup-*.tar.gz -C /

# Restore permissions
chown -R traefik:traefik /etc/traefik /var/lib/traefik
chmod 600 /var/lib/traefik/acme.json

# Restart service
systemctl restart traefik
```

## Tags

- `traefik`: Complete Traefik setup
- `traefik_install`: Installation only
- `traefik_config`: Configuration files
- `traefik_ssl`: SSL/TLS setup
- `traefik_firewall`: Firewall rules

## License

MIT

## Contributing

1. Test configuration changes in staging environment first
2. Validate SSL certificates work correctly
3. Monitor logs for errors after deployment
4. Update documentation for any custom configurations

## Support

For issues and questions:
- Check Traefik logs: `/var/log/traefik/traefik.log`
- Review configuration: `/etc/traefik/traefik.yml`
- Test connectivity: Use curl to test backend services
- Monitor metrics: Access Prometheus metrics at `:8080/metrics`