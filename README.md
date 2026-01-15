# Proxmox Node Configuration

An Ansible role for configuring Proxmox VE nodes after initial installation. This role automates post-installation setup, repository management, API token creation, storage configuration, and ISO/image downloads.

## Status

**This is still in development** - Some features are marked as TODO and may not be fully implemented.

## Credits

- @tteck for the original scripts that inspired this work

## Requirements

- **Ansible**: 2.17.1 or later
- **Proxmox VE**: Version 9.0 or later
- **Required Collections**:
  - `community.proxmox` (>=1.4.0)
  - `ansible.posix` (>=2.1.0)
  - `community.general`
  - `community.crypto`

### Installing Collections

```bash
ansible-galaxy collection install -r collections/requirements.yml
```

## Overview

This role performs the following configuration tasks on Proxmox nodes:

1. **Post-installation configuration** - Repository management, subscription settings, service configuration
2. **API user and token creation** - Creates restricted API user with appropriate permissions
3. **Storage configuration** - Adds NFS, iSCSI, and Proxmox Backup Server storage
4. **Resource pool creation** - Sets up resource pools for VM/container organization
5. **ISO and image downloads** - Downloads and imports OS ISOs and cloud images from vendors
6. **Cluster setup** - Configures cluster settings (TODO: cluster steps)

## Usage

### Basic Playbook

Create a playbook that uses this role:

```yaml
---
- name: Configure Proxmox nodes
  hosts: proxmox_nodes
  become: true
  roles:
    - role: proxmox_node_configure
```

### Inventory Example

```ini
[proxmox_nodes]
proxmox01.example.com ansible_host=192.168.1.10
proxmox02.example.com ansible_host=192.168.1.11

[master_node]
proxmox01.example.com
```

### Running with Tags

You can run specific parts of the configuration using tags:

```bash
# Post-installation only
ansible-playbook site.yml --tags post_install

# Create API user and token
ansible-playbook site.yml --tags user,token

# Download ISOs only
ansible-playbook site.yml --tags ubuntu,alma,rocky

# Storage configuration
ansible-playbook site.yml --tags nfs,iscsi,backup
```

## Configuration Variables

All variables are defined in `defaults/main.yml` and can be overridden in your playbook or inventory.

### Post-Installation Settings

```yaml
# Node type
proxmox_single_node: false          # Set to true for standalone nodes
proxmox_enterprise: false            # Set to true if using enterprise subscription

# Repository management (when not enterprise)
proxmox_disable_enterprise_sources: true
proxmox_no_subscription_repo: true
proxmox_beta_test_repo: true
proxmox_beta_enable: false          # Enable beta test repo
proxmox_disable_ceph: true          # Disable Ceph repos if not using Ceph

# Subscription reminder
proxmox_disable_subscription_reminder: true

# High Availability (when single_node is true)
proxmox_ha_disable: false
proxmox_corosync_disable: true

# System updates
apply_updates: true                 # Will reboot if required
disable_ipv6: true

# Debugging
debug_vars: false                   # Enable debug output throughout tasks

# Cluster
cluster_name: homelab
```

### API Configuration

```yaml
# API user creation
create_api_user: true
proxmox_api_user: 'ansible_api@pve'
proxmox_token_id: ansible

# API connection settings
proxmox_api_url: "{{ ansible_host }}"
proxmox_api_port: 8006
validate_certs: false

# Note: proxmox_api_token_id and proxmox_api_token_secret are
# automatically created and discovered during the play
# If create_api_user is false, you must set these manually
```

### Storage Configuration

#### NFS Storage

```yaml
nfs_storage: false
nfs_server:
  - ip: 192.168.1.100
    export: /nfs/storage
    content:
      - "rootdir"
      - "images"
      - "iso"
      - "import"
```

#### iSCSI Storage

```yaml
iscsi_storage: false
iscsi:
iscsi_portal:
iscsi_target:
iscsi_content:
  - "rootdir"
  - "images"
  - "iso"
  - "import"
```

#### Proxmox Backup Server

```yaml
backup_storage: false
backup_server: proxmox-backup-server.example.com
backup_username: backup@pbs
backup_password: password123
backup_datastore: backup
backup_fingerprint: "F3:04:D2:C1:33:B7:35:B9:88:D8:7A:24:85:21:DC:75:EE:7C:A5:2A:55:2D:99:38:6B:48:5E:CA:0D:E3:FE:66"
backup_export: "/mnt/storage01/b01pbs01"
```

### Resource Pools

```yaml
resource_pools:
  - debian
  - el8
  - el9
  - el10
  - el11
  - suse
  - ubuntu
  - win_host
  - win_server
```

### OS Download Configuration

For each OS type, you can configure:
- `{os}_iso`: Download ISO files (boolean)
- `{os}_image`: Download and import cloud images (boolean)
- `{os}_amd64`: Download x86_64/amd64 versions (boolean)
- `{os}_arm64`: Download aarch64/arm64 versions (boolean)
- `{os}_releases`: List of versions to download

#### AlmaLinux

```yaml
alma_iso: true
alma_image: true
alma_amd64: true
alma_arm64: true
alma_releases:
  - os_version: 9
    os_release: 7
  - os_version: 10
    os_release: 1
```

#### Rocky Linux

```yaml
rocky_iso: true
rocky_amd64: true
rocky_arm64: true
rocky_image: true
rocky_releases:
  - os_version: 8
    os_release: 10
  - os_version: 9
    os_release: 6
  - os_version: 10
    os_release: 1
```

#### Oracle Linux

```yaml
oracle_iso: true
oracle_amd64: true
oracle_arm64: false
oracle_image: true
oracle_releases:
  - os_version: 9
    os_release: 7
  - os_version: 10
    os_release: 1
```

#### Ubuntu

```yaml
ubuntu_iso: true
ubuntu_amd64: true
ubuntu_arm64: true
ubuntu_image: false
ubuntu_releases:
  - os_version: 22
    os_release: '04'
    min: 5
  - os_version: 24
    os_release: '04'
    min: 3
```

#### Debian

```yaml
deb_iso: true
debian_amd64: true
debian_arm64: true
debian_image: false
debian_release:
  - major: 11
    minor: 11
  - major: 12
    minor: 12
  - major: 13
    minor: 2
```

#### SUSE

```yaml
suse_iso: true
suse_amd64: true
suse_arm64: true
suse_image: true
suse_releases:
  - os_version: 15
    os_release: '6'
    osname: Leap
```

### Packer Integration Variables

```yaml
proxmox_vm_cpu_type: host
proxmox_vm_cpu_sockets: 1
proxmox_vm_cpu_core: 2
proxmox_vm_memory: 2048
```

## Task Tags

The role uses tags to allow selective execution:

- `post_install` - Post-installation configuration
- `user` - API user creation
- `token` - API token generation
- `pools` - Resource pool creation
- `nfs` - NFS storage configuration
- `iscsi` - iSCSI storage configuration
- `backup` - Proxmox Backup Server configuration
- `ubuntu` - Ubuntu ISO/image downloads
- `alma` - AlmaLinux ISO/image downloads
- `rocky` - Rocky Linux ISO/image downloads
- `oracle` - Oracle Linux ISO/image downloads
- `suse` - SUSE ISO/image downloads
- `cluster_setup` - Cluster configuration (master node only)
- `undo` - Undo operations (WIP)

## File Structure

```
proxmox_node_configure/
├── collections/
│   └── requirements.yml          # Ansible collection requirements
├── defaults/
│   └── main.yml                 # Default variables
├── handlers/
│   └── main.yml                 # Ansible handlers
├── meta/
│   └── main.yml                 # Role metadata
├── tasks/
│   ├── main.yml                 # Main task file
│   ├── post_install.yml         # Post-installation tasks
│   ├── token_user.yml           # API user creation
│   ├── generate_token.yml       # API token generation
│   ├── resource_pools.yml       # Resource pool creation
│   ├── single_node.yml          # Single node configuration
│   ├── cluster_setup.yml        # Cluster setup (TODO)
│   ├── clustered_node.yml       # Clustered node configuration
│   ├── undo.yml                 # Undo operations (WIP)
│   ├── storage/
│   │   ├── nfs.yml              # NFS storage
│   │   ├── iscsi.yml            # iSCSI storage (TODO)
│   │   └── backup.yml           # Backup storage (TODO)
│   ├── almalinux/
│   │   ├── main.yml
│   │   ├── alma_isos.yml
│   │   └── alma_import.yml
│   ├── rockylinux/
│   │   ├── main.yml
│   │   ├── rocky_isos.yml
│   │   └── rocky_import.yml
│   ├── ubuntulinux/
│   │   ├── main.yml
│   │   ├── ubuntu_isos.yml
│   │   └── ubuntu_import.yml
│   ├── debianlinux/
│   │   ├── main.yml
│   │   ├── debian_isos.yml
│   │   └── debian_import.yml
│   ├── oraclelinux/
│   │   ├── main.yml
│   │   ├── oracle_isos.yml
│   │   └── oracle_import.yml
│   └── suselinux/
│       ├── main.yml
│       ├── suse_isos.yml
│       └── suse_import.yml
├── templates/
│   ├── nonag_conf.j2            # Subscription reminder disable config
│   ├── nonag_sh.j2              # Subscription reminder disable script
│   ├── pve-install-repo.sources.j2  # Repository sources template
│   └── pvetest-for-beta.sources.j2  # Beta repository template
├── vars/
│   ├── main.yml                 # Variable definitions
│   ├── almalinux.yml            # AlmaLinux specific vars
│   ├── rockylinux.yml           # Rocky Linux specific vars
│   ├── ubuntulinux.yml          # Ubuntu specific vars
│   ├── debianlinux.yml          # Debian specific vars
│   ├── oraclelinux.yml          # Oracle Linux specific vars
│   └── suselinux.yml            # SUSE specific vars
├── site.yml                     # Example playbook
└── README.md                    # This file
```

## Known Limitations / TODO

- Cluster setup steps are not yet fully implemented
- Enterprise license addition is not yet implemented
- iSCSI storage configuration is marked as TODO
- Backup storage configuration is marked as TODO
- Undo functionality is work in progress
- Windows OS support is not yet implemented

## Troubleshooting

### Debug Mode

Enable debug output to investigate issues:

```yaml
debug_vars: true
```

Then run with verbose output:

```bash
ansible-playbook site.yml -vvv
```

### Common Issues

1. **API token creation fails**: Ensure the user has appropriate permissions to create API tokens
2. **ISO downloads fail**: Check network connectivity and verify the download URLs are accessible
3. **Storage configuration fails**: Verify storage credentials and network connectivity
4. **Version check fails**: Ensure Proxmox VE version is 9.0 or later

## License

MIT License

## Contributing

Contributions are welcome. Please ensure:
- Code follows Ansible best practices
- Variables are properly documented
- Tasks are idempotent where possible
- Test your changes before submitting
