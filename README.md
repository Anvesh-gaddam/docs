---

#  Unified Certificate Deployment POC

### Ansible Demo for PEM / JKS / KDB Certificate Renewal

### Designed for Shield Certificate Automation (Wipro → Verizon)

This repository contains a fully working Proof-of-Concept (POC) demonstrating **metadata-driven SSL/TLS certificate deployment** using **Ansible**.
It is designed to simulate a real enterprise workflow where:

* Jenkins triggers the playbook
* Metadata JSON drives all variables
* Dynamic inventory determines target hosts
* PEM, JKS, and KDB keystores can be updated
* Services can be stopped, backed up, and restarted

This POC is safe, self-contained, and runs entirely inside a GitHub Codespace or local Linux environment.

---

#  Project Structure

```
ansible-poc/
│
├── playbooks/
│     └── cert_renew.yml          # Main certificate deployment playbook
│
├── inventory/
│     └── dynamic_inventory.ini   # Demo inventory (localhost)
│
├── metadata/
│     └── new_cert_metadata.json  # Metadata passed by Jenkins
│
└── incoming/
      ├── demo_cert.crt           # Demo certificate (PEM)
      └── demo_cert.key           # Demo private key (PEM)
```

---

#  What This POC Demonstrates

##  Metadata-Driven Execution

Jenkins (or any CI pipeline) can run:

```bash
ansible-playbook playbooks/cert_renew.yml \
  -i inventory/dynamic_inventory.ini \
  --extra-vars "@metadata/new_cert_metadata.json"
```

The JSON file controls:

* Certificate label
* Certificate type (PEM / JKS / KDB)
* Destination directories
* Service name
* Keystore filenames
* Passwords

---

##  Multi-Format Certificate Handling

| Format        | Action Performed                |
| ------------- | ------------------------------- |
| **PEM**       | Copies `.crt` + `.key` files    |
| **JKS**       | Simulates `keytool -importcert` |
| **KDB**       | Simulates `runmqckm -cert -add` |
| **All Types** | Backup + summary                |

---

##  Backup, Deployment & Restart Flow

The playbook performs:

1. Read metadata
2. Stop the application service (simulated)
3. Create timestamped backup directory
4. Backup existing cert/keystore (simulated)
5. Deploy new certificate files
6. Update keystores depending on type
7. Restart service (simulated)
8. Summary output

This mirrors real enterprise certificate automation without touching production systems.

---

#  Metadata Example (`new_cert_metadata.json`)

```json
{
  "cert_label": "demo_cert",
  "cert_type": "PEM",
  "service_name": "none",

  "cert_dest_dir": "/etc/pki/tls/certs",
  "key_dest_dir": "/etc/pki/tls/private",
  "cert_dest_file": "/etc/pki/tls/certs/demo_cert.crt",

  "jks_file": "demo.jks",
  "kdb_file": "demo.kdb",
  "keystore_password": "changeit"
}
```

Change `"cert_type"` to `"JKS"` or `"KDB"` to test different flows.

---

#  Playbook Overview (`cert_renew.yml`)

### High-Level Steps:

* Stop service (simulated)
* Create backup
* Backup old cert/keystore (simulated)
* For PEM → copy `.crt` and `.key`
* For JKS → simulate `keytool`
* For KDB → simulate `runmqckm`
* Restart service (simulated)
* Output final summary

The playbook is idempotent, modular, and production-safe.

---

#  How to Run the POC (Inside Codespace or Linux)

### 1. Install Ansible

```bash
sudo apt update
sudo apt install ansible -y
```

### 2. Ensure demo cert files exist

```bash
ls ansible-poc/incoming
```

You should see:

```
demo_cert.crt
demo_cert.key
```

### 3. Create destination directories

```bash
sudo mkdir -p /etc/pki/tls/certs
sudo mkdir -p /etc/pki/tls/private
```

### 4. Run the playbook

```bash
ansible-playbook \
  playbooks/cert_renew.yml \
  -i inventory/dynamic_inventory.ini \
  --extra-vars "@metadata/new_cert_metadata.json"
```

---

#  Expected Output (Success)

```
TASK [Copy certificate] changed: [localhost]
TASK [Copy private key] changed: [localhost]
TASK [Final Summary]
ok: [localhost] => {
    "msg": [
        "Certificate deployment completed.",
        "Label: demo_cert",
        "Type: PEM",
        "Service restarted: none"
    ]
}
```

This proves the POC works end-to-end.

---

#  Future Extensions (Production Ready)

You can extend this POC to include:

* Real `keytool` integrations
* Real `runmqckm` commands
* Windows server certificate deployment
* Environment-based dynamic inventory generation
* Automatic certificate validation (`openssl x509`)
* Jenkins Pipelinefile integrations
* Slack/Email notifications

This structure matches large-scale enterprise certificate automation projects.

---

#  Credits

This POC is part of the **Shield Certificate Automation** initiative, demonstrating how Wipro can automate enterprise-wide SSL/TLS lifecycle workflows using Ansible + Jenkins + Python metadata ingestion.


