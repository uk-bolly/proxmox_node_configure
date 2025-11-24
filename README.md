# Configure proxmox nodes

## This is still in development

## Credits
- @tteck for the original scripts that have inspired this work

## Overview

Once a system has been built before any other configuration takes place. This can setup your based on variables below:

min_ansible_version: 2.17.1
```yml
# Ability to output debug for investigation
# With debugs throughout the tasks this will help to find where issues maybe
debug_vars: false

# file to show setting for proxmox configuration


## Stages

- Run post install steps
  including:
  - If a single node or cluster node (TODO cluster steps)
  - Using enterprise (testing needed if is enterprise)
    - should add license (TODO)
    - If not enterprise
      - Disables repo
      - Adds no subscription repo
      - Turn other repos on or off
      - remove nag that you are not licensed
      - Disable HS
      - Disable corosync

### post install variables

```yml
# If just a single node and not cluster
proxmox_single_node: true

# If server has a proxmox subscription
proxmox_enterprise: false

# Disable subscription sources
# When not enterprise
proxmox_disable_enterprise_sources: true

# Disabled ceph repos if not using ceph storage
proxmox_disable_ceph: true

# Add the no subscription repo
# When not enterprise
proxmox_no_subscription_repo: true

# Add the beta test repo
proxmox_beta_test_repo: true

# Enable the beta_test repo
proxmox_beta_enable: false

# Turn off subscription reminder
# When not enterprise
proxmox_disable_subscription_reminder: true

# Turn off HA service
# When single node true
proxmox_ha_disable: true

# Turn off corosync services
# When single node true
proxmox_corosync_disable: true

## Allow latest packages to be installed
# Will reboot if required
apply_updates: true

## Disable ipv6
disable_ipv6: true
```

- Creates an api user and group and role with restriuced permissions
  - Utilises CLI

### api vars

```yml
## API settings
# The following are created and discovered during the play, bt can be set manually
# The option to create user needs to be false
# proxmox_api_token_id : # Make sure values is not full string as per proxmox e.g user@pam!token_name only token_name is variable
# proxmox_api_token_secret: This is created and discovered
create_api_user: true
proxmox_api_url: "{{ ansible_host }}"
proxmox_api_port: 8006
proxmox_api_user: 'ansible_api@pve'
proxmox_token_id: ansible
```

- Creates an API token

- Adds Storage (via API)
  Of types
  - NFS (tested and working)
  - ISCSI (TODO)
  - Backup (TODO)

### Storage vars

```yml
nfs_storage: false
nfs_server:
  - ip: xx.xx.xx.xx
    export: /nfs/someting
    content:
      - "rootdir"
      - "images"
      - "iso"
      - "import"
# Backup
backup_storage: false
backup_server: proxmox-backup-server.example.com
backup_username: backup@pbs
backup_password: password123
backup_datastore: backup
backup_fingerprint: "F3:04:D2:C1:33:B7:35:B9:88:D8:7A:24:85:21:DC:75:EE:7C:A5:2A:55:2D:99:38:6B:48:5E:CA:0D:E3:FE:66"
backup_export: "/mnt/storage01/b01pbs01"

# ISCSI
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

- Add ISO files (via API)
  - Will download a list of ISOs to the ISO storage listed

### vars for ISOs

```yml
ISOs:
  # Alma
  - url: https://repo.almalinux.org/almalinux/9.6/isos/x86_64/AlmaLinux-9.6-x86_64-minimal.iso
  - url: https://repo.almalinux.org/almalinux/8.10/isos/x86_64/AlmaLinux-8.10-x86_64-minimal.iso
  # Debian
  - url: https://cdimage.debian.org/cdimage/archive/12.12.0/amd64/iso-cd/debian-12.12.0-amd64-netinst.iso
  - url: https://cdimage.debian.org/debian-cd/current/amd64/iso-cd/debian-13.2.0-amd64-netinst.iso
  # Rocky
  - url: https://download.rockylinux.org/pub/rocky/9/isos/x86_64/Rocky-9.6-x86_64-minimal.iso
  - url: https://download.rockylinux.org/pub/rocky/8/isos/x86_64/Rocky-8.10-x86_64-minimal.iso
  # opensuse
  - url: https://download.opensuse.org/tumbleweed/iso/openSUSE-Tumbleweed-NET-x86_64-Current.iso
  - url: https://download.opensuse.org/distribution/leap/16.0/offline/Leap-16.0-online-installer-x86_64.install.iso
  # Oracle
  - url: https://yum.oracle.com/ISOS/OracleLinux/OL9/u6/x86_64/OracleLinux-R9-U6-x86_64-boot.iso
  - url: https://yum.oracle.com/ISOS/OracleLinux/OL10/u0/x86_64/OracleLinux-R10-U0-x86_64-boot.iso
  # Ubuntu
  - url: https://releases.ubuntu.com/22.04/ubuntu-22.04.5-live-server-amd64.iso
  - url: https://releases.ubuntu.com/24.04.3/ubuntu-24.04.3-live-server-amd64.iso
  ```

- Undo - if needed (WIP)

- Delete API token
