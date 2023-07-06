# Ansible-andes
Automation for Andes on a host CDOS Managed Linux Desktop distro.

It uses Ansible to deploy the Andes stack on two lxd/lxc containers (one for the web-server and one for the database server).
In order to be accessed from outside, the containers will need additional proxy devices to forward web and/or database connections.

NOTE: the passwords in this repo are completely accessible. It doesn't matter as long as DB connections are not permitted from outside, so use the DB proxy with care. For now, the database web-app user is not limited to connections from its andes-web.lxd host. This is for debugging purposed and creating db connections to see the tables. DO NOT create a proxy database device untill the database is properly sealed off.



# Quickstart
Get `pip`
``` bash   
curl https://bootstrap.pypa.io/get-pip.py -o get-pip.py
python3 get-pip.py --user
rm get-pip.py 
```

install and activate virtualenv
``` bash
python3 -m pip install virtualenv --user
python3 -m virtualenv venv
source venv/bin/activate
```
Now for ansible
``` bash
pip install ansible
```
now run the playbook
```
ansible-playbook playbook.yml
```

## Manual stuff (todo: automate this)
- private andes deploy key, make sure there is `./files/id_rsa_andes` under

- need to create the containers
 ```
lxc launch ubuntu:22.04 andes-web
lxc launch ubuntu:22.04 andes-db
lxc launch ubuntu:22.04 nginx-proxy
ansible-galaxy collection install community.general -f

# Make these containers available outside
attach the proxy profiles
``` bash
lxc profile add andes-web proxy-web 
lxc profile add andes-db proxy-db
```

where the profiles are:
``` yaml
config: {}
description: ""
devices:
  hostport80:
    connect: tcp:127.0.0.1:80
    listen: tcp:0.0.0.0:80
    type: proxy
name: proxy-web
```
and
``` yaml
config: {}
description: ""
devices:
  hostport3389:
    connect: tcp:127.0.0.1:3389
    listen: tcp:0.0.0.0:3389
    type: proxy
name: proxy-db
```
# Upload new fixtures from andes-preprod-east

Download and dump the fixtures (as a zip file) in `./fixtures/`

The `update_fixtures.yaml` playbook script will take care of finding the most recent one and will send it to the container to be loaded. Note that a backup of the older database will be made, before it is dropped.

# TODO:
- time server
- printer