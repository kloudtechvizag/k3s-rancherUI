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
    k3s-master-02:
      ansible_host: 10.0.1.11
    k3s-master-03:
      ansible_host: 10.0.1.12
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

# HA setup (production only)
ha_enabled: true
k3s_datastore_endpoint: "mysql://user:pass@tcp(host:3306)/db"
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
k3s_datastore_endpoint: "mysql://user:pass@tcp(host:3306)/db"
```

## Security Notes

- Change default passwords immediately
- Use Let's Encrypt or custom TLS in production
- Restrict network access to Rancher UI
- Keep clusters updated

## Troubleshooting

**Check cluster status:**
```bash
sudo k3s kubectl get pods -n cattle-system
sudo k3s kubectl get certificates -A
journalctl -u k3s
```

**Common fixes:**
- Ensure DNS resolves to your server
- Check firewall allows ports 80/443
- Verify TLS certificate configuration

## Resources

- [K3s Documentation](https://docs.k3s.io/)
- [Rancher Documentation](https://rancher.com/docs/)
- [Ansible Documentation](https://docs.ansible.com/)