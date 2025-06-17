# PostgreSQL Role

A comprehensive Ansible role for installing, configuring, and managing PostgreSQL databases for the Trackify application infrastructure.

## Description

This role provides a complete PostgreSQL deployment solution with:
- PostgreSQL 14 installation and configuration
- Automated database and user creation
- Performance tuning and optimization
- Security hardening with proper authentication
- Monitoring integration with Prometheus PostgreSQL Exporter
- Automated backup and retention policies
- Firewall configuration for network security
- SSL/TLS support for production environments

## Requirements

- **Ansible**: >= 2.9
- **Target OS**: AlmaLinux 9.x / RHEL 9.x / CentOS 9
- **Python**: 3.6+ with psycopg2 module
- **Privileges**: Root/sudo access on target hosts
- **Network**: Firewalld service available
- **Collections**: 
  - `community.postgresql`
  - `community.general`
  - `ansible.posix`

Install required collections:
```bash
ansible-galaxy install -r requirements.yml
```

## Role Variables

### Required Variables

These variables must be defined in your inventory or group_vars:

```yaml
# Database credentials (store in vault)
trackify_db_password: "{{ vault_trackify_db_password }}"
postgresql_exporter_password: "{{ vault_postgres_exporter_password }}"
```

### Default Variables

Available in `defaults/main.yml`:

#### Basic Configuration
```yaml
postgresql_version: "14"                    # PostgreSQL version
postgresql_port: 5432                       # Database port
trackify_db_name: "trackify"               # Application database name
trackify_db_user: "trackify_user"          # Application database user
```

#### Performance Tuning
```yaml
postgresql_max_connections: 200            # Maximum concurrent connections
postgresql_shared_buffers: "256MB"         # Shared buffer size
postgresql_effective_cache_size: "1GB"     # Effective cache size
postgresql_work_mem: "4MB"                 # Work memory per query
postgresql_maintenance_work_mem: "64MB"    # Memory for maintenance operations
```

#### Security & Authentication
```yaml
postgresql_listen_addresses: "*"           # Listen addresses (* for all)
postgresql_password_encryption: "scram-sha-256"  # Password encryption method
postgresql_ssl: false                      # Enable SSL (set to true for production)
```

#### Monitoring
```yaml
postgresql_monitoring_enabled: true        # Enable Prometheus monitoring
postgresql_exporter_version: "0.12.0"     # PostgreSQL Exporter version
postgresql_exporter_port: 9187            # Exporter metrics port
```

#### Backup Configuration
```yaml
postgresql_backup_enabled: true           # Enable automated backups
postgresql_backup_time: "02:00"          # Daily backup time (HH:MM)
postgresql_backup_retention_days: 30     # Backup retention period
postgresql_backup_dir: "/backup/postgresql"  # Backup directory
```

#### Network & Firewall
```yaml
postgresql_firewall_enabled: true         # Configure firewall rules
postgresql_internal_networks:             # Allowed internal networks
  - "192.168.1.0/24"
postgresql_dmz_networks:                  # Allowed DMZ networks  
  - "192.168.100.0/24"
```

#### Logging
```yaml
postgresql_log_min_duration_statement: 1000  # Log queries longer than (ms)
postgresql_log_statement: "ddl"             # Statement types to log
postgresql_log_line_prefix: "%t [%p]: [%l-1] user=%u,db=%d,app=%a,client=%h "
```

### Host-Specific Variables

Override defaults in `host_vars/` for environment-specific tuning:

**Development (`host_vars/devdb01.yml`)**:
```yaml
postgresql_max_connections: 100
postgresql_shared_buffers: "128MB"
postgresql_log_statement: "all"
postgresql_backup_retention_days: 7
```

**Production (`host_vars/prddb01.yml`)**:
```yaml
postgresql_max_connections: 300
postgresql_shared_buffers: "512MB"
postgresql_ssl: true
postgresql_ssl_cert_file: "/etc/ssl/certs/postgresql.crt"
postgresql_backup_retention_days: 90
```

### Advanced Configuration

#### Custom Databases and Users
```yaml
postgresql_databases:
  - name: "trackify"
    encoding: "UTF8"
    lc_collate: "en_US.UTF-8"
    lc_ctype: "en_US.UTF-8"
  - name: "analytics"
    encoding: "UTF8"

postgresql_users:
  - name: "trackify_user"
    password: "{{ vault_trackify_db_password }}"
    db: "trackify"
    priv: "ALL"
  - name: "analytics_user"
    password: "{{ vault_analytics_db_password }}"
    db: "analytics"
    priv: "ALL"
```

#### Autovacuum Configuration
```yaml
postgresql_autovacuum: true
postgresql_autovacuum_max_workers: 3
postgresql_autovacuum_naptime: "1min"
postgresql_autovacuum_vacuum_scale_factor: 0.2
```

## Dependencies

This role has no dependencies on other Ansible roles, but it integrates well with:
- Monitoring roles (Prometheus, Grafana)
- Application roles (Spring Boot applications)
- Load balancer roles (Traefik, NGINX)

## Example Playbook

### Basic Usage

```yaml
---
- hosts: postgresql
  become: yes
  roles:
    - role: postgresql
      vars:
        trackify_db_password: "{{ vault_trackify_db_password }}"
        postgresql_exporter_password: "{{ vault_postgres_exporter_password }}"
```

### Advanced Usage with Custom Configuration

```yaml
---
- hosts: postgresql
  become: yes
  roles:
    - role: postgresql
      vars:
        # Database configuration
        trackify_db_password: "{{ vault_trackify_db_password }}"
        postgresql_exporter_password: "{{ vault_postgres_exporter_password }}"
        
        # Performance tuning
        postgresql_max_connections: 500
        postgresql_shared_buffers: "1GB"
        postgresql_effective_cache_size: "4GB"
        
        # Enable SSL for production
        postgresql_ssl: true
        postgresql_ssl_cert_file: "/etc/ssl/certs/postgresql.crt"
        postgresql_ssl_key_file: "/etc/ssl/private/postgresql.key"
        
        # Custom backup schedule
        postgresql_backup_time: "01:30"
        postgresql_backup_retention_days: 60
        
        # Multiple databases
        postgresql_databases:
          - name: "trackify"
            encoding: "UTF8"
          - name: "analytics"
            encoding: "UTF8"
        
        postgresql_users:
          - name: "trackify_user"
            password: "{{ vault_trackify_db_password }}"
            db: "trackify"
            priv: "ALL"
          - name: "analytics_user"
            password: "{{ vault_analytics_db_password }}"
            db: "analytics"
            priv: "ALL"
```

### Environment-Specific Deployment

```yaml
---
# Deploy to development
- hosts: devdb01
  become: yes
  roles:
    - role: postgresql
      tags: development

# Deploy to production with enhanced security
- hosts: prddb01
  become: yes
  roles:
    - role: postgresql
      tags: production
      vars:
        postgresql_ssl: true
        postgresql_log_statement: "ddl"
        postgresql_backup_retention_days: 90
```

## File Structure

```
roles/postgresql/
├── README.md                      # This file
├── meta/main.yml                  # Role metadata
├── defaults/main.yml              # Default variables
├── vars/
│   ├── main.yml                   # Internal variables
│   └── RedHat.yml                 # OS-specific variables
├── handlers/main.yml              # Service handlers
├── tasks/
│   ├── main.yml                   # Main task coordinator
│   ├── install.yml                # Installation tasks
│   ├── configure.yml              # Configuration tasks
│   ├── databases.yml              # Database creation tasks
│   ├── monitoring.yml             # Monitoring setup tasks
│   ├── backup.yml                 # Backup configuration tasks
│   └── firewall.yml               # Firewall configuration tasks
└── templates/
    ├── postgresql.conf.j2         # Main PostgreSQL configuration
    ├── pg_hba.conf.j2            # Authentication configuration
    ├── postgres_exporter.env.j2   # Exporter environment file
    ├── postgres_exporter.service.j2 # Systemd service file
    ├── postgresql.logrotate.j2    # Log rotation configuration
    └── postgresql-backup.sh.j2    # Backup script template
```

## Usage Examples

### Deploy PostgreSQL with Monitoring

```bash
# Deploy PostgreSQL to all database servers
ansible-playbook -i inventory/hosts.yml playbooks/postgresql.yml

# Deploy only to development environment
ansible-playbook -i inventory/hosts.yml playbooks/postgresql.yml -l devdb01

# Deploy with specific tags
ansible-playbook -i inventory/hosts.yml playbooks/postgresql.yml --tags postgresql_install,postgresql_config
```

### Common Administrative Tasks

```bash
# Run backup manually
ansible postgresql -i inventory/hosts.yml -m shell -a "/usr/local/bin/postgresql-backup.sh" -u postgres

# Check PostgreSQL status
ansible postgresql -i inventory/hosts.yml -m systemd -a "name=postgresql-14 state=started"

# Verify monitoring endpoint
ansible postgresql -i inventory/hosts.yml -m uri -a "url=http://localhost:9187/metrics"

# Check database connectivity
ansible postgresql -i inventory/hosts.yml -m postgresql_ping -u postgres
```

### Maintenance Operations

```bash
# Run VACUUM ANALYZE on all databases
ansible-playbook -i inventory/hosts.yml playbooks/postgresql.yml --tags maintenance

# Update PostgreSQL configuration only
ansible-playbook -i inventory/hosts.yml playbooks/postgresql.yml --tags postgresql_config

# Restart PostgreSQL services
ansible postgresql -i inventory/hosts.yml -m systemd -a "name=postgresql-14 state=restarted"
```

## Security Considerations

1. **Vault Usage**: Always store passwords in Ansible Vault:
   ```bash
   ansible-vault create group_vars/postgresql/vault.yml
   ```

2. **SSL Configuration**: Enable SSL for production environments:
   ```yaml
   postgresql_ssl: true
   postgresql_ssl_cert_file: "/path/to/cert.crt"
   postgresql_ssl_key_file: "/path/to/private.key"
   ```

3. **Network Security**: Configure appropriate network restrictions:
   ```yaml
   postgresql_internal_networks:
     - "192.168.1.0/24"    # Backend network only
   postgresql_dmz_networks:
     - "192.168.100.0/24"  # DMZ for web servers
   ```

4. **Authentication**: Use strong password encryption:
   ```yaml
   postgresql_password_encryption: "scram-sha-256"
   ```

## Monitoring Integration

The role automatically sets up PostgreSQL Exporter for Prometheus monitoring:

- **Metrics endpoint**: `http://hostname:9187/metrics`
- **Grafana dashboards**: Compatible with standard PostgreSQL dashboards
- **Alerting**: Configure Prometheus alerts for database health

Example Prometheus configuration:
```yaml
- job_name: 'postgresql'
  static_configs:
    - targets: ['devdb01:9187', 'prddb01:9187']
```

## Backup and Recovery

### Automated Backups
- Daily backups at configured time
- Compressed SQL dumps with integrity verification
- Automatic cleanup based on retention policy
- Global backup for roles and tablespaces

### Manual Recovery
```bash
# List available backups
ls -la /backup/postgresql/

# Restore database
gunzip -c /backup/postgresql/trackify_20240429_020001.sql.gz | psql -U postgres -d trackify
```

## Troubleshooting

### Common Issues

1. **Connection Refused**
   ```bash
   # Check service status
   ansible postgresql -m systemd -a "name=postgresql-14"
   
   # Check firewall
   ansible postgresql -m shell -a "firewall-cmd --list-ports"
   
   # Verify pg_hba.conf
   ansible postgresql -m shell -a "cat /var/lib/pgsql/14/data/pg_hba.conf"
   ```

2. **Performance Issues**
   ```bash
   # Check current settings
   ansible postgresql -m shell -a "sudo -u postgres psql -c 'SHOW ALL;'" 
   
   # Monitor active connections
   ansible postgresql -m shell -a "sudo -u postgres psql -c 'SELECT count(*) FROM pg_stat_activity;'"
   ```

3. **Backup Failures**
   ```bash
   # Check backup logs
   ansible postgresql -m shell -a "tail -f /backup/postgresql/backup_*.log"
   
   # Test backup script manually
   ansible postgresql -m shell -a "/usr/local/bin/postgresql-backup.sh" -u postgres
   ```

### Log Locations
- **PostgreSQL logs**: `/var/log/postgresql/`
- **Backup logs**: `/backup/postgresql/backup_*.log`
- **Systemd logs**: `journalctl -u postgresql-14`
- **Exporter logs**: `journalctl -u postgres_exporter`

## Testing

### Verification Commands

```bash
# Test database connection
ansible postgresql -m postgresql_ping

# Verify databases exist
ansible postgresql -m shell -a "sudo -u postgres psql -l"

# Check user permissions
ansible postgresql -m shell -a "sudo -u postgres psql -d trackify -c '\du'"

# Test monitoring endpoint
ansible postgresql -m uri -a "url=http://localhost:9187/metrics"

# Verify backup functionality
ansible postgresql -m shell -a "test -f /usr/local/bin/postgresql-backup.sh && echo 'Backup script exists'"
```

## License

MIT
