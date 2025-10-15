# Overview
This role automates generation of Talos machine configuration bundles and per-node specs.
It decrypts secrets with SOPS, renders control plane or worker configs via talosctl, and drops host-scoped outputs into a local cache.

# Requirements
- `talosctl` available on the Ansible control host.
- SOPS-encrypted secrets file accessible at `taloscfg_secrets_file`.
- `community.sops` collection for the decrypt filter.
- Heml cli for the installation of Cilium.

# Changing the default CNI

## Cilium
To use Cilium as a CNI the variable `taloscfg_use_cilium_cni` to `true`.
Cilium is then added to the machine configuration as an inline manifest through a patch.
The manifest is generated through the official Cilium Helm chart.
The chart can be configured by adding a list of values under `taloscfg_cilium_helm_args`, in the same manner as using the Helm cli.
Arguments under `taloscfg_cilium_helm_args` do not have prefixed with `--set`.
Generation of the manifest uses the default configuration as shown on the official [Talos website](https://docs.siderolabs.com/kubernetes-guides/cni/deploying-cilium#deploying-cilium-cni).
