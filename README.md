# K3s + Rancher Automation

Deploy production-ready Kubernetes clusters with Rancher management in minutes using Ansible.

**What you get:**
- K3s cluster (single-node or HA)
- Rancher management UI
- Automatic TLS certificates
- Essential tools (kubectl, helm, k9s)

## Quick Start

1. **Configure your hosts** in `inventories/dev/hosts.yaml`
2. **Set variables** in `inventories/dev/group_vars/all.yaml`
3. **Deploy**: `ansible-playbook -i inventories/dev/hosts.yaml main.yaml`
4. **Access Rancher** at your configured hostname

## Prerequisites

**Target servers:**
- Ubuntu 18.04+ or CentOS 7+
- 4GB RAM, 2+ CPU cores, 20GB storage
- SSH access

**Local machine:**
```bash
pip install ansible kubernetes
ansible-galaxy collection install kubernetes.core community.kubernetes
```

## Configuration

### 1. Hosts Setup

**Single node** (`inventories/dev/hosts.yaml`):
```yaml
all:
  hosts:
    k3s-server:
      ansible_host: your-server-ip
      ansible_user: ubuntu
```

**HA cluster** (`inventories/production/hosts.yaml`):
```yaml
all:
  hosts:
    k3s-master-01:
      ansible_host: 10.0.1.10
      ansible_user: ubuntu
    k3s-master-02:
      ansible_host: 10.0.1.11
      ansible_user: ubuntu
```

### 2. Variables

**Essential settings** (`inventories/dev/group_vars/all.yaml`):
```yaml
# Basic config
rancher_hostname: "rancher.yourdomain.com"
rancher_bootstrap_password: "YourSecurePassword123!"

# TLS options: rancher, letsEncrypt, secret
rancher_tls_source: "letsEncrypt"
lets_encrypt_email: "admin@yourdomain.com"

# HA setup (requires external database)
ha_enabled: true
datastore_endpoint: "mysql://k3s_user:SecurePass123@tcp(mysql-server:3306)/k3s_db"
```

### 3. MySQL Database Setup (HA Only)

**Create database and user:**
```sql
CREATE DATABASE k3s_db;
CREATE USER 'k3s_user'@'%' IDENTIFIED BY 'SecurePass123';
GRANT ALL PRIVILEGES ON k3s_db.* TO 'k3s_user'@'%';
FLUSH PRIVILEGES;
```

**Grant access from specific IPs:**
```sql
GRANT ALL PRIVILEGES ON k3s_db.* TO 'k3s_user'@'10.0.1.10';
GRANT ALL PRIVILEGES ON k3s_db.* TO 'k3s_user'@'10.0.1.11';
FLUSH PRIVILEGES;
```

## Deployment

```bash
# Test connection
ansible -i inventories/dev/hosts.yaml all -m ping

# Deploy cluster
ansible-playbook -i inventories/dev/hosts.yaml main.yaml

# Production deployment
ansible-playbook -i inventories/production/hosts.yaml main.yaml
```

## Access & Verification

**Rancher UI:** `https://your-rancher-hostname`
- Login: `admin` / `your-bootstrap-password`
- Set new admin password on first login

**Verify cluster:**
```bash
sudo k3s kubectl get nodes
sudo k3s kubectl get pods -n cattle-system
```

## Cleanup

```bash
# Automated cleanup
ansible-playbook -i inventories/dev/hosts.yaml uninstall.yaml

# Manual cleanup (if needed)
sudo /usr/local/bin/k3s-uninstall.sh
```

## Advanced Usage

### Create Additional Clusters
```bash
# Deploy new clusters via Rancher UI
ansible-playbook -i inventories/dev/hosts.yaml cluster-deploy.yaml

# Import existing clusters
ansible-playbook -i inventories/dev/hosts.yaml cluster-import.yaml
```

## Customization

### TLS Options
```yaml
# Auto-generated (default)
rancher_tls_source: "rancher"

# Let's Encrypt
rancher_tls_source: "letsEncrypt"
lets_encrypt_email: "admin@example.com"

# Custom certificates
rancher_tls_source: "secret"
tls_cert_file: "/path/to/cert.crt"
tls_key_file: "/path/to/key.key"
```

### HA Configuration
```yaml
ha_enabled: true
datastore_endpoint: "mysql://user:pass@tcp(host:3306)/db"
```

**HA Deployment Process:**
1. Setup MySQL database with proper user permissions
2. First node becomes primary master with external datastore
3. Secondary nodes join using token from primary
4. Management tools installed only on primary node

**Database Requirements:**
- MySQL 5.7+ or 8.0+
- Network accessible from all K3s nodes
- User with full privileges on K3s database

## Security Notes

- Change default passwords immediately
- Use Let's Encrypt or custom TLS in production
- Restrict network access to Rancher UI
- Keep clusters updated

## Troubleshooting

### K3s Service Issues

**Check cluster status:**
```bash
sudo k3s kubectl get nodes
sudo k3s kubectl get pods -n cattle-system
journalctl -u k3s -f
```

**Service won't start:**
```bash
# Check service status
systemctl status k3s

# View detailed logs
journalctl -u k3s --no-pager

# Manual cleanup if needed
sudo /usr/local/bin/k3s-uninstall.sh
```

### MySQL Database Issues

**Test database connectivity:**
```bash
# From K3s nodes
mysql -h mysql-server -P 3306 -u k3s_user -p k3s_db
```

**Common database errors:**

1. **Access denied for user:**
```sql
-- Connect as root and grant permissions
GRANT ALL PRIVILEGES ON k3s_db.* TO 'k3s_user'@'node-ip';
FLUSH PRIVILEGES;
```

2. **Bootstrap data encrypted with different token:**
```sql
-- Clear existing data
USE k3s_db;
DROP TABLE IF EXISTS kine;
-- Or recreate database
DROP DATABASE k3s_db;
CREATE DATABASE k3s_db;
```

3. **Connection refused:**
- Check MySQL is running: `systemctl status mysql`
- Verify port 3306 is open
- Check bind-address in MySQL config

**MySQL Configuration:**
```ini
# /etc/mysql/mysql.conf.d/mysqld.cnf
[mysqld]
bind-address = 0.0.0.0
port = 3306
max_connections = 200
```

### Network & Firewall

**Required ports:**
- 6443: Kubernetes API
- 80/443: Rancher UI
- 3306: MySQL (HA only)

**Check connectivity:**
```bash
# Test API server
curl -k https://rancher-hostname:6443

# Test MySQL
telnet mysql-server 3306
```

### Certificate Issues

**Check certificates:**
```bash
sudo k3s kubectl get certificates -A
sudo k3s kubectl describe certificate rancher -n cattle-system
```

**Let's Encrypt issues:**
```bash
# Check cert-manager logs
sudo k3s kubectl logs -n cert-manager deployment/cert-manager

# Verify DNS resolution
nslookup rancher.yourdomain.com
```

### Common Fixes

- **DNS resolution**: Ensure hostname resolves to server IP
- **Firewall**: Allow ports 80, 443, 6443, 3306
- **Database permissions**: Grant access from all K3s node IPs
- **Clean installation**: Use uninstall script before retry
- **Token mismatch**: Clear database tables for fresh start

## Database Setup Examples

### MySQL with Docker
```bash
docker run -d \
  --name mysql-k3s \
  -e MYSQL_ROOT_PASSWORD=rootpass \
  -e MYSQL_DATABASE=k3s_db \
  -e MYSQL_USER=k3s_user \
  -e MYSQL_PASSWORD=SecurePass123 \
  -p 3306:3306 \
  mysql:8.0
```

### MySQL on Ubuntu
```bash
# Install MySQL
sudo apt update
sudo apt install mysql-server

# Secure installation
sudo mysql_secure_installation

# Create database
sudo mysql -e "CREATE DATABASE k3s_db;"
sudo mysql -e "CREATE USER 'k3s_user'@'%' IDENTIFIED BY 'SecurePass123';"
sudo mysql -e "GRANT ALL PRIVILEGES ON k3s_db.* TO 'k3s_user'@'%';"
sudo mysql -e "FLUSH PRIVILEGES;"
```

## Resources

- [K3s Documentation](https://docs.k3s.io/)
- [Rancher Documentation](https://rancher.com/docs/)
- [Ansible Documentation](https://docs.ansible.com/)
- [MySQL Documentation](https://dev.mysql.com/doc/)