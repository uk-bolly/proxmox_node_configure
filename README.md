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
### token_user.yml

- Creates an api user and group and role with restricted permissions
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
### generate_token

- Creates an API token

### Storage

- Adds Storage (via API)
  Of types
  - NFS (tested and working)
  - ISCSI (TODO)
  - Backup (TODO)

#### Storage vars

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
### ISOS
- Add ISO files (via API)
  - Will download a list of ISOs to the ISO storage listed

#### ISO Vars
- These are set in vars/main.yml currently


```yml
# EL Vendors and release
download_el: true
download_alma: true
alma_url: https://repo.almalinux.org/almalinux/
alma_amd64: true
alma_arm64: true
download_oracle: true
oracle_url: https://yum.oracle.com/ISOS/OracleLinux/
oracle_amd64: true
oracle_arm64: true
# Note you will need either enterprise or developer account to enable this
download_rhel: false
rhel_url:
download_rocky: true
rocky_url: https://download.rockylinux.org/pub/rocky/
rocky_amd64: true
rocky_arm64: true
el_releases:
  - major: 8
    minor: 10
  - major: 9
    minor: 6
  - major: 10
    minor: 0

# SUSE based releases
download_suse: true
suse_amd64: true
suse_arm64: false
suse_releases:
  - major: 15
    minor: 6
    osname: Leap
suse_url: https://download.opensuse.org/distribution/leap/

# Debian based releases
download_deb: true
deb_rels:
  - major: 12
    minor: 12
  - major: 13
    minor: 2

debian_url: https://cdimage.debian.org/

# Ubuntu Releases
download_ubuntu: true
ubuntu_amd64: true
ubuntu_arm64: true
ubuntu_releases:
  - major: 22
    minor: '04'
    min: 5
  - major: 24
    minor: '04'
    min: 3
ubuntu_url: https://releases.ubuntu.com/
  ```

### Undo.yml

- Undo - if needed (WIP)

- Delete API token
