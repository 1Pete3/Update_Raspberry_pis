<div align="center">
<h1>Raspberry Pi Automated Patch Management</h1>

  

[![Ansible-lint](https://github.com/1Pete3/Update_Raspberry_Pis/actions/workflows/ansible-lint.yml/badge.svg)](https://github.com/1Pete3/Update_Raspberry_Pis/actions/workflows/ansible-lint.yml)   [![YAML-lint](https://github.com/1Pete3/Update_Raspberry_Pis/actions/workflows/yaml-lint.yml/badge.svg)](https://github.com/1Pete3/Update_Raspberry_Pis/actions/workflows/yaml-lint.yml)


Automated patch management for a Raspberry Pi homelab fleet. Runs daily via cron, updates all nodes, and sends a Discord notification confirming the result — whether packages changed or not.
</div>

---

## What It Does
<p align="center">
<img width="415" height="650" alt="Discord notifcations raspberry pi updates" src="https://github.com/user-attachments/assets/899998c7-eb64-42bb-8192-64195d95d287" />
</p>

- Runs `apt update`, `apt upgrade`, and `autoremove` across all Pis
- Sends a Discord embed notification per host with timestamp and change status
- Notifies on both changed and unchanged runs so you always know it executed
- Targets Debian/Ubuntu based systems only

---

## Stack

- **Ansible** — orchestration and task execution
- **Ansible Vault** — secrets management for the Discord webhook URL
- **Discord Webhooks** — notification delivery
- **Raspberry Pi** — target fleet (Debian/Raspbian)
- **Cron** — daily execution trigger

---

## Project Structure

```
Update_Raspberry_Pis/
├── ansible.cfg            # Ansible configuration (inventory path, SSH key)
├── inventory.example.ini  # Example inventory file — copy to inventory.ini and fill in your IPs
├── group_vars/
│   └── pi.example.yml     # Example vars file — copy to pi.yml and configure
├── run-playbook.example.sh # Example shell script used by cron to run the playbook
└── Update_Pis.yml             # Main playbook — apt update/upgrade with Discord notifications
```

---

## Prerequisites

- Ansible installed on your control node
- SSH key access to all Pi nodes
- A Discord webhook URL
- Ansible Vault password set up

---

## Setup

### 1. Clone the repo

```bash
git clone https://github.com/1Pete3/Update_Raspberry_Pis.git
cd Update_Raspberry_Pis
```

### 2. Configure your inventory

```bash
cp inventory.example.ini inventory.ini
```

Edit `inventory.ini` and replace the placeholder IPs with your actual Pi addresses.

### 3. Configure your variables

```bash
cp group_vars/pi.example.yml group_vars/pi.yml
```

Edit `group_vars/pi.yml` and add your Discord webhook URL. Encrypt it with Ansible Vault:

```bash
ansible-vault encrypt group_vars/pi.yml
```

### 4. Set up the cron job

```bash
cp run-playbook.example.sh run-playbook.sh
```

Edit `run-playbook.sh` and update the paths to match your environment. Then add it to cron:

```bash
crontab -e
```

Example cron entry to run daily at 2am:

```
0 2 * * * /path/to/your/run-playbook.sh >> /path/to/your/run-playbook.log 2>&1
```

---

## Running Manually

```bash
ansible-playbook Update_Pis.yml --vault-password-file vault_file_path
```

---

## Architecture Decisions

**Why notify on both changed and unchanged runs?**
A notification only on changes tells you when something happened — not whether the automation ran at all. Notifying either way gives you confidence the cron job is alive and the playbook executed successfully.

**Why Ansible over a plain bash cron job?**
Ansible handles multiple hosts in parallel, provides idempotent task execution, structured output, and easy extensibility. Adding a new Pi to the fleet is one line in the inventory.

**Why Ansible Vault for the webhook URL?**
The Discord webhook URL is effectively a secret — anyone with it can post to your server. Vault keeps it encrypted at rest in the repo without needing an external secrets manager.

---

## Future Plans

- Email notification support (toggle via `send_email` variable)
- Additional playbooks for service management and configuration drift detection

---
