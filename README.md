Ansible WireGuard VPN Setup

This repository contains an Ansible playbook and role for automating the setup of a WireGuard VPN server and multiple client configurations. It installs WireGuard, generates all server and client keys, builds the wg0.conf server config, creates 10 client configs, configures routing and firewall/NAT rules, and enables the VPN interface to start on boot.

The result is a fully automated, repeatable, and secure WireGuard deployment with a single command.

Prerequisites

Linode server or dedicated server

Ansible installed on your control node (e.g., Ubuntu VM)

Target Ubuntu/Debian-based system for the WireGuard server (e.g., Linode)

SSH key-based access from control node → target server

Sudo/root privileges on the target server

Directory Structure
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

File Descriptions

inventory.ini – Defines the WireGuard server host

playbook.yml – Top-level Ansible playbook that runs the wireguard role

roles/wireguard/defaults/main.yml – Default variables (VPN subnet, client count, port, etc.)

roles/wireguard/handlers/main.yml – Restarts wg-quick@wg0

roles/wireguard/tasks/main.yml – Automation logic (packages, keys, configs, routing, service)

roles/wireguard/templates/wg0.conf.j2 – Jinja2 server WireGuard config

roles/wireguard/templates/client.conf.j2 – Jinja2 client WireGuard configs

Configuration
1. Inventory

Edit inventory.ini and set your WireGuard server’s IP and SSH user:

[wireguard]
wgserver ansible_host=YOUR_SERVER_IP ansible_user=root ansible_ssh_private_key_file=~/.ssh/id_ed25519


ansible_host = your server IP (e.g., Linode)

ansible_user = SSH user

ansible_ssh_private_key_file = your SSH private key path

2. WireGuard Variables

Key variables are located in:

roles/wireguard/defaults/main.yml


Important options include:

wireguard_network: "10.10.10.0/24"
wireguard_interface: "wg0"
wireguard_port: 51820
wireguard_client_count: 10
wireguard_dns: "1.1.1.1"
wireguard_allowed_ips: "0.0.0.0/0, ::/0"
wireguard_persistent_keepalive: 25
external_interface: ""
wireguard_endpoint: ""

Usage

Run the playbook:

ansible-playbook -i inventory.ini playbook.yml


This will:

Install WireGuard

Enable IPv4 forwarding

Auto-detect outbound interface (if not set)

Generate:

Server private/public key

10 client keypairs

10 preshared keys

Render:

/etc/wireguard/wg0.conf

/etc/wireguard/clients/client1.conf … client10.conf

Enable and start wg-quick@wg0

Show WireGuard runtime status

What the Playbook Does (Detailed)
Installs required packages

wireguard

wireguard-tools

iproute2

iptables

qrencode

Network configuration

Enables IPv4 forwarding

Sets MASQUERADE NAT rule for Internet access

Auto-detects external interface (if not set)

Key generation

Server private/public keys → /etc/wireguard/

Client keys (private/public/psk) → /etc/wireguard/clients/

Builds a dynamic list of clients with VPN IP assignments

Configuration rendering

Renders WireGuard server config

Renders all client configs ready for import

Enables wg-quick auto-start

Verification

Shows wg show output after configuration

Post-Installation

After running the playbook:

Server config:
/etc/wireguard/wg0.conf

Client configs:
/etc/wireguard/clients/client1.conf
/etc/wireguard/clients/client2.conf
...
/etc/wireguard/clients/client10.conf


You can retrieve a config:

scp root@YOUR_SERVER_IP:/etc/wireguard/clients/client1.conf ~/client1.conf


Then import into WireGuard on:

Windows

macOS

Linux

Android

iOS

Troubleshooting

Check WireGuard status:

sudo wg show
sudo systemctl status wg-quick@wg0


Check NAT rules:

sudo iptables -t nat -L -n -v


Check service logs:

sudo journalctl -u wg-quick@wg0

Security Considerations

Treat all private keys and client configs as sensitive

Rotate keys if compromise is suspected

Restrict SSH access

Consider cloud firewall rules for port 51820/UDP
