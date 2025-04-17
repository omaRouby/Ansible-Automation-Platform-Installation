Here's a structured documentation of your Ansible Automation Platform 2.4 installation steps with prerequisites:

---

### **Ansible Automation Platform 2.4 Installation Guide**
**Architecture:**
- Control Node (VM1)
- Automation Hub (VM2)
- Execution Node (VM3)

---

### **Prerequisites**

1. **System Requirements**:
   - All VMs must have **RHEL 8/9** (subscription active)
   - Minimum specs per node:
     - 4 vCPU
     - 8 GB RAM
     - 40 GB Disk
   - [Official Requirements](https://access.redhat.com/articles/7003807)

2. **Network Configuration**:
   - Confirm SSH connectivity between nodes:
     ```bash
     ssh controlnode
     ssh automationhub
     ssh executionnode
     ```
   - Ensure `/etc/hosts` matches your setup on all nodes:
     ```
     192.168.100.139  controlnode
     192.168.100.100  automationhub
     192.168.100.114  executionnode
     ```

3. **Software Requirements**:
   - Podman (required for Automation Hub)
   - Python 3.8+
   - `tar` and `unzip` utilities

---

### **Step 1: Prepare All Nodes**
1. Register systems with Red Hat Subscription Manager:
   ```bash
   sudo subscription-manager register --username=<RH_USER> --password=<RH_PASSWORD>
   sudo subscription-manager attach --auto
   ```

2. Enable required repositories:
   ```bash
   sudo subscription-manager repos --enable ansible-automation-platform-2.4-for-rhel-9-x86_64-rpms
   ```

3. Install base packages:
   ```bash
   sudo dnf install -y podman python3 python3-pip
   ```

---

### **Step 2: Install Control Node (VM1)**
1. Transfer bundle to controlnode:
   ```bash
   scp ansible-automation-platform-2.4-setup-bundle.tar.gz controlnode:
   ```

2. Extract the bundle:
   ```bash
   tar xvf ansible-automation-platform-2.4-setup-bundle.tar.gz
   cd ansible-automation-platform-2.4-setup-bundle
   ```

3. Configure inventory file (`inventory`):
   ```ini
   [automationcontroller]
   controlnode

   [automationhub]
   automationhub

   [execution_nodes]
   executionnode

   [all:vars]
   admin_password='<YOUR_STRONG_PASSWORD>'
   pg_host='controlnode'
   pg_port=5432
   pg_database='awx'
   pg_username='awx'
   pg_password='<DB_PASSWORD>'
   ```

4. Run installer:
   ```bash
   sudo ./setup.sh -i inventory
   ```

---

### **Step 3: Configure Automation Hub (VM2)**
1. Verify Podman is running:
   ```bash
   systemctl enable --now podman
   ```

2. Access web UI post-installation:
   ```
   https://automationhub:443
   ```

---

### **Step 4: Set Up Execution Node (VM3)**
1. From Control Node web UI:
   - Navigate to **Instance Groups** → **Add execution node**
   - Use SSH credentials for `executionnode`

---

### **Post-Installation Steps**
1. Verify installation:
   ```bash
   curl -k https://controlnode/api/v2/ping/  # Should return {"ha":true,"version":"2.4.0"}
   ```

2. Access interfaces:
   - Control Node: `https://controlnode`
   - Automation Hub: `https://automationhub`

---

### **Important Links**
1. [Official Installation Guide](https://access.redhat.com/documentation/en-us/ansible_automation_platform/2.4/html/red_hat_ansible_automation_platform_installation_guide/index)
2. [Troubleshooting Guide](https://access.redhat.com/articles/7004767)

---

### **Troubleshooting Tips**
1. If nodes can't communicate:
   ```bash
   sudo firewall-cmd --permanent --add-service={http,https,ssh}
   sudo firewall-cmd --reload
   ```

2. Check service status:
   ```bash
   sudo systemctl status automation-controller
   sudo podman ps  # On Automation Hub
   ```

Let me know if you need clarification on any step or encounter issues during installation!
