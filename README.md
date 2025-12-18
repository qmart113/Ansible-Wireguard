# Ansible WireGuard VPN Setup

This repository contains an Ansible playbook and role for automating the setup of a **WireGuard VPN server** and multiple **client configurations**. It installs WireGuard, generates all server and client keys, builds the `wg0.conf` server config, creates 10 client configs, configures routing and firewall/NAT rules, and enables the VPN interface to start on boot.

The result is a fully automated, repeatable, and secure WireGuard deployment with a single command.

---

## Prerequisites

- Linode server or dedicated server
- Ansible installed on your **control node** (e.g., Ubuntu VM)
- Target **Ubuntu/Debian-based** system for the WireGuard server (e.g., Linode)
- SSH key-based access from control node → target server
- Sudo/root privileges on the target server

---

## Directory Structure

```text
.
├── inventory.ini
├── playbook.yml
└── roles/
    └── wireguard/
        ├── defaults/
        │   └── main.yml
        ├── handlers/
        │   └── main.yml
        ├── tasks/
        │   └── main.yml
        └── templates/
            ├── wg0.conf.j2
            └── client.conf.j2


---

##

inventory.ini – Ansible inventory file defining the WireGuard server host

playbook.yml – Top-level Ansible playbook that runs the wireguard role

roles/wireguard/defaults/main.yml – Default variables (VPN subnet, client count, port, DNS, etc.)

roles/wireguard/handlers/main.yml – Handler to restart wg-quick@wg0 when config changes

roles/wireguard/tasks/main.yml – Main automation logic (packages, keys, configs, routing, service)

roles/wireguard/templates/wg0.conf.j2 – Jinja2 template for the server WireGuard config

roles/wireguard/templates/client.conf.j2 – Jinja2 template for the client WireGuard configs
