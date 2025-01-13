# Talos Kubernetes Cluster on Proxmox

This playbook automates the deployment of a 3-node Talos Kubernetes cluster on Proxmox VE.

## Prerequisites

- Ansible installed on your control node
- Access to Proxmox VE API
- `talosctl` installed on your control node

## Configuration

1. Update `group_vars/all.yml` with your Proxmox credentials:
```yaml
proxmox_api_host: "proxmox.local"
proxmox_api_user: "root@pam"
proxmox_api_password: "your_password"
cluster_name: "homelab"
```

2. Network Configuration:
- Control Plane VIP: 192.168.0.200
- Node 1: 192.168.0.201
- Node 2: 192.168.0.202
- Node 3: 192.168.0.203
- Gateway: 192.168.0.1
- DNS Server: 192.168.0.2

## Usage

1. Create your inventory file:
```ini
[proxmox_host]
proxmox.local ansible_user=root
```

2. Run the playbook:
```bash
ansible-playbook -i inventory main.yaml
```

3. Bootstrap the cluster:
```bash
talosctl --nodes 192.168.0.201 bootstrap
```

## Files Structure

```
talos-proxmox/
├── README.md
├── main.yaml
├── group_vars/
│   └── all.yml
└── templates/
    └── controlplane-patch.yaml.j2
```