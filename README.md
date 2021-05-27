#wp-boiler

## A modern Wordpress development to production workflow boilerplate/starter

### local development(VBox & Vagrant) > version control(github) > production (Digital Ocean droplet)

#### This uses trellis and bedrock from roots.io

##### What is wp-boiler?

-

##### What does it do?

- it creates a Bedrock instalation of Wordpress roots.io/bedrock
- it creates a Trellis installation to run the bedrock installation in local development in a virtual machine.
- it uses Trellis and Ansible to provision the Digital Ocean droplet,
- it uses version control via git and github
- it automatically pulls updates/changes from version control to the digitalocean droplet.

##### development environment requirements/dependencies:

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
  ```bash
  git clone https://github.com/jasenmichael/wp-boiler [my-new-site.com]
  ```
- create your own repo in github
- remove original git remote, add your repo as remote, commit and push to gh
  `cd [my-new-site.com]`
  `git remote remove origin`
  `git remote add origin https://github.com/[your-gh-user]]/[my-new-site.com].git`
  `git add -A && git commit -m "chore: clone wp-boiler"`
  `git push --set-upstream origin main`
- configure development(local):
  `cd bedrock && composer install && cd ../`
  `cd trellis && vagrant up && vargant provision && cd ../`
  open broser to https://trellis.local

- configure production(remote)
  - config digital ocean droplet using Ubuntu 20.04lts
  - config ssh access
  - config domain and dns to point to digitalocean droplet.
  - edit group_vars
    - trellis/group_vars/all/users.yml
    - trellis/group_vars/all/vault.yml
    - trellis/group_vars/development/vault.yml
    - trellis/group_vars/development/wordpress_sites.yml
    - trellis/group_vars/production/vault.yml
    - trellis/group_vars/production/wordpress_sites.yml
  - create .vault_pass(in trellis dir) with only a string as encryption key, use quotes if containing special chars. 
  - add .vault_pass to .gitignore
  - use ansible-vault to encrypt secrets(before pushing to version control)
    ` cd trellis`
    `ansible-vault encrypt group_vars/all/vault.yml group_vars/development/vault.yml group_vars/production/vault.yml`
    and to decrypt(to access vars later)
    `ansible-vault decrypt group_vars/all/vault.yml group_vars/development/vault.yml group_vars/production/vault.yml`
  - push changes to github(make sure encrypted vault before push!)
    `git add -A && git commit -m "chore: setup wp-boiler && git push"`

notes.....
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

---

install

node, npm in dev/prod server script

apt install curl build-essential
curl -sL https://deb.nodesource.com/setup_lts.x | sudo -E bash -
sudo apt install nodejs
./npm-g-nosudo.sh
. ~/.bashrc 2> /dev/null
. ~/.zshrc 2> /dev/null

---

<!-- npm-g-nosudo.sh -->

#!/bin/sh

usage()
{
cat << EOF
usage: $0 [-d] [-v]

This script is intended to fix the common problem where npm users
are required to use sudo to install global packages.

It will backup a list of your installed packages remove all but npm,
then create a local directory, configure node to use this for global installs
whilst also fixing permissions on the .npm dir before, reinstalling the old packages.

OPTIONS:
-h Show this message
-d debug
-v Verbose
EOF
}

DEBUG=0
VERBOSE=0
while getopts "dv" OPTION
do
case $OPTION in
d)
DEBUG=1
;;
v)
VERBOSE=1
;;
?)
usage
exit
;;
esac
done

to_reinstall='/tmp/npm-reinstall.txt'

if [ 1 = ${VERBOSE} ]; then
printf "\nSaving list of existing global npm packages\n"
fi

#Get a list of global packages (not deps)
#except for the npm package
#save in a temporary file.
npm -g list --depth=0 --parseable --long | cut -d: -f2 | grep -v '^npm@\|^$' >$to_reinstall

if [ -f $NVM_DIR/nvm.sh ]; then
printf "Found a possible nvm install - this script may cause issues, so will exit\n"
printf "a list of your current node's npm packages is in $to_reinstall\n\n"
printf "If you save this file somewhere else, e.g. ~/npm-reinstall.txt, you can run\n\n"
printf "cat ~/npm-reinstall.txt | xargs npm -f install\n\n"
printf "when you add new node versions with nvm\n"
exit
fi

if [ 1 = ${VERBOSE} ]; then
printf "\nRemoving existing packages temporarily - you might need your sudo password\n\n"
fi
#List the file
#replace the version numbers
#remove the newlines
#and pass to npm uninstall

uninstall='sudo npm -g uninstall'
if [ 1 = ${DEBUG} ]; then
printf "Won't uninstall\n\n"
uninstall='echo'
fi
if [ -s $to_reinstall ]; then
cat $to_reinstall | sed -e 's/@.\*//' | xargs $uninstall
fi

defaultnpmdir="${HOME}/.npm-packages"
npmdir=''

read -p "Choose your install directory. Default (${defaultnpmdir}) : " npmdir

if [ -z ${npmdir} ]; then
npmdir=${defaultnpmdir}
else
    if [ ! -d ${npmdir} ]; then
        echo "${npmdir} is not a directory."
exit
fi
npmdir="${npmdir}/.npm-packages"
fi

if [ 1 = ${VERBOSE} ]; then
printf "\nMake a new directory ${npmdir} for our "-g" packages\n"
fi

if [ 0 = ${DEBUG} ]; then
mkdir -p ${npmdir}
npm config set prefix $npmdir
fi

if [ 1 = ${VERBOSE} ]; then
printf "\nFix permissions on the .npm directories\n"
fi

me=`whoami`
sudo chown -R $me $npmdir

if [ 1 = ${VERBOSE} ]; then
printf "\nReinstall packages\n\n"
fi

#list the packages to install
#and pass to npm
install='npm -g install'
if [ 1 = ${DEBUG} ]; then
install='echo'
fi
if [ -s $to_reinstall ]; then
cat $to_reinstall | xargs $install
fi

envfix='
export NPM_PACKAGES="%s"
export NODE_PATH="$NPM_PACKAGES/lib/node_modules${NODE_PATH:+:$NODE_PATH}"
export PATH="$NPM_PACKAGES/bin:$PATH"

# Unset manpath so we can inherit from /etc/manpath via the `manpath`

# command

unset MANPATH # delete if you already modified MANPATH elsewhere in your config
export MANPATH="$NPM_PACKAGES/share/man:$(manpath)"
'

fix_env() {
if [ -f "${HOME}/.bashrc" ]; then
printf "${envfix}" ${npmdir} >> ~/.bashrc
        printf "\nDon't forget to run 'source ~/.bashrc'\n"
    fi
    if [ -f "${HOME}/.zshrc" ]; then
printf "${envfix}" ${npmdir} >> ~/.zshrc
printf "\nDon't forget to run 'source ~/.zshrc'\n"
fi

}

echo_env() {
printf "\nYou may need to add the following to your ~/.bashrc / .zshrc file(s)\n\n"
printf "${envfix}\n\n" ${npmdir}
}

printf "\n\n"
read -p "Do you wish to update your .bashrc/.zshrc file(s) with the paths and manpaths? [yn] " yn
case $yn in
[Yy]_ ) fix_env;;
[Nn]_ ) echo_env;; \* ) printf "\nInvalid choice\n"; echo_env;;
esac

rm $to_reinstall

printf "\nDone - current package list:\n\n"
npm -g list -depth=0
