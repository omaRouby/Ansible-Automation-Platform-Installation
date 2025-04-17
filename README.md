Here’s a **GitHub README-style document** combining the architecture diagram, prerequisites, installation steps, and key details:

---

# Ansible Automation Platform 2.4 Deployment Guide

## Architecture Overview

```
+-------------------+          +-------------------+          +-------------------+
|   Control Node    |          |  Automation Hub   |          |  Execution Node   |
| (VM1: 192.168.100.139) | <-----> | (VM2: 192.168.100.100) | <-----> | (VM3: 192.168.100.114) |
+-------------------+          +-------------------+          +-------------------+
          |                             |                             |
          | SSH/HTTPS                    | HTTPS/API                  | SSH/HTTPS
          v                             v                             v
+-------------------+          +-------------------+          +-------------------+
| - Ansible Control |          | - Private Content |          | - Runs Ansible    |
|   Controller      |          |   Collections     |          |   Jobs            |
| - PostgreSQL DB   |          | - Podman Runtime  |          | - Lightweight     |
| - Web UI (443/TCP)|          | - Galaxy NG       |          |   Execution       |
+-------------------+          +-------------------+          +-------------------+
```

### Communication Flow
- **Control Node** manages playbooks, inventories, and dispatches jobs to **Execution Nodes**.
- **Automation Hub** serves custom Ansible Collections/Execution Environments to the Control Node.
- All nodes resolve each other via `/etc/hosts` or DNS.

---

## Prerequisites

### 1. System Requirements
| Node              | vCPU | RAM  | Disk  | OS           |
|-------------------|------|------|-------|--------------|
| **Control Node**  | 4    | 8 GB | 40 GB | RHEL 8/9     |
| **Automation Hub**| 4    | 8 GB | 40 GB | RHEL 8/9     |
| **Execution Node**| 2    | 4 GB | 20 GB | RHEL 8/9     |

- **All VMs** must have:
  - Active Red Hat subscriptions.
  - SSH connectivity between nodes.
  - `/etc/hosts` configured as:
    ```
    192.168.100.139  controlnode
    192.168.100.100  automationhub
    192.168.100.114  executionnode
    ```

### 2. Software Requirements
- Podman (Automation Hub)
- Python 3.8+
- `ansible-automation-platform-2.4-setup-bundle.tar.gz` ([Download Link](https://access.redhat.com/downloads/content/480/ver=2.4/rhel---9/2.4/x86_64/product-software))

---

## Installation Steps

### 1. Prepare All Nodes
```bash
# Register RHEL subscriptions
sudo subscription-manager register --username=<RH_USER> --password=<RH_PASSWORD>
sudo subscription-manager attach --auto

# Enable AAP 2.4 repository
sudo subscription-manager repos --enable ansible-automation-platform-2.4-for-rhel-9-x86_64-rpms

# Install dependencies
sudo dnf install -y podman python3 python3-pip tar unzip
```

### 2. Configure Control Node (VM1)
```bash
# Extract the AAP bundle
tar xvf ansible-automation-platform-2.4-setup-bundle.tar.gz
cd ansible-automation-platform-2.4-setup-bundle

# Edit inventory file (example snippet)
cat <<EOF > inventory
[automationcontroller]
controlnode

[automationhub]
automationhub

[execution_nodes]
executionnode

[all:vars]
admin_password='<STRONG_PASSWORD>'
pg_host='controlnode'
pg_port=5432
pg_database='awx'
pg_username='awx'
pg_password='<DB_PASSWORD>'
EOF

# Run installer
sudo ./setup.sh -i inventory
```

### 3. Configure Automation Hub (VM2)
```bash
# Start Podman
sudo systemctl enable --now podman

# Verify Automation Hub post-installation
curl -k https://automationhub:443/api/automation-hub/_ui/v1/feature-flags/
```

### 4. Set Up Execution Node (VM3)
- From the Control Node web UI (`https://controlnode`):
  1. Navigate to **Administration → Execution Environments**.
  2. Add the Execution Node using SSH credentials for `executionnode`.

---

## Post-Installation

### Verify Connectivity
```bash
# Check Control Node health
curl -k https://controlnode/api/v2/ping/  # Should return {"ha":true,"version":"2.4.0"}

# Check Automation Hub
curl -k https://automationhub:443/api/automation-hub/v3/_ui/feature-flags/
```

### Firewall Rules
```bash
# Open ports on all nodes
sudo firewall-cmd --permanent --add-service={http,https,ssh}
sudo firewall-cmd --reload
```

---

## References
1. [Official AAP 2.4 Installation Guide](https://access.redhat.com/documentation/en-us/ansible_automation_platform/2.4/html/red_hat_ansible_automation_platform_installation_guide/index)
2. [Troubleshooting AAP](https://access.redhat.com/articles/7004767)
3. [Ansible Automation Hub Docs](https://access.redhat.com/documentation/en-us/ansible_automation_hub/2.4)

---

## Troubleshooting
| Issue                          | Solution                                  |
|--------------------------------|-------------------------------------------|
| Nodes can’t resolve hostnames  | Verify `/etc/hosts` on all VMs.           |
| Automation Hub Podman errors   | Run `sudo systemctl restart podman`.      |
| Jobs stuck in "pending" state  | Check SSH keys between Control/Execution Nodes. |

---

Save this as `README.md` in your project repository. Let me know if you need adjustments! 🚀
