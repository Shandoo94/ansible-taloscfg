# Overview
This role automates generation of Talos machine configuration bundles and per-node specs.
It decrypts secrets with SOPS, renders control plane or worker configs via talosctl, and drops host-scoped outputs into a local cache.
