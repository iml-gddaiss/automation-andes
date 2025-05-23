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

## deploying a monolithic Postprod instance
For postprod instances, it's easier/simpler to follow the monolithic system architecture described in the in the docs.

1. Prep files and container
 - create a yaml under `./inventory/host_vars/<MISSION>.yml`
 - looking at the other examples, define `fixture_filename` and `web_port` variables
 - place the fixture in `./fixtures/` directory (make sure it has the same name as defined)
 - run the init_postprod_containers playbook, and make sure the containers and profiles are there:

 ```
 ansible-playbook playbooks/init_postprod_containers.yml
 # check for the mission container
 lxc ls 
 # check for web proxy profil (for the mission container)
 lxf profile ls

 ```
2. config the container
- host in config_postpro.yml (consider copying as a new file)
 ansible-playbook playbooks/config_postprod.yml 
 
## Manual stuff (todo: automate this)
### Disable hibernation
CDOS Linux images may suspend and/or hibernate, you may want to [disable this](https://www.tecmint.com/disable-suspend-and-hibernation-in-linux/) if the station willbe used as a server.

``` bash
sudo systemctl mask sleep.target suspend.target hibernate.target hybrid-sleep.target
```

### update ansible plugins
The controller (system where scripts are executed) should have an updated version of the plugins,
``` bash
ansible-galaxy collection install community.general -f
```

### make a deploy key (to obtain Andes source code)
Create a private/public key pair and place let github use it as a deploy key.
One of the playbooks will copy the key to the right container, simply place the private part under `./files/id_rsa_andes`.


### (obsolete) create the containers
NOTE: This is now automated using the `init_containers.yml` playbook, it's kept here in case manual interfention is needed.

``` bash
lxc launch ubuntu:22.04 andes-web
lxc launch ubuntu:22.04 andes-db
lxc launch ubuntu:22.04 nginx-proxy
```

attach the proxy profiles to forward incomming requests, in this case example it is `andes-web` and `andes-db`, but both should be pointing to `gninx-proxy`
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

# Playbooks
## 1. `init_containers.yml`
Will create 3 containers `andes-web`, `andes-web` and `gninx-proxy`.

## 2. `config_andes_db.yml`
Will configure a MySQL server in the `andes-db` container

## 3. `config_andes_web.yml`
Will download the andes source code and configure a webserver in the `andes-web` container

## 4. `config_gninx.yml`
Will configure a reverse proxy in the `nginx-proxy` container.

## 5. `update_andes.yml`
Will update the andes instance: pull new code into `andes-web` and perform migrations.

## 6. `update_fixtures.yml`
Will restore using the most recent a backup found under `./fixtures`.
This will wipe the database and restore whatever was in the backup. A backup will be performed before attempting the restore.

# TODO:
- time server
- printer

# Droits et Licence

Sa Majesté le Roi du chef du Canada représentée par le ministre du ministère des Pêches et des Océans, 2025.

MIT
[Guide pour la publication du code source libre](https://www.canada.ca/fr/gouvernement/systeme/gouvernement-numerique/innovations-gouvernementales-numeriques/logiciels-libres/guide-pour-la-publication-du-code-source-libre.html)
