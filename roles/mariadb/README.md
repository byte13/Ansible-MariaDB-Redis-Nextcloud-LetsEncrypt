Role Name
=========
mariadb
This role installs MariaDB server on Ubuntu

Requirements
------------
Tested against Canonical Ubuntu 18.04 image on AWS EC2 instances

Role Variables
--------------
All variables commented and set in vars/main.yml and values are kept in vault file.
Typical command to be used from Terraform :
$ ansible-playbook --ssh-common-args='-o StrictHostKeyChecking=no' \
                   -i '${var.Ansible-PlayDir}/Backend-servers_inventory' -l AWS_MariaDB \
                   --extra-vars 'nc_serverPrivIP=${var.EC2-FrontendSRV-PrivIPv4} \
                                 nc_serverPrivFQDN=${var.EC2-FrontendSRV-PrivFQDN} \
                                 nc_dbserver=${var.EC2-BackendSRV-PrivFQDN} \
                                 nc_trusteddomain=${var.Nextcloud-FQDN} \
                                 nc_version=${var.Nextcloud-Version}' \
                   --vault-id nc@${var.Ansible-NCVaultPwd}
                   Linux_Install-NextCloud_with-roles.yml

To edit the vault, use "ansible-vault edit nc@prompt vars/nc_vault.yml"; you'll be prompted for the passphrase
or use "ansible-vault edit nc@<filename> vars/nc_vault.yml"; <filename> must contain the plaintext passphrase

nc_dbname: "{{ vault_nc_dbname }}" # Name of Nextcloud database
nc_dbuser: "{{ vault_nc_dbuser }}" # DB owner name used by Nextcloud
nc_dbuserpassword: "{{ vault_nc_dbuserpassword }}" # DB owner password
nc_dbadmin: "{{ vault_nc_dbadmin }}" # MariaDB admin username
nc_dbadminpassword: "{{ vault_nc_dbadminpassword }}" # MariaDB admin password, required for remote SQL management, only
nc_maildomain: "{{ vault_nc_maildomain }}" # Not used, yet

Dependencies
------------
Tested against Canonical Ubuntu 18.04 image on AWS EC2 instances
No Ansible role dependency.

Example Playbook
----------------
Only role to be executed on MariaDB backend server.
Nextcloud role depends on this role.

- hosts: AWS_MariaDB 
  remote_user: ubuntu
  become: yes
  become_method: sudo
  become_user: root
  roles: 
    - mariadb

License
-------

BSD

Author Information
------------------
Organisation : Byte13
Author : tag@bluewin.ch
PGP public key : 
-----BEGIN PGP PUBLIC KEY BLOCK-----

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
=QP6G
-----END PGP PUBLIC KEY BLOCK-----

