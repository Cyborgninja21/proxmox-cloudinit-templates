# Proxmox Cloud-Init VM Template Automation

This repository automates the creation of Proxmox VM templates using cloud-init-enabled images for various Linux distributions like Ubuntu, Alpine, and Oracle Linux.

Each template is built with parameters defined in Ansible variable files and named using the convention `<os_name>-<os_version>`, such as `ubuntu-24.04` or `alpine-3.20`.

---

## 🧰 Features

- Create Proxmox VM templates with cloud-init support.
- Download official cloud images automatically.
- Define separate configurations for different distributions.
- Use versioned naming (e.g., `ubuntu-24.04`, `oracle-9`).
- Easily extendable for other Linux distributions.
- Fully automated with Ansible.

---

## 📦 Supported Distributions

| Distribution | Version  | Source URL |
|--------------|----------|------------|
| Ubuntu       | 24.04    | cloud-images.ubuntu.com |
| Alpine       | 3.20     | dl-cdn.alpinelinux.org |
| Oracle Linux | 9        | yum.oracle.com |

Add more by creating a new `.yml` file in `vars/`.

---

## 📁 Directory Structure

```
proxmox_cloudinit_templates/
├── ansible.cfg
├── inventory/
│   └── hosts
├── playbooks/
│   └── create_template.yml
├── roles/
│   └── vm_template/
│       ├── defaults/
│       │   └── main.yml
│       ├── tasks/
│       │   └── main.yml
├── vars/
│   ├── ubuntu-24.04.yml
│   ├── alpine-3.20.yml
│   └── oracle-9.yml
└── README.md
```

---

## ⚙️ Prerequisites

- ✅ Proxmox VE (tested with 7.x+)
- ✅ SSH access to Proxmox node (configured in inventory)
- ✅ Ansible (2.9+ recommended)
- ✅ Git + GitHub CLI (`gh`) for repo setup

Ensure the Proxmox CLI (`qm`) is available and cloud-init is enabled on your nodes.

---

## 🚀 Quick Start

### 1. Clone the Repository

```bash
git clone https://github.com/<your-gh-username>/proxmox_cloudinit_templates.git
cd proxmox_cloudinit_templates
```

### 2. Customize the Proxmox Host Inventory

Edit the `inventory/hosts` file:

```ini
[proxmox]
proxmox-host ansible_host=192.168.1.100 ansible_user=root
```

### 3. Create a Template for a Specific OS

Choose one of the predefined variable files:

```bash
ansible-playbook -i inventory/hosts playbooks/create_template.yml -e "@vars/ubuntu-24.04.yml"
ansible-playbook -i inventory/hosts playbooks/create_template.yml -e "@vars/alpine-3.20.yml"
ansible-playbook -i inventory/hosts playbooks/create_template.yml -e "@vars/oracle-9.yml"
```

---

## 📄 Example: Variable File (`vars/ubuntu-24.04.yml`)

```yaml
vm_id: 9001
os_name: ubuntu
os_version: "24.04"
image_url: https://cloud-images.ubuntu.com/minimal/releases/24.04/release/ubuntu-24.04-minimal-cloudimg-amd64.img
image_filename: ubuntu-24.04-minimal-cloudimg-amd64.img
memory: 2048
cores: 2
disk_size: 10G
```

This file drives the creation of the template using your selected distribution.

---

## 🧪 Testing the Templates

After running the playbook, you can list the templates in Proxmox with:

```bash
qm list
```

Each VM will be marked as a template and can be cloned as needed.

---

## 🔁 Extend to Other Distributions

To add a new OS version or distribution:

1. Copy one of the existing files in `vars/`.
2. Update the VM ID, OS name/version, and image URL.
3. Run the playbook using your new `.yml` file.

---

## 🔐 Security Tips

- Avoid reusing `vm_id` values — ensure they are unique.
- Prefer minimal cloud images from official sources.
- Only run Ansible playbooks from trusted machines or CI pipelines.

---

## 🛠 GitHub Repo Initialization

To initialize and push this as a public GitHub repository using the GitHub CLI:

```bash
git init
gh repo create proxmox_cloudinit_templates --public --source=. --remote=origin --push
```

---

## 📝 License

This project is licensed under the MIT License.

---

## 🙋 Support

If you have issues using this template builder, feel free to open an issue or PR in the GitHub repository.

