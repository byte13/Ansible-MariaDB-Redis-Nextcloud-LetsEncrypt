# Ansible-Nextcloud-LetsEncrypt
This set of Ansible roles install MariaDB, Redis and Nextcloud on Ubuntu.

To install Ansible on Ubuntu, please follow instructions available there :
* [Install Ansible on Ubuntu 18.04-LTS](https://linuxconfig.org/how-to-install-ansible-on-ubuntu-18-04-bionic-beaver-linux)

If you want to run it on a two tiers Virtual Private Cloud (VPC) in AWS of on a two tiers Resource Group (RG) in Azure, you may be interrested in either of these :
* [byte13/Terraform-AWS-TwoTiersVPC](https://github.com/byte13/Terraform-AWS-TwoTiersVPC)
* [byte13/Terraform-Azure-TwoTiersRG](https://github.com/byte13/Terraform-Azure-TwoTiersRG)

## Please, read carefully this REAMDE file up to the end !
It contains crucial information regarding protection of 
secrets set during installation.

## Main steps to play this Ansible playbook :
1. Ensure you have a pair of SSH keys authorized on target system(s)
2. Unlock the SSH private key using the following commands :
```
    $ ssh-agent
    $ ssh-add <path-to-the-private-key>
        ... when prompted enter the passphrase
```  
3. Go to parent directory of this playbook
4. Edit host repository file to specify your hosts
```
    $ vi Backend-servers_inventory
    $ vi Frontend-servers_inventory
``` 
5. Edit the vaults containing secrets (see instructions below)
6. Syntax to run the playbook to install MariaDB on backend server :
```
   $ ansible-playbook --ssh-common-args='-o StrictHostKeyChecking=no' -i '<your inventory file>' -l '<your group in inventory file>' --extra-vars 'nc_serverPrivIP=<IPv4 address of the frontend server running Nextcloud> nc_serverPrivFQDN=<FQDN of the frontend server running Nextcloud>' --vault-id nc@<ncVault_pwd.txt | prompt> Linux_Install-NextCloud_with-roles.yml
```
7. Syntax to run the playbook to install Nextcloud on frontend server :
``` 
   $ ansible-playbook --ssh-common-args='-o StrictHostKeyChecking=no' -i '<your inventory file>' -l '<your group in inventory file>' --extra-vars 'nc_dbserver=<FQDN to be used to connect to MariaDB service> nc_datadir=<Nextcloud data directory> nc_version=<Version of Nextcloud to be installed> nc_trusteddomain=<Public FQDN used to access Nextcloud service> le_email=<e-mail to be used for Let's Encrypt to send notifications>' --vault-id redis@<redisVault_pwd.txt | prompt > --vault-id nc@<ncVault_pwd.txt | prompt> Linux_Install-NextCloud_with-roles.yml
```

## Directory structure :
```
.
├── Linux_Install-NextCloud_with-roles.yml
├── ncVault_pwd.txt
├── README.md
├── redisVault_pwd.txt
└── roles
    ├── apache-php-mysqlcli
    │   ├── defaults
    │   │   └── main.yml
    │   ├── files
    │   ├── handlers
    │   │   └── main.yml
    │   ├── meta
    │   │   └── main.yml
    │   ├── README.md
    │   ├── tasks
    │   │   └── main.yml
    │   ├── templates
    │   ├── tests
    │   │   ├── inventory
    │   │   └── test.yml
    │   └── vars
    │       └── main.yml
    ├── lets-encrypt
    │   ├── defaults
    │   │   └── main.yml
    │   ├── files
    │   ├── handlers
    │   │   └── main.yml
    │   ├── meta
    │   │   └── main.yml
    │   ├── README.md
    │   ├── tasks
    │   │   └── main.yml
    │   ├── templates
    │   ├── tests
    │   │   ├── inventory
    │   │   └── test.yml
    │   └── vars
    │       └── main.yml
    ├── mariadb
    │   ├── defaults
    │   │   └── main.yml
    │   ├── files
    │   │   └── my.cnf
    │   ├── handlers
    │   │   └── main.yml
    │   ├── meta
    │   │   └── main.yml
    │   ├── README.md
    │   ├── tasks
    │   │   └── main.yml
    │   ├── templates
    │   ├── tests
    │   │   ├── inventory
    │   │   └── test.yml
    │   └── vars
    │       └── main.yml
    ├── nextcloud
    │   ├── defaults
    │   │   └── main.yml
    │   ├── files
    │   │   └── my.cnf
    │   ├── handlers
    │   │   └── main.yml
    │   ├── meta
    │   │   └── main.yml
    │   ├── README.md
    │   ├── tasks
    │   │   └── main.yml
    │   ├── templates
    │   │   ├── apache_nextcloud_site.j2
    │   │   ├── apache_nextcloud_site.j2_le-ssl
    │   │   ├── config.php.j2
    │   │   ├── php.ini.j2
    │   │   └── php.ini.j2.old
    │   ├── tests
    │   │   ├── inventory
    │   │   └── test.yml
    │   └── vars
    │       ├── main.yml
    │       └── nc_vault.yml
    └── redis
        ├── defaults
        │   └── main.yml
        ├── files
        ├── handlers
        │   └── main.yml
        ├── meta
        │   └── main.yml
        ├── README.md
        ├── tasks
        │   └── main.yml
        ├── templates
        │   └── redis.conf.j2
        ├── tests
        │   ├── inventory
        │   └── test.yml
        └── vars
            ├── main.yml
            └── redis_vault.yml
```

Secrets, like MariaDB and application accounts passwords are kept in 2 vaults :

* roles/redis/vars/redis_vault
* roles/nextcloud/vars/nc_vault

The following files contain the respective passphrase to unlock the vaults :

* ncVault_pwd.txt
* redisVault_pwd.txt

# !!! WARNING - WARNING - WARNING !!!
**After cloning this playbook from GitHub and before running this Ansible playbook for the first time**, change all values in the vaults using the following command from the parent directory :
```
    $ ansible-vault edit --vault-id redis@redisVault_pwd.txt roles/redis/vars/redis_vault.yml
    $ ansible-vault edit --vault-id nc@ncVault_pwd.txt       roles/nextcloud/vars/nc_vault.yml
```
Then, change the passphrase of each safe using the following command from the parent directory :
```
    $ ansible-vault rekey --vault-id redis@redisVault_pwd.txt --new-vault-id redis@prompt roles/redis/vars/redis_vault.yml
    $ ansible-vault rekey --vault-id nc@redisVault_pwd.txt    --new-vault-id nc@prompt    roles/nc/vars/nc_vault.yml
```
Note that you will be prompted for the new passphrase.\
Then change the passphrase in repsective plain text file :
* ncVault_pwd.txt
* redisVault_pwd.txt

or specify respective password when prompted ( if you use @prompt instead of @&lt;path to file&gt; in aforementionned  ansible-playbook argument)

If you want to push your own tree on Github, remember to change the password with dummy ones, or prevent the upload of your files to github using [.gitignore](https://help.github.com/en/articles/ignoring-files)
