# Keycloak Identity and Access Management Role

This Ansible role installs and configures Keycloak as an Identity and Access Management (IAM) solution for the Trackify application infrastructure.

## Requirements

- AlmaLinux 9 or compatible RHEL-based system
- Ansible 2.9+
- Java 21 (automatically installed)
- PostgreSQL database (configured separately)
- Firewall access for port 8080 (HTTP) and 8443 (HTTPS)

## Features

- **Identity Management**: User authentication and authorization
- **Single Sign-On (SSO)**: Centralized authentication across applications
- **OAuth2/OIDC Support**: Modern authentication protocols
- **RBAC**: Role-based access control
- **Database Integration**: PostgreSQL backend for persistence
- **SSL/TLS Support**: Secure communications
- **Monitoring Integration**: Health checks and metrics
- **High Performance**: Optimized JVM settings
- **Security Hardening**: Production-ready security configuration

## Role Variables

### Basic Configuration
```yaml
keycloak_version: "23.0.0"
keycloak_user: keycloak
keycloak_group: keycloak
keycloak_home_dir: /opt/keycloak
```

### Database Configuration
```yaml
keycloak_db_vendor: postgres
keycloak_db_host: "{{ groups['tier_db'][0] if groups['tier_db'] is defined else 'localhost' }}"
keycloak_db_port: 5432
keycloak_db_name: keycloak
keycloak_db_user: keycloak_user
keycloak_db_password: "{{ vault_keycloak_db_password }}"
```

### Admin Configuration
```yaml
keycloak_admin_user: admin
keycloak_admin_password: "{{ vault_keycloak_admin_password }}"
```

### Network Configuration
```yaml
keycloak_bind_address: 0.0.0.0
keycloak_port: 8080
keycloak_ssl_port: 8443
keycloak_hostname: "{{ inventory_hostname }}.jokulab.ch"
```

## Example Playbook

```yaml
---
- hosts: keycloak
  become: yes
  roles:
    - role: keycloak
      vars:
        keycloak_admin_password: "{{ vault_keycloak_admin_password }}"
        keycloak_db_password: "{{ vault_keycloak_db_password }}"
```

## Integration with Trackify

Keycloak provides authentication for:
- Trackify Web Application (Frontend)
- Trackify REST API (Backend)
- MinIO Object Storage
- Administrative interfaces

## Security Features

- HTTPS/TLS encryption
- Strong password policies
- Session management
- Account lockout protection
- Audit logging
- CSRF protection

## License

MIT