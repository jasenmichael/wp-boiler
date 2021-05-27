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
ansible-playbook server.yml -e env=production

install ssh stuff so ansible works

create ssh key for github
 ssh-keygen -t rsa -b 4096 -C "jasen@jasenmichael.com"

---
db stuff

https://discourse.roots.io/t/database-syncing/7283/10

# export production alias to local
wp db export \ # backup the local database
&& wp db reset \ # optional - create a blank slate
&& wp @production db export - | wp db import - \ # export the production database and pipe it into our local database
&& wp search-replace '//production-url.com' '//local-url.test' --skip-columns=guid --report-changed-only \
&& wp cache flush

# export local db to staging alias
wp @staging db export \ # backup the staging database
&& wp @staging db reset \ # optional - create a blank slate on staging
&& wp db export - | wp @staging db import - \ # export the local database and pipe it into our staging database
&& wp @staging search-replace '//local-url.test' '//staging-url.com' --skip-columns=guid --report-changed-only \
&& wp @staging cache flush
