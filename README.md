# K3s + Rancher Ansible Automation

Automated deployment of K3s Kubernetes cluster with Rancher management UI using Ansible. Supports single-node and HA configurations with TLS certificate management.

## 🏗️ Architecture

- **K3s**: Lightweight Kubernetes distribution
- **Rancher**: Kubernetes management platform
- **cert-manager**: Automatic TLS certificate management
- **Traefik**: Built-in ingress controller
- **Common Tools**: kubectl, helm, k9s

## 📁 Project Structure

```
k3s-rancherUI/
├── inventories/
│   ├── dev/
│   │   ├── hosts.yaml              # Development hosts
│   │   └── group_vars/all.yaml     # Dev configuration
│   └── production/
│       ├── hosts.yaml              # Production hosts
│       └── group_vars/all.yaml     # Prod configuration
├── roles/
│   ├── common_tools/               # kubectl, helm, k9s installation
│   ├── k3s_master/                 # K3s server setup
│   ├── cert_manager/               # Certificate management
│   └── rancher/                    # Rancher deployment
├── main.yaml                       # Main installation playbook
├── uninstall.yaml                  # Cleanup playbook
└── ansible.cfg                     # Ansible configuration
```

## 🔧 Prerequisites

### System Requirements
- **OS**: Ubuntu 18.04+, Amazon Linux 2, CentOS 7+, Oracle Linux 9
- **Memory**: 4GB+ RAM
- **CPU**: 2+ cores
- **Storage**: 20GB+ available space

### Local Requirements
- Python 3.6+
- Ansible 2.12+
- SSH access to target nodes
- Internet connectivity for downloads

### Install Dependencies
```bash
pip install ansible kubernetes
ansible-galaxy collection install kubernetes.core community.kubernetes
```

## 📋 Configuration

### Inventory Setup

#### Development Environment
Edit `inventories/dev/hosts.yaml`:
```yaml
all:
  hosts:
    dev-k3s-master:
      ansible_host: your-server-ip
      ansible_user: ubuntu
      ansible_ssh_private_key_file: /path/to/your/key.pem
```

#### Production Environment
Edit `inventories/production/hosts.yaml`:
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

### Variable Configuration

#### Development Variables (`inventories/dev/group_vars/all.yaml`)
```yaml
# K3s Configuration
ha_enabled: false                    # Single node for dev
k3s_version: v1.27.5+k3s1
datastore_endpoint: ""               # SQLite for single node

# Rancher Configuration
rancher_version: "latest"
rancher_hostname: "your-domain.com"
rancher_bootstrap_password: "YourSecurePassword123!"
rancher_helm_repo: "https://releases.rancher.com/server-charts/stable"

# TLS Configuration
rancher_tls_source: "rancher"        # Options: rancher, letsEncrypt, secret

# Let's Encrypt (if using letsEncrypt)
lets_encrypt_email: "admin@example.com"
lets_encrypt_environment: "staging"  # staging or production

# Features
traefik_enabled: true
```

#### Production Variables (`inventories/production/group_vars/all.yaml`)
```yaml
# K3s HA Configuration
ha_enabled: true
k3s_version: "v1.27.7+k3s1"
k3s_datastore_endpoint: "mysql://user:pass@tcp(mysql.example.com:3306)/k3s_db"

# Rancher Production Settings
rancher_version: "latest"
rancher_hostname: "rancher.yourdomain.com"
rancher_bootstrap_password: "ProductionPassword123!"
rancher_tls_source: "letsEncrypt"

# Let's Encrypt Production
lets_encrypt_email: "admin@yourdomain.com"
lets_encrypt_environment: "production"

# Custom TLS (if using secret)
tls_cert_file: "/path/to/tls.crt"
tls_key_file: "/path/to/tls.key"
tls_ca_file: "/path/to/ca.crt"
```

## 🚀 Installation

### Single Command Deployment
```bash
# Development environment
ansible-playbook -i inventories/dev/hosts.yaml main.yaml

# Production environment
ansible-playbook -i inventories/production/hosts.yaml main.yaml
```

### Step-by-Step Deployment
```bash
# 1. Test connectivity
ansible -i inventories/dev/hosts.yaml all -m ping

# 2. Check configuration (dry-run)
ansible-playbook -i inventories/dev/hosts.yaml main.yaml --check

# 3. Deploy with verbose output
ansible-playbook -i inventories/dev/hosts.yaml main.yaml -v

# 4. Deploy specific roles only
ansible-playbook -i inventories/dev/hosts.yaml main.yaml --tags "k3s"
```

## 🔍 Verification

### K3s Cluster Status
```bash
# On target node
sudo k3s kubectl get nodes
sudo k3s kubectl get pods --all-namespaces
sudo k3s kubectl cluster-info
```

### Rancher Status
```bash
# Check Rancher pods
sudo k3s kubectl get pods -n cattle-system

# Check certificates (if cert-manager used)
sudo k3s kubectl get certificates -A

# Check ingress
sudo k3s kubectl get ingress -A
```

### Access Rancher UI
1. Open browser: `https://your-rancher-hostname`
2. Login with bootstrap password
3. Set new admin password
4. Import existing cluster or create new ones

## 🗑️ Uninstallation

### Complete Cleanup
```bash
# Remove everything
ansible-playbook -i inventories/dev/hosts.yaml uninstall.yaml

# Remove specific components (edit uninstall.yaml to uncomment)
# - Rancher
# - cert-manager  
# - Common tools
```

### Manual Cleanup (if needed)
```bash
# On target nodes
sudo /usr/local/bin/k3s-uninstall.sh
sudo rm -rf /etc/rancher /var/lib/rancher
sudo rm -f /usr/local/bin/{kubectl,helm,k9s}
```

## 🔧 Customization

### TLS Certificate Options

#### 1. Rancher-Generated (Default)
```yaml
rancher_tls_source: "rancher"
# Requires cert-manager
```

#### 2. Let's Encrypt
```yaml
rancher_tls_source: "letsEncrypt"
lets_encrypt_email: "admin@example.com"
lets_encrypt_environment: "production"  # or "staging"
```

#### 3. Custom Certificates
```yaml
rancher_tls_source: "secret"
tls_cert_file: "/path/to/certificate.crt"
tls_key_file: "/path/to/private.key"
tls_ca_file: "/path/to/ca.crt"
```

### Tool Versions
Edit `roles/common_tools/defaults/main.yaml`:
```yaml
common_tools:
  - name: kubectl
    version: "v1.27.5"
  - name: helm
    version: "v3.12.1"
  - name: k9s
    version: "v0.50.13"
```

### K3s Options
```yaml
# High Availability
ha_enabled: true
datastore_endpoint: "mysql://user:pass@tcp(host:3306)/db"

# Disable Traefik (use external ingress)
traefik_enabled: false

# Custom K3s version
k3s_version: "v1.27.7+k3s1"
```

## 🔒 Security Considerations

- Change default passwords immediately
- Use strong TLS certificates in production
- Restrict network access to Rancher UI
- Regular security updates
- Monitor cluster logs
- Implement RBAC policies

## 🐛 Troubleshooting

### Common Issues

#### K3s Installation Fails
```bash
# Check system requirements
free -h
df -h
systemctl status k3s

# Manual installation test
curl -sfL https://get.k3s.io | sh -
```

#### Rancher Not Accessible
```bash
# Check pods status
sudo k3s kubectl get pods -n cattle-system

# Check ingress
sudo k3s kubectl get ingress -n cattle-system

# Check certificates
sudo k3s kubectl get certificates -A
```

#### cert-manager Issues
```bash
# Check cert-manager pods
sudo k3s kubectl get pods -n cert-manager

# Check certificate requests
sudo k3s kubectl get certificaterequests -A

# Check logs
sudo k3s kubectl logs -n cert-manager deployment/cert-manager
```

### Log Locations
- K3s logs: `journalctl -u k3s`
- Rancher logs: `sudo k3s kubectl logs -n cattle-system deployment/rancher`
- cert-manager logs: `sudo k3s kubectl logs -n cert-manager deployment/cert-manager`

## 📚 Additional Resources

- [K3s Documentation](https://docs.k3s.io/)
- [Rancher Documentation](https://rancher.com/docs/)
- [cert-manager Documentation](https://cert-manager.io/docs/)
- [Ansible Documentation](https://docs.ansible.com/)

## 🤝 Contributing

1. Fork the repository
2. Create feature branch
3. Test changes thoroughly
4. Submit pull request

## 📄 License

This project is licensed under the MIT License.