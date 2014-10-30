Ansible Tower Bootstrapper
=======

This playbook is setup to bootstrap a tower instance into your infrastructure. This is beneficial if you have to setup multiple tower instances for testing, evaluations, or just plain fun!

Variables
=========

The variables are pulled out of environment variables. This is useful for cloud-init, you can set user-data in AWS to pass these variables in, then run the bootstrap script in this readme.

Set the following environment variables:
* LOGIN_PASS
* ADMIN_PASS
* MUNIN_PASS
* RABBIT_PASS
* TOWER_S3_BUCKET
* TOWER_S3_OBJECT
* TOWER_VERSION
* AWS_ACCESS_KEY
* AWS_SECRET_KEY

Bootstrap Script
================

This playbook was designed to be run locally. You can ssh into your machine, export your variables, and then run this playbook manually, but that's NOT the recommended way to do it. If you take this bash script and pasted it into your user-data in EC2, it should stand up a tower instance (provided you setup your variables correctly, and have the tower installer in S3).

```bash
#! /bin/bash

export LOGIN_PASS=changeme
export ADMIN_PASS=reallychangeme!
export MUNIN_PASS=usedinternally
export RABBIT_PASS=moreinternal
export TOWER_S3_BUCKET=my-unique-s3-bucket
export TOWER_S3_OBJECT=installs/ansible-tower-setup-latest.tar
export TOWER_VERSION=2.0.2
export AWS_ACCESS_KEY=myawsaccesskey
export AWS_SECRET_KEY=mysuper_secret_access_key

function install_ansible_ubuntu {
  apt-get install -y python-dev python-yaml python-paramiko python-jinja2 python-pip git-core
  pip install ansible
}

function install_ansible_centos_6 {
  yum install -y \
  http://download.fedoraproject.org/pub/epel/6/x86_64/epel-release-6-8.noarch.rpm
  install_ansible_centos
}

function install_ansible_centos_7 {
  yum install -y \
  http://download.fedoraproject.org/pub/epel/7/x86_64/e/epel-release-7-1.noarch.rpm
  install_ansible_centos
}

function install_ansible_centos {
  yum install -y python-pip git python-devel gcc
  mount -o remount,noexec,nosuid,nodev tmpfs
  pip install ansible
  mount -a
}

function clone_ansible_bootstrapper {
  if [ -z $1 ]; then
    echo "Cloning playbook to /tmp/tower"
    if [ ! -d "/tmp/tower" ]; then
      mkdir /tmp/tower
    fi
    git clone https://github.com/zarlant/ansible_tower_bootstrap.git /tmp/tower
  else
    echo "Cloning playbook to $1"
    git clone https://github.com/zarlant/ansible_tower_bootstrap.git $1
  fi
}

function bootstrap_tower {
  if [ -z $1 ]; then
    echo "Bootstrapping tower with ansible-playbook"
    ansible-playbook -i /tmp/tower/hosts -c local /tmp/tower/tower.yml
  else
    echo "Running Ansible inside $1"
    ansible-playbook -i $1/hosts -c local $1/tower.yml
  fi
}

function install_ansible {
  which yum > /dev/null
  yum=$?

  if [ $yum -eq "1" ]
  then
         echo "Installing Ansible on Ubuntu"
         install_ansible_ubuntu
  elif [ $yum -eq "0" ] 
  then
        release=`cat /etc/issue | grep release | awk '{ print $3 }' | cut -c1`
        if [ $release == "6" ]; then
          echo "Installing ansible Centos 6"
          install_ansible_centos_6
        elif [  $release == "7" ]; then
          echo "Installing ansible Centos 7"
          install_ansible_centos_7
        else
          echo "Not 6 or 7, maybe it's amazon and we'll just try the install"
          install_ansible_centos
        fi
  else
          echo "No Support for non Ubuntu/Centos O/S"
  fi
}

install_ansible
clone_ansible_bootstrapper $1
bootstrap_tower $1
```
