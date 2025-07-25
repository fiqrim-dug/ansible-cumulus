# Ansible NVUE Switch Configuration

This repository automates NVIDIA NVUE switch configuration using Ansible. It supports both **leaf** and spine switches, with the following key features:

- Modular roles per function (interface, breakout etc)
- Single atomic revision per playbook run
- Per-region and per-host variable support (e.g., DNS, DHCP)
- Secure credential handling using Ansible Vault
- Pre-configuration backup

---

## 🔄 End-to-End Workflow

```plaintext
┌─────────────┐
│ Inventory   │
│ - Hosts     │
│ - Region    │
│ - Vault     │
└────┬────────┘
     │
     ▼
┌─────────────┐
│ Pre-Backup  │
│ - Get config│
│ - Save YAML │
└────┬────────┘
     │
     ▼
┌─────-──────────┐
│ Create Revision│
└───-┬─-─────────┘
     │
     ▼
┌───-─────────────────┐
│ Role Execution      │
│ - Interfaces        │
│ - SNMP, TACACS      │
│ - Description/Host  │
│ All under same revid│
└────┬────────────────┘
     │
     ▼
┌──-──────────────────┐
│ Post-Check          │
│ - Config diff (TBD) │
│ - Validate          │
└────┬────────────────┘
     │
     ▼
┌──────────────┐
│ Apply Config │
│ - Apply revid│
│ - Wait       │
└──────────────┘
```

---

## 🔧 Directory Structure

```bash
.
├── backups
│   └── kgtg-slf-1-6_pre_config.yml
├── group_vars
│   ├── kl.yml
│   ├── leaf.yml
│   └── spine.yml
├── host_vars
│   └── kgtg-slf-1-6.yml
├── inventory.yml
├── playbook.yml
├── README.md
├── requirements.yml
└── roles
    ├── base_config
    │   └── tasks
    │       └── main.yml
    ├── nvue_config
    │   ├── defaults
    │   │   └── main.yml
    │   ├── tasks
    │   │   └── main.yml
    │   └── vars
    │       └── main.yml
    └── nvue_revision
        └── tasks
            ├── post.yml
            └── pre.yml

```

---

## 📦 Roles Overview

| Role            | Purpose                           |
| --------------- | --------------------------------- |
| `nvue_config`   | Set interface breakout and bridge |
| `base_config`   | Configure dhcp-relay, hostname    |
| `nvue_revision` | Create, apply revision, and diff  |

Each role modifies the switch via `nvidia.nvue.*` modules and attaches to a common revision ID.

---

## 🔐 Secrets Handling (Vault)

Each switch can have its own login credentials encrypted using [Ansible Vault](https://docs.ansible.com/ansible/latest/vault_guide/index.html).

To create or edit a host's credentials:

```bash
ansible-vault edit host_vars/kgtg-slf-1-6.yml
```

To run the playbook:

```bash
ansible-playbook playbook.yml  -i inventory.yml  --ask-vault-pass
```

---

## 🌍 Regional Variables

Each region has its own group vars file (e.g. `group_vars/kl.yml`) defining DNS and DHCP:

```yaml
dns01: 172.18.255.101
dns02: 172.18.255.102
dhcp_server: 172.18.244.48
```

---

<!-- ## ▶️ How to Run -->
<!---->
<!-- 1. Backup current config: -->
<!---->
<!-- ```bash -->
<!-- ansible-playbook playbooks/pre.yml -->
<!-- ``` -->
<!---->
<!-- 2. Apply configuration (all roles under one revision): -->
<!---->
<!-- ```bash -->
<!-- ansible-playbook playbooks/site.yml -->
<!-- ``` -->
<!---->
<!-- 3. Apply the revision and show changes: -->
<!---->
<!-- ```bash -->
<!-- ansible-playbook playbooks/post.yml -->
<!-- ``` -->
<!---->
<!-- --- -->

## 📁 Backups

- Config backups are stored in `backups/` per host

---

## ✅ Requirements

- Python 3.8+
- Ansible 2.14+
- NVIDIA NVUE-supported switches
- Collections:

  ```bash
  ansible-galaxy collection install nvidia.nvue
  ```

---

## 🔄 Example Inventory (`inventory/hosts.yml`)

```yaml
all:
  children:
    kl:
      children:
        leaf:
          hosts:
            kgtg-slf-1-6:
            kgtg-slf-1-7:
          vars:
            ansible_network_os: nvidia.nvue.nvue
        spine:
          hosts:
            kgtg-ssp-1-7:
            kgtg-ssp-1-8:
          vars:
            ansible_network_os: nvidia.nvue.nvue
```

---

## 🧩 Future Improvements

- Add config validation tasks
- Role for TACACS and SNMP
- Rearrange inventory file
- Calculate how many port being splitted when applied the breakout value
- Separate playbook for backup config, apply config
- Integrate CI for linting and test runs
