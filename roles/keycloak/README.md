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