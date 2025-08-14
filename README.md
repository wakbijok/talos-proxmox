# Talos Kubernetes Cluster on Proxmox

Automated deployment of a production-ready Talos Kubernetes cluster on Proxmox using Ansible.

## Features

- ✅ **Fully Automated**: Complete cluster deployment from VM creation to ready state
- ✅ **High Availability**: 3-node control plane with Virtual IP (VIP)
- ✅ **Static IP Configuration**: Custom network setup with static IPs
- ✅ **Calico CNI**: Advanced networking with network policies
- ✅ **Control Plane Scheduling**: Enabled for small cluster efficiency
- ✅ **Malaysian Localization**: Uses SIRIM NTP servers
- ✅ **Custom Talos Images**: Factory-generated images with embedded config

## Prerequisites

### Infrastructure

- Proxmox VE cluster with API access
- Available VM IDs (default: 203, 204, 205)
- Network segment with available static IPs
- Storage datastore for VM disks

### Software

- Ansible with required collections:
  ```bash
  ansible-galaxy collection install community.general
  ```
- `talosctl` CLI tool
- `kubectl` CLI tool

### Dependencies

Install required Python packages for Ansible:

```bash
pipx inject ansible proxmoxer requests
```

## Quick Start

1. **Clone and configure:**

   ```bash
   git clone https://github.com/wakbijok/talos-proxmox.git
   cd talos-proxmox
   ```
2. **Update configuration:**
   Edit `group_vars/all.yml` with your environment settings:

   - Proxmox API details
   - Network configuration (IPs, gateway, DNS)
   - Node specifications (VM IDs, resources)
3. **Set Proxmox credentials:**

   ```bash
   ansible-vault edit group_vars/vault.yml
   ```

   Add: `vault_proxmox_api_password: "your_password"`
4. **Deploy cluster:**

   ```bash
   ansible-playbook main.yaml --ask-vault-pass
   ```
5. **Access cluster:**

   ```bash
   export KUBECONFIG=./talos-config/kubeconfig-vip
   kubectl get nodes
   ```

## Configuration

### Node Configuration (group_vars/all.yml)

To change node IPs, VM IDs, or resources, modify the `nodes` section:

```yaml
nodes:
  - name: "talos-cp1"
    role: "controlplane"
    vm_id: "203"              # Change VM ID
    memory: "4096"            # Change RAM (MB)
    cores: "2"                # Change CPU cores
    ip: "192.168.0.201"       # Change IP address
  - name: "talos-cp2"
    role: "controlplane"
    vm_id: "204"
    memory: "4096"
    cores: "2"
    ip: "192.168.0.202"       # Change IP address
  - name: "talos-cp3"
    role: "controlplane"
    vm_id: "205"
    memory: "4096"
    cores: "2"
    ip: "192.168.0.203"       # Change IP address
```

### Network Configuration

```yaml
# Network Configuration
controlplane_vip: "192.168.0.200"  # Virtual IP for cluster access
gateway: "192.168.0.1"              # Network gateway
dns_server: "192.168.0.2"           # DNS server
```

### Proxmox Configuration

```yaml
# Proxmox Configuration
proxmox_api_host: "192.168.0.21"    # Proxmox API endpoint
proxmox_node: "pve-bijok01"          # Proxmox node name
storage: "datastore01"               # Storage for VM disks
```

### Cluster Configuration

```yaml
# Cluster Configuration
cluster_id: "1"
cluster_secret: "yoursecrethere"     # Change for production!
talos_version: "v1.5.5"             # Talos version
```

## File Structure

```
├── main.yaml                    # Main playbook
├── group_vars/
│   ├── all.yml                 # Main configuration variables
│   └── vault.yml               # Encrypted credentials
├── templates/
│   ├── controlplane-patch.yaml.j2  # Machine-specific config
│   └── talos-schematic.yaml.j2     # Talos Factory schematic
├── talos-config/               # Generated after deployment
│   ├── kubeconfig-vip         # Production kubeconfig (uses VIP)
│   ├── kubeconfig             # Debug kubeconfig (direct node)
│   ├── talosconfig            # Talos CLI config
│   └── patches/               # Generated machine configs
└── README.md                   # This file
```

## Usage Examples

### Change Node IPs

Edit `group_vars/all.yml` and modify the `ip` field for each node:

```yaml
nodes:
  - name: "talos-cp1"
    ip: "10.0.1.10"    # New IP
```

### Change VM Resources

Modify `memory` and `cores` in `group_vars/all.yml`:

```yaml
nodes:
  - name: "talos-cp1"
    memory: "8192"      # 8GB RAM
    cores: "4"          # 4 CPU cores
```

### Change VM IDs

Update `vm_id` in `group_vars/all.yml`:

```yaml
nodes:
  - name: "talos-cp1"
    vm_id: "100"        # New VM ID
```

### Access Cluster

```bash
# Production access (via VIP)
export KUBECONFIG=./talos-config/kubeconfig-vip
kubectl get nodes

# Direct node access (for debugging)
export KUBECONFIG=./talos-config/kubeconfig
kubectl get nodes

# Talos management
export TALOSCONFIG=./talos-config/talosconfig
talosctl --endpoints 192.168.0.200 get members
```

## Troubleshooting

### Common Issues

1. **VM already exists:**

   - Playbook automatically stops and removes existing VMs
   - Check VM IDs don't conflict with other VMs
2. **Network connectivity:**

   - Verify static IP range doesn't conflict with DHCP
   - Ensure gateway and DNS settings are correct
3. **Etcd leader changes:**

   - Normal during CNI initialization
   - Playbook includes retry logic
4. **Proxmox API errors:**

   - Check credentials in vault.yml
   - Verify Proxmox node name and storage exist

### Logs and Debugging

```bash
# Check Talos node status
talosctl --endpoints 192.168.0.201 get members

# Check Kubernetes status
kubectl get pods -n kube-system

# Check Calico status
kubectl get pods -n kube-system -l k8s-app=calico-node
```

## Production Considerations

1. **Change default secrets:**

   - Update `cluster_secret` in all.yml
   - Use strong Proxmox passwords in vault.yml
2. **Resource planning:**

   - Minimum 4GB RAM per control plane node
   - Ensure adequate storage space
3. **Network security:**

   - Configure firewall rules for Kubernetes ports
   - Secure Proxmox API access
4. **Backup:**

   - Regular etcd backups via Talos
   - Save generated certificates and configs

## Architecture

```
┌─────────────────┐    ┌─────────────────┐    ┌─────────────────┐
│   talos-cp1     │    │   talos-cp2     │    │   talos-cp3     │
│ 192.168.0.201   │    │ 192.168.0.202   │    │ 192.168.0.203   │
│                 │    │                 │    │                 │
│ Control Plane   │    │ Control Plane   │    │ Control Plane   │
│ + Worker        │    │ + Worker        │    │ + Worker        │
└─────────────────┘    └─────────────────┘    └─────────────────┘
         │                       │                       │
         └───────────────────────┼───────────────────────┘
                                 │
                    ┌─────────────────┐
                    │      VIP        │
                    │ 192.168.0.200   │
                    │                 │
                    │ Load Balanced   │
                    │ API Access      │
                    └─────────────────┘
```

## Contributing

1. Test changes in a lab environment
2. Update documentation for new features
3. Follow Ansible best practices
4. Ensure compatibility with latest Talos versions

## License

MIT License - see LICENSE file for details.
