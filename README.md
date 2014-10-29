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

Example Environment Variables
=============================

export LOGIN_PASS=changeme
export ADMIN_PASS=reallychangeme!
export MUNIN_PASS=usedinternally
export RABBIT_PASS=moreinternal
export TOWER_S3_BUCKET=my-unique-s3-bucket
export TOWER_S3_OBJECT=installs/ansible-tower-setup-latest.tar
export TOWER_VERSION=2.0.2
export AWS_ACCESS_KEY=myawsaccesskey
export AWS_SECRET_KEY=mysuper_secret_access_key

Bootstrap Script
================

This playbook was designed to be run locally. You can ssh into your machine, export your variables, and then run this playbook manually, but that's NOT the recommended way to do it.

```bash

```
