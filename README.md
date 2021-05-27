#wp-boiler

## A modern development workflow Wordpress setup

### This uses trellis and bedrock from roots.io
### Ansible to deploy to a Digital Ocean droplet

What this does:
it creates a Bedrock instalation of Wordpress read more here - roots.io/bedrock

it creates a Trellis installation to run the bedrock instalation in local development in a virtual machine.

it uses Trellis and Ansible to provision the Digital Ocean droplet, and push the Bedrock instalation there.

development environment requirements/dependencies:
- php-cli
- composer
- node, npm
- pip
- ansible
- virtualbox
- vagrant
- nfs-common

---

instalation process:
- clone this repo
- create your own repo
- remove original git remote, add your repo as remote
- configure development(local):
  -  ```cd bedrock && composer install && cd ../```
  -  ```cd trellis && vagrant up && vargant provision && cd ../```
  -  open broser to https://trellis.local

- configure staging(remote)
  - to do

- configure production(remote)
  - to do
  - edit vars
  - trellis vault
  - create .vault_pass


 <!-- ansible-vault encrypt group_vars/all/vault.yml group_vars/development/vault.yml group_vars/production/vault.yml  -->
 <!-- ansible-vault decrypt group_vars/all/vault.yml group_vars/development/vault.yml group_vars/production/vault.yml  -->

pip install -r requirements.txt
ansible-galaxy install -r galaxy.yml


