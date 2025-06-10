# Proxmox Cloud-Init VM Templates

[![Proxmox](https://img.shields.io/badge/Proxmox-VE-orange?style=for-the-badge&logo=proxmox)](https://www.proxmox.com/)
[![Ansible](https://img.shields.io/badge/Ansible-EE0000?style=for-the-badge&logo=ansible&logoColor=white)](https://www.ansible.com/)
[![Cloud-Init](https://img.shields.io/badge/Cloud--Init-326CE5?style=for-the-badge&logo=cloud&logoColor=white)](https://cloud-init.io/)
[![Ubuntu](https://img.shields.io/badge/Ubuntu-E95420?style=for-the-badge&logo=ubuntu&logoColor=white)](https://ubuntu.com/)
[![Alpine Linux](https://img.shields.io/badge/Alpine_Linux-0D597F?style=for-the-badge&logo=alpine-linux&logoColor=white)](https://alpinelinux.org/)
[![Oracle Linux](https://img.shields.io/badge/Oracle_Linux-F80000?style=for-the-badge&logo=oracle&logoColor=white)](https://www.oracle.com/linux/)

An Ansible-based automation solution for creating cloud-init enabled VM templates in Proxmox VE. This project streamlines the process of downloading cloud images and converting them into reusable Proxmox templates that support cloud-init for automated VM provisioning.

## ✨ Features

- **🚀 Automated Template Creation**: Download cloud images and convert them to Proxmox templates
- **☁️ Cloud-Init Support**: All templates are configured with cloud-init for automated provisioning
- **🐧 Multiple OS Support**: Pre-configured for Ubuntu, Alpine Linux, and Oracle Linux
- **🎯 Dynamic VM ID Assignment**: Automatically assigns available VM IDs to avoid conflicts
- **🔧 Flexible Configuration**: Easy to extend for additional operating systems
- **🔄 Idempotent Operations**: Safe to run multiple times without conflicts

## 📋 Supported Operating Systems

| OS | Version | Image Type | Default Resources |
|---|---|---|---|
| Ubuntu | 24.04 LTS | Minimal Cloud Image | 2GB RAM, 2 cores, 10GB disk |
| Alpine Linux | 3.20 | Virtual ISO | 512MB RAM, 1 core, 2GB disk |
| Oracle Linux | 9 | Cloud Image | 2GB RAM, 2 cores, 10GB disk |

## 📦 Prerequisites

- Proxmox VE 7.x or 8.x
- Ansible 2.9 or higher
- SSH access to Proxmox host with root privileges
- Sufficient storage space for downloading and storing images
- Network connectivity to download cloud images

## 🚀 Quick Start

### 1. Clone Repository

```bash
git clone https://github.com/yourusername/proxmox-cloudinit-templates.git
cd proxmox-cloudinit-templates
```

### 2. Configure Inventory

Edit `inventory/hosts` to match your Proxmox environment:

```ini
[proxmox]
proxmox-host ansible_host=YOUR_PROXMOX_IP ansible_user=root
```

### 3. Create Templates

Create Ubuntu 24.04 template:
```bash
ansible-playbook playbooks/create_template.yml -e @vars/ubuntu-24.04.yml
```

Create Alpine Linux template:
```bash
ansible-playbook playbooks/create_template.yml -e @vars/alpine-3.20.yml
```

Create Oracle Linux template:
```bash
ansible-playbook playbooks/create_template.yml -e @vars/oracle-9.yml
```

## 📁 Project Structure

```
proxmox-cloudinit-templates/
├── ansible.cfg                    # Ansible configuration
├── inventory/
│   └── hosts                      # Proxmox host inventory
├── playbooks/
│   └── create_template.yml        # Main template creation playbook
├── roles/
│   └── vm_template/
│       ├── defaults/
│       │   └── main.yml          # Default variables
│       └── tasks/
│           └── main.yml          # Template creation tasks
└── vars/
    ├── ubuntu-24.04.yml          # Ubuntu configuration
    ├── alpine-3.20.yml           # Alpine configuration
    └── oracle-9.yml              # Oracle Linux configuration
```

## ⚙️ Configuration

### Default Settings

The role uses these default settings (can be overridden in var files):

```yaml
os_name: "ubuntu"
os_version: "24.04"
memory: 1024                      # RAM in MB
cores: 1                          # CPU cores
disk_size: 4G                     # Disk size
net_bridge: "vmbr0"              # Network bridge
proxmox_storage_id: "mixedstore"  # Storage pool
```

### Storage Configuration

By default, the playbook uses a storage pool named `mixedstore`. To use a different storage:

1. Edit `roles/vm_template/defaults/main.yml`
2. Change `proxmox_storage_id` to your storage pool name
3. Or override via command line: `-e proxmox_storage_id=local-lvm`

### Custom OS Configuration

To add support for additional operating systems, create a new variable file in `vars/`:

```yaml
# vars/debian-12.yml
os_name: debian
os_version: "12"
image_url: https://cloud.debian.org/images/cloud/bookworm/latest/debian-12-generic-amd64.qcow2
image_filename: debian-12-generic-amd64.qcow2
memory: 2048
cores: 2
disk_size: 10G
```

## 🎯 Template Usage

Once created, templates can be used with:

### Proxmox Web UI
1. Navigate to your node in the Proxmox web interface
2. Right-click on the template (e.g., "ubuntu-24.04")
3. Select "Clone" to create new VMs

### Terraform
```hcl
resource "proxmox_vm_qemu" "example" {
  name        = "my-vm"
  target_node = "proxmox-node"
  clone       = "ubuntu-24.04"
  
  # Cloud-init configuration
  os_type   = "cloud-init"
  ipconfig0 = "ip=dhcp"
  
  # Customize as needed
  memory = 4096
  cores  = 2
}
```

### CLI (qm command)
```bash
# Clone template to create new VM
qm clone <template-id> <new-vm-id> --name my-new-vm
```

## 🔧 Troubleshooting

### Common Issues

**Permission Denied**
- Ensure SSH key authentication is configured for root user
- Verify Proxmox user has sufficient privileges

**Storage Not Found**
- Check if the storage pool exists: `pvesm status`
- Verify storage pool name in configuration matches Proxmox

**Download Failures**
- Verify internet connectivity from Proxmox host
- Check if URLs in var files are accessible
- Some images may require updated URLs

**VM ID Conflicts**
- The script automatically finds available VM IDs starting from 9000
- Manually check with `qm list` if issues persist

### Debugging

Enable verbose output:
```bash
ansible-playbook playbooks/create_template.yml -e @vars/ubuntu-24.04.yml -vvv
```

Check Proxmox logs:
```bash
journalctl -u pvedaemon -f
```

## 🚀 Advanced Usage

### Customizing VM Resources

Override default resources per OS:
```bash
ansible-playbook playbooks/create_template.yml \
  -e @vars/ubuntu-24.04.yml \
  -e memory=4096 \
  -e cores=4 \
  -e disk_size=20G
```

### Batch Template Creation

Create all templates at once:
```bash
for os_file in vars/*.yml; do
  echo "Creating template from $os_file"
  ansible-playbook playbooks/create_template.yml -e @"$os_file"
done
```

### Integration with CI/CD

The playbooks can be integrated into CI/CD pipelines for automated template updates:

```yaml
# .github/workflows/update-templates.yml
name: Update Proxmox Templates
on:
  schedule:
    - cron: '0 2 * * 0'  # Weekly on Sunday
  workflow_dispatch:

jobs:
  update-templates:
    runs-on: self-hosted
    steps:
      - uses: actions/checkout@v3
      - name: Update Ubuntu Template
        run: |
          ansible-playbook playbooks/create_template.yml \
            -e @vars/ubuntu-24.04.yml
```

## 💡 Use Cases

### Infrastructure as Code

These templates integrate perfectly with infrastructure automation tools:

- **Terraform**: Clone templates to provision VMs with predictable configurations
- **Ansible**: Use cloud-init for post-deployment configuration
- **Packer**: Create custom images based on these foundational templates

### Development Environments

- Quickly spin up consistent development environments
- Test applications across different operating systems
- Create isolated environments for different projects

### Production Workloads

- Deploy containerized applications (Docker, Kubernetes)
- Create database clusters with consistent base configurations
- Build scalable web application infrastructure

## 🤝 Contributing

1. Fork the repository
2. Create a feature branch: `git checkout -b feature/new-os-support`
3. Add your OS configuration in `vars/`
4. Test the template creation
5. Submit a pull request

### Adding New OS Support

When adding support for new operating systems:

1. Research the official cloud image URL
2. Create a new variable file in `vars/`
3. Test template creation
4. Update this README with the new OS information
5. Consider resource requirements and adjust defaults

## 🔒 Security Considerations

- Templates created with this automation have cloud-init enabled
- Ensure proper SSH key management when using templates
- Consider network security when downloading images
- Regularly update templates to include latest security patches
- Review cloud image sources for authenticity and security

## 📖 Best Practices

### Template Management
- Create templates with minimal resource allocation
- Use descriptive naming conventions (os-version format)
- Regularly update templates with latest cloud images
- Document any customizations made to base templates

### Storage Planning
- Ensure adequate storage space for multiple templates
- Consider using dedicated storage for templates
- Monitor disk usage during image downloads
- Plan for template versioning if needed

### Network Configuration
- Verify network connectivity for image downloads
- Consider local caching for frequently used images
- Test template networking after creation
- Document any special network requirements

## 📄 License

This project is licensed under the MIT License - see the [LICENSE](LICENSE) file for details.

## 🙏 Acknowledgments

- Proxmox VE team for the excellent virtualization platform
- Cloud image maintainers for providing standardized images
- Ansible community for automation tools and best practices

## 💬 Support

For issues and questions:
- Check the [troubleshooting section](#troubleshooting)
- Review Proxmox VE documentation
- Open an issue in this repository
- Consult the Ansible documentation for automation questions
