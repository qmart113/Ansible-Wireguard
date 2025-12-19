# Ansible WireGuard VPN Setup

This repository contains an Ansible playbook and role for automating the setup of a **WireGuard VPN server** and multiple **client configurations**. It installs WireGuard, generates all server and client keys, builds the `wg0.conf` server config, creates 10 client configs, configures routing and firewall/NAT rules, and enables the VPN interface to start on boot.

The result is a fully automated, repeatable, and secure WireGuard deployment with a single command.

---

## Prerequisites

- Linode server or dedicated server  
- Ansible installed on your control node (e.g., Ubuntu VM)  
- Target Ubuntu/Debian-based system for the WireGuard server  
- SSH key-based access from control node → server  
- Root or sudo access on the server  

---

## Directory Structure

```text
.
├── inventory.ini
├── playbook.yml
└── roles/
    └── wireguard/
        ├── defaults/
    │ └── main.yml
    ├── handlers/
    │ └── main.yml
    ├── tasks/
    │ └── main.yml
    └── templates/
        ├── wg0.conf.j2
        └── client.conf.j2

```

---

## File Descriptions

- **inventory.ini** – Lists your WireGuard server host  
- **playbook.yml** – Main playbook  
- **roles/wireguard/defaults/main.yml** – All configuration variables  
- **roles/wireguard/tasks/main.yml** – Installs WireGuard, generates keys, writes configs  
- **roles/wireguard/templates/** – Jinja2 templates that generate actual `.conf` files  

---

## Configuration

### Inventory File (`inventory.ini`)

Edit inventory.ini and set your WireGuard server’s IP/DNS and SSH user:
```
[wireguard]
wgserver ansible_host=YOUR_SERVER_IP ansible_user=root ansible_ssh_private_key_file=~/.ssh/id_ed25519
```

- ansible_host – Public IP or DNS of your WireGuard server (e.g., Linode)

- ansible_user – SSH user (often root on fresh cloud servers)

- ansible_ssh_private_key_file – Path to the private key used for authentication

---

## WireGuard variables

Key variables are defined in roles/wireguard/defaults/main.yml. Examples:

```
wireguard_network: "10.10.10.0/24"   # VPN subnet
wireguard_interface: "wg0"           # WireGuard interface name
wireguard_port: 51820                # UDP port WireGuard listens on
wireguard_client_count: 10           # Number of client configs to generate
wireguard_dns: "1.1.1.1"             # DNS pushed to clients
wireguard_allowed_ips: "0.0.0.0/0, ::/0"  # Full tunnel by default
wireguard_persistent_keepalive: 25   # Keepalive for roaming clients

# Optional: External interface and endpoint
external_interface: ""               # If blank, auto-detected (e.g., eth0)
wireguard_endpoint: ""               # If blank, uses ansible_host
```
---

## Usage

Run the playbook:

``` 
ansible-playbook -i inventory.ini playbook.yml

```

This will:

- Install WireGuard  
- Enable IPv4 forwarding  
- Detect external interface  
- Generate server keys  
- Generate 10 client keypairs + preshared keys  
- Render /etc/wireguard/wg0.conf on the server  
- Render  /etc/wireguard/clients/client1.conf … client10.con 
- Enable and start `wg-quick@wg0`  
- Show WireGuard status  

---

## What the Playbook Does (Details)

- Installs required packages:

- wireguard, wireguard-tools, iproute2, iptables, qrencode

- Enables IPv4 forwarding via sysctl

- Auto-detects the outbound network interface (if external_interface is not set)

- Generates and stores:

- Server keypair under /etc/wireguard/

- Client keys + preshared keys under /etc/wireguard/clients/

- Builds a dynamic list of clients with:

- Unique IPs in the VPN subnet (e.g., 10.10.10.2/32 … 10.10.10.11/32)

- Matching keys and preshared keys

- Renders wg0.conf with one [Peer] entry per client

- Renders each client .conf file ready to import into a WireGuard client app

- Applies NAT (MASQUERADE) rules for internet access through the server

- Enables and starts wg-quick@wg0 using systemd

- Prints wg show for status after configuration

  ---

  ## Post-Installation

After running the playbook:

The server config will be at:

```
/etc/wireguard/wg0.conf
```

- Client configs will be at:

```
/etc/wireguard/clients/client1.conf
/etc/wireguard/clients/client2.conf
...
/etc/wireguard/clients/client10.conf
```

You can copy a client config down to your local machine with scp, for example:
```
scp root@YOUR_SERVER_IP:/etc/wireguard/clients/client1.conf ~/client1.conf
```

Then import client1.conf into the WireGuard client on Windows, macOS, Linux, iOS, or Android.

---

Notes

- This playbook assumes a Debian/Ubuntu-based WireGuard server (e.g., Ubuntu on Linode).
.
- wireguard_client_count can be increased or decreased to change how many client configs are generated.

- The playbook is designed around a single central hub (wg0 on one server) and multiple clients.

---

## Troubleshooting

If you run into issues:

- Check Ansible output for failed tasks or error messages.

- Verify WireGuard status on the server:
 ```
sudo wg show
sudo systemctl status wg-quick@wg0
```

- Check that IP forwarding and NAT are working:

- Confirm net.ipv4.ip_forward=1

- Inspect iptables rules for the MASQUERADE rule on the external interface

- Make sure the UDP port (wireguard_port, default 51820) is allowed in any cloud firewall or security group.

- Confirm client devices use the correct config and that their system clocks are reasonably in sync.




