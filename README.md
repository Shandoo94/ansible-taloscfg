# Talos Configuration Generator

An Ansible role for automating the generation of Talos machine configuration bundles and per-node specifications. This role streamlines the process of creating Talos cluster configurations by handling secret decryption, configuration patching, and CNI integration.

## Features

- **Automated Configuration Generation**: Generates both control plane and worker node configurations using `talosctl`
- **SOPS Integration**: Seamlessly decrypts SOPS-encrypted secrets during configuration generation
- **Flexible Patching System**: Supports multiple levels of configuration patches (common, role-specific, and host-specific)
- **CNI Support**: Built-in support for Cilium CNI with Helm chart integration
- **High Availability**: Optional VIP configuration for control plane high availability
- **Secure by Default**: Automatically cleans up decrypted secrets after generation

## Requirements

### System Requirements
- Ansible 2.9 or higher
- `talosctl` binary available on the Ansible control host
- `helm` CLI for Cilium CNI support (optional)

### Ansible Collections
- `community.sops` - Required for SOPS secret decryption
- `ansible.utils` - Required for IP address filtering

## Quick Start

### Basic Usage
```yaml
- hosts: talos_nodes
  roles:
    - role: taloscfg.taloscfg
      vars:
        taloscfg_secrets_file: "{{ playbook_dir }}/secrets/secrets.yaml"
        taloscfg_cluster_name: "my-cluster"
        taloscfg_api_endpoint_cidr: "10.0.0.10/24"
```

### With Cilium CNI
```yaml
- hosts: talos_nodes
  roles:
    - role: taloscfg.taloscfg
      vars:
        taloscfg_secrets_file: "{{ playbook_dir }}/secrets/secrets.yaml"
        taloscfg_cluster_name: "my-cluster"
        taloscfg_api_endpoint_cidr: "10.0.0.10/24"
        taloscfg_use_cilium_cni: true
        taloscfg_cilium_replace_kubeproxy: true
```

## Configuration

### Core Variables

| Variable | Required | Default | Description |
|----------|----------|---------|-------------|
| `taloscfg_secrets_file` | Yes | - | Path to SOPS-encrypted Talos secrets file |
| `taloscfg_cluster_name` | No | `talos-cluster` | Name of the Talos cluster |
| `taloscfg_api_endpoint_cidr` | No | `10.10.10.5/24` | API endpoint address with CIDR |
| `taloscfg_api_endpoint_port` | No | `6443` | TCP port for API endpoint |
| `taloscfg_kubernetes_version` | No | `v1.34.0` | Kubernetes version |
| `taloscfg_talos_version` | No | `v1.11.1` | Talos OS version |
| `taloscfg_target_dir` | No | `{{ playbook_dir }}/.cache` | Output directory for generated configs |

### Inventory Groups

| Variable | Default | Description |
|----------|---------|-------------|
| `taloscfg_server_group` | `servers` | Inventory group for control plane nodes |
| `taloscfg_worker_group` | `workers` | Inventory group for worker nodes |

### Patching System

The role supports multiple levels of configuration patches:

- **Common Patches**: Applied to all nodes
- **Role-specific Patches**: Applied only to control plane or worker nodes
- **Host-specific Patches**: Applied to individual hosts

```yaml
taloscfg_common_patches:
  - "{{ playbook_dir }}/patches/common.yaml"
taloscfg_common_server_patches:
  - "{{ playbook_dir }}/patches/control-plane.yaml"
taloscfg_common_worker_patches:
  - "{{ playbook_dir }}/patches/worker.yaml"
taloscfg_machine_patches:
  - "{{ playbook_dir }}/patches/{{ inventory_hostname }}.yaml"
```

## CNI Configuration

### Cilium CNI

To use Cilium as the CNI, set `taloscfg_use_cilium_cni` to `true`. The role will:

1. Generate Cilium manifests using the official Helm chart
2. Apply them as inline manifests through configuration patches
3. Support kube-proxy replacement mode

#### Cilium Variables

| Variable | Default | Description |
|----------|---------|-------------|
| `taloscfg_use_cilium_cni` | `false` | Enable Cilium CNI |
| `taloscfg_cilium_version` | `1.18.2` | Cilium Helm chart version |
| `taloscfg_cilium_replace_kubeproxy` | `false` | Enable kube-proxy replacement |
| `taloscfg_cilium_helm_args` | `[]` | Additional Helm arguments |

#### Example Cilium Configuration
```yaml
taloscfg_use_cilium_cni: true
taloscfg_cilium_replace_kubeproxy: true
taloscfg_cilium_version: "1.18.2"
taloscfg_cilium_helm_args:
  - "operator.replicas=2"
  - "hubble.relay.enabled=true"
  - "hubble.ui.enabled=true"
```

## High Availability

### Virtual IP Configuration

For control plane high availability, enable VIP mode:

```yaml
taloscfg_api_use_vip: true
taloscfg_vip_interface_name: "eth0"
```

This will configure Talos to manage a virtual IP for the API server endpoint.

## Output

Generated configurations are stored in:
```
{{ taloscfg_target_dir }}/{{ taloscfg_cluster_name }}/
├── {{ inventory_hostname }}.yaml    # Per-host machine config
└── {{ taloscfg_cluster_name }}.yaml # Talos client config
```

## Security

- Secrets are decrypted temporarily and automatically cleaned up
- Generated configuration files have restricted permissions (0600)
- SOPS integration ensures secrets remain encrypted at rest
