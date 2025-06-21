Linux Active Directory Domain Join with ansible
=========

This is an ansible role that join Linux machine to Active directory domain using realm, sssd and samba-winbind.

This role is tested only on Ubuntu 20.04

Features
------------

- Support both `sssd` and `winbind` via `realm`
- Can restrict login policy
- Add sudoers


Requirements
------------

### Host

- ansible > 2.1

### Target

- Ubuntu
  - Internet connection (currently under proxy environment does not supported)

- Debian
  - Internet connection (currently under proxy environment does not supported)

- RHEL/Centos 7
  - Internet connection (currently under proxy environment does not supported)
  - NOTE: Centos 7 only tested with SSSD. if you want to use join with winbind, it may not work as expected

Role Variables
--------------

### defaults/main.yml

- basic domain information

  |variable|description|
  |--------|-----------|
  |`join_method`| join domain method (default: `sssd`, possible value: `sssd` or `winbind`)|
  |`domain_name`| Active directory domain name|
  |`domain_netbios_name`| Active directory NetBIOS name (winbind only)|
  |`domain_admin_user`| User that can join to domain (default: Administrator)
  |`domain_admin_password`| Domain admin password|

- misc

  |variable|description|
  |--------|-----------|
  |`domain_sudoers`|list of group name that want to give sudo permission. if you don't want to give sudo permission, than leave as blank<br/>(default: administrators, domain admins, enterprise admins)|
  |`user_home_creation`|if enabled, home directory created when new user login (default: yes)|
  |`set_default_domain`|if enabled, set domain as system default (default: yes)|

- sssd configuration

  - login policy

    the below configuration is stands for restrict login policy when using `sssd`. if you don't want to restrict login policy, than set `sssd_permit_deny_all`to `no` and leave blank others

    |variable|description|
    |--------|-----------|
    |`sssd_permit_deny_all`|disable login for all users and groups. this should be yes if want to restrict login with below allow list (default: yes)|
    |`sssd_permit_allow_groups`|allow login specific groups<br/>(default: administrators, domain admins, enterprise admins)|
    |`sssd_permit_allow_users`|give permission to login in specific user (default: blank)|
    |`sssd_permit_block_groups`|block login specific groups (default: blank)|
    |`sssd_permit_block_users`|block login specific users (default: blank)|

  - dynamic dns update

    |variable|description|
    |--------|-----------|
    |`sssd_dyndns_update`| enable/disable dyndns option (default: yes)|
    |`sssd_dyndns_refresh_interval`| `dyndns_refresh_inverval` (default: 43200) |
    |`sssd_dyndns_update_ptr`| `dyndns_update_ptr` (default: true)|
    |`sssd_dyndns_ttl` | `dyndns_ttl` (default: 3600) |

- winbind configuration
  
  - idmap

    note that current implementation on winbind idmap is `tdb` only. no other methods are supported.

    |variable|description|
    |--------|-----------|
    |`winbind_idmap_default_range`|idmap range<br/>(default: 10000-999999)|
    |`winbind_idmap_current_domain_range`|default domain id map range<br/>(default: 2000000-2999999)|

  - login policy

    if you don't want to restrict login policy, than leave as blank

    |variable|description|
    |--------|-----------|
    |`winbind_permit_allow_sids`|sids that allow login<br/>(default: BUILTIN\administrator)|
    |`winbind_permit_allow_names`|group name that allow login<br/>(default: domain admins, enterprise admins)|


### vars/main.yml

|variable|description|
|--------|-----------|
|`sudoers_path`|sudoers config directory location (default: `/etc/sudoers.d`)|
|`sssd_config_path`|sssd config location (default: `/etc/sssd/sssd.config`)|
|`samba_config_path`|samba config location (default: `/etc/samba/smb.conf`)|
|`template_sudoers_sssd`|sudoers template for sssd (default: `sudoers.sssd.j2`)|
|`template_sudoers_winbind`|sudoers template for winbind (default: `sudoers.winbind.j2`)|

Dependencies
------------

- [community.general](https://docs.ansible.com/ansible/latest/collections/community/general/index.html)

Example Playbook
----------------

- minimum setup (sssd)

  The minimum setup for sssd. after join domain, you can login with following groups
  - administrator
  - domain admins
  - enterprise admins

  ```yml
      - hosts: servers
        roles:
          - lazyrichard.linux_join_ad_domain
        vars:
          join_method: sssd
          domain_name: contoso.com
          domain_admin_user: domain_admin
          domain_admin_password: really-strong-password
  ```

- minimum setup (winbind)

  The minimum setup for winbind. after join domain, you can login with following groups
  - administrator
  - domain admins
  - enterprise admins

  ```yml
      - hosts: servers
        roles:
          - lazyrichard.linux_join_ad_domain
        vars:
          join_method: winbind
          domain_name: contoso.com
          domain_netbios_name: CONTOSO
          domain_admin_user: domain_admin
          domain_admin_password: really-strong-password
  ```


License
-------

BSD
