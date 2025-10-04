# K3s + Rancher Ansible Playbooks

This project provides a modular and production-ready Ansible setup for installing **K3s** (single or HA) and **Rancher** on Linux nodes. Supports **cert-manager**, Traefik ingress, and TLS options. Includes installation and uninstallation playbooks.

---

## Table of Contents

- [Folder Structure](#folder-structure)
- [Prerequisites](#prerequisites)
- [Inventory](#inventory)
- [Roles](#roles)
- [Variables](#variables)
- [Installation](#installation)
- [Uninstallation](#uninstallation)
- [Verification](#verification)
- [Notes](#notes)

---

## Folder Structure
k3s-rancher-ansible/
├── inventories/
│ ├── dev/
│ │ ├── hosts.yaml
│ │ └── group_vars/all.yaml
│ └── production/
│ ├── hosts.yaml
│ └── group_vars/all.yaml
├── roles/
│ ├── k3s_master/
│ │ └── tasks/main.yaml
│ ├── rancher/
│ │ └── tasks/main.yaml
│ └── cert_manager/
│ └── tasks/main.yaml
├── main.yaml
├── uninstall.yaml
└── README.md


---

## Prerequisites

- Linux nodes (Ubuntu / Amazon Linux / CentOS)
- Python 3 and `ansible >= 2.12`
- `kubectl` installed
- SSH access to all nodes

---

## Inventory

### Example: `dev/hosts.yaml`

```yaml
all:
  hosts:
    dev-node1:
      ansible_host: 172.31.60.194
      ansible_user: ubuntu
      ansible_become: true
```
### Example: production/hosts.yaml
```yaml
all:
  hosts:
    prod-master1:
      ansible_host: 10.0.1.10
      ansible_user: ubuntu
      ansible_become: true
    prod-master2:
      ansible_host: 10.0.1.11
      ansible_user: ubuntu
      ansible_become: true
```
Roles
k3s_master

Installs K3s server

HA support with external datastore

Generates kubeconfig

rancher

Installs Rancher via Helm

TLS options:

rancher (requires cert-manager)

letsEncrypt (requires cert-manager)

secret (from files, no cert-manager)

cert_manager

Installs cert-manager via Helm

Required for Rancher-generated certificates or Let's Encrypt

# Installation

### Run main playbook:

``` ansible-playbook -i inventories/dev/hosts.yaml main.yaml ```


# Production inventory:

``` ansible-playbook -i inventories/production/hosts.yaml main.yaml ```

# Uninstallation
``` ansible-playbook -i inventories/dev/hosts.yaml uninstall.yaml  ```


Removes:

Rancher Helm release

Cert-manager (if installed)

K3s server

Verification
sudo k3s kubectl get nodes
sudo k3s kubectl get pods --all-namespaces
sudo k3s kubectl get pods -n cattle-system
sudo k3s kubectl get certificates -A  # if cert-manager is used

Notes

K3s kubeconfig: /etc/rancher/k3s/k3s.yaml

HA setup requires external datastore shared among master nodes

TLS options:

rancher → Rancher-generated (requires cert-manager)

letsEncrypt → Automatic Let's Encrypt (requires cert-manager)

secret → Use existing TLS secret

Traefik enabled by default; disable with enable_traefik: false in group_vars



