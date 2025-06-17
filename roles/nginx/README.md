
# NGINX Role for Trackify Frontend

This Ansible role installs and configures NGINX to serve the Trackify React frontend application with automatic building from Git repository.

## Requirements

- AlmaLinux 9 or compatible RHEL-based system
- Ansible 2.9+
- Git repository access for Trackify frontend
- Node.js 20+ (automatically installed)

## Features

- **Automatic Git Deployment**: Clones and builds from dev branch for development servers (devweb*) and main branch for production (prdweb*)
- **React Build Pipeline**: Installs Node.js, runs npm install and build
- **Traefik Integration**: Optimized for Traefik reverse proxy (uses real IP headers)
- **Static File Serving**: Optimized React app hosting with SPA routing
- **SPA Support**: Proper React Router fallback handling
- **Security Headers**: Comprehensive security headers and CSP
- **SSL Support**: Optional SSL with self-signed certificates for development
- **Caching**: Optimized caching for static assets
- **Rate Limiting**: API rate limiting protection

## Role Variables

### Basic Configuration
```yaml
nginx_user: nginx
nginx_group: nginx
nginx_configure_firewall: true
```

### Trackify Application
```yaml
trackify_git_repo: "https://github.com/your-org/trackify-frontend.git"
trackify_source_dir: /opt/trackify-frontend
trackify_web_root: /var/www/trackify
trackify_build_dir: "build"

# Automatically determined based on hostname
trackify_git_branch: "{{ 'dev' if inventory_hostname.startswith('dev') else 'main' }}"
trackify_production_build: "{{ not inventory_hostname.startswith('dev') }}"
```

### Node.js Configuration
```yaml
nodejs_version: "20"
nodejs_packages:
  - nodejs
  - npm
  - gcc-c++
  - make
  - git
```

## Directory Structure

```
/opt/trackify-frontend/     # Source code
/var/www/trackify/          # Built application
/etc/nginx/                 # NGINX configuration
/var/log/nginx/             # Logs
/etc/nginx/ssl/             # SSL certificates (if enabled)
```

## Example Playbook

### Basic Usage
```yaml
---
- hosts: webservers
  become: yes
  roles:
    - role: nginx
      vars:
        trackify_git_repo: "https://github.com/myorg/trackify-frontend.git"
```

### Production Configuration
```yaml
---
- hosts: prdweb01
  become: yes
  roles:
    - role: nginx
      vars:
        trackify_git_repo: "https://github.com/myorg/trackify-frontend.git"
```

### Development Configuration
```yaml
---
- hosts: devweb01
  become: yes
  roles:
    - role: nginx
      vars:
        trackify_git_repo: "https://github.com/myorg/trackify-frontend.git"
```

## Branch Strategy

The role automatically selects the appropriate Git branch:
- **Development servers** (`devweb*`): Uses `dev` branch
- **Production servers** (`prdweb*`): Uses `main` branch

## Build Process

1. **Clone Repository**: Fetches latest code from specified branch
2. **Install Dependencies**: Runs `npm install`
3. **Build Application**: Executes `npm run build`
4. **Deploy Files**: Copies built files to web root
5. **Set Permissions**: Ensures proper file ownership and permissions

## API Routing

All API requests are handled by Traefik:
```
Frontend: https://trackify.example.com/api/tickets  → Traefik → Spring Boot
Direct:   Frontend static files only                → NGINX
```

NGINX serves **only the React frontend static files**, while Traefik handles all API routing to the backend.

## Traefik Integration

NGINX is configured to work behind Traefik reverse proxy:
- **SSL Termination**: Handled by Traefik (*rpx01), NGINX serves HTTP only
- **Real IP Headers**: Uses `X-Forwarded-For` headers for actual client IPs
- **Rate Limiting**: Based on real client IPs from Traefik headers
- **Simple Configuration**: No SSL complexity at NGINX level

## Security Features

- **CSP Headers**: Content Security Policy protection
- **Rate Limiting**: API endpoint protection  
- **File Access Control**: Blocks access to sensitive files
- **Security Headers**: X-Frame-Options, XSS protection, etc.
- **Traefik Integration**: Proper real IP handling behind reverse proxy

## Maintenance

### Update Application
```bash
# Re-run the build tasks to pull latest code
ansible-playbook -i inventory site.yml --tags nginx_build

# Or just the specific host
ansible-playbook -i inventory site.yml --limit devweb01 --tags nginx_build
```

### Check Logs
```bash
# NGINX access and error logs
tail -f /var/log/nginx/trackify_access.log
tail -f /var/log/nginx/trackify_error.log

# Build logs are displayed during playbook execution
```

### Rebuild from Scratch
```bash
# Force complete rebuild
ansible-playbook -i inventory site.yml --tags nginx --extra-vars "force_rebuild=true"
```

## Troubleshooting

### Build Failures
```bash
# Check Node.js version
node --version
npm --version

# Manual build test
cd /opt/trackify-frontend
sudo -u nginx npm run build
```

### Static File Issues
```bash
# Check React build output
ls -la /var/www/trackify/
cat /var/www/trackify/index.html

# Test static file serving
curl -v http://localhost/
```

### Permission Issues
```bash
# Fix ownership
chown -R nginx:nginx /var/www/trackify
chown -R nginx:nginx /opt/trackify-frontend

# Check SELinux
setsebool -P httpd_can_network_connect 1
```

## Tags

- `nginx`: Complete NGINX setup
- `nginx_install`: Installation only
- `nginx_nodejs`: Node.js setup
- `nginx_build`: Application build
- `nginx_config`: Configuration files
- `nginx_firewall`: Firewall rules

## License

MIT