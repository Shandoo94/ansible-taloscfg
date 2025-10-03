# Overview
This role automates generation of Talos machine configuration bundles and per-node specs.
It decrypts secrets with SOPS, renders control plane or worker configs via talosctl, and drops host-scoped outputs into a local cache.

# Requirements
- 'talosctl' available on the Ansible control host.
- SOPS-encrypted secrets file accessible at 'taloscfg_secrets_file'.
- 'community.sops' collection for the decrypt filter.
- 'ansible.utils' collection for the 'ipaddr' filter.
