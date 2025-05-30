# Ansible-andes
Automation for Andes on a host CDOS Managed Linux Desktop distro.

It uses Ansible to deploy the Andes stack on two lxd/lxc containers (one for the web-server and one for the database server).
In order to be accessed from outside, the containers will need additional proxy devices to forward web and/or database connections.

NOTE: Change passwords from the default values. Do no stage files in the repo with passwords.


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

## deploying a monolithic Postprod instance
For postprod instances, it's easier/simpler to follow the monolithic system architecture described in the in the docs.

1. Prep files and container
 - create a yaml under `./inventory/MISSION_CODE.yml` (use `iml-20XX-XXX.yml` as a template)
 - looking at the other examples, define `fixture_filename`, `year`, `no.notif` and `git_sha`
 - place the fixture in `./fixtures/` directory (make sure it has the same name as defined)
 - run the init_andes_monolithic playbook, and make sure the containers and profiles are there

 ``` bash
 ansible-playbook playbooks/init_andes_monolithic.yml -e host=MISSION_CODE
 # check for the mission container
 lxc ls 
 # check for web proxy profil (for the mission container)
 lxc profile ls
 ```

2. Install and configure ANDES in the container
``` bash
# before running, make sure the github deploy key is available. It will be copied in the container.
ansible-playbook playbooks/config_andes_monolithic.yml -e host=MISSION_CODE
```

## Manual stuff (todo: automate this)
### Disable hibernation
CDOS Linux images may suspend and/or hibernate, you may want to [disable this](https://www.tecmint.com/disable-suspend-and-hibernation-in-linux/) if the station will be used as a server.

``` bash
sudo systemctl mask sleep.target suspend.target hibernate.target hybrid-sleep.target
```

(view state with `systemctl list-unit-files --state=masked`)

On a laptop, this will spam systemd with error messages as soon as the switch is detected (and it tries to perform a masked operation).
Take a look (need to run as root to see the `logind` logs):
``` bash
sudo journalctl --no-full -f
```

To stop spam, open `/etc/systemd/logind.conf` and set change (and uncomment) these two:
``` conf
HandleLidSwitch=ignore
HandleLidSwitchExternalPower=ignore
```
and restart the daemon service
``` bash
sudo systemctl restart systemd-logind
```

### update ansible plugins
The controller (system where scripts are executed) should have an updated version of the plugins,
``` bash
ansible-galaxy collection install community.general -f
```

### make a deploy key (to obtain Andes source code)
Create a private/public key pair and place let github use it as a deploy key.
One of the playbooks will copy the key to the right container, simply place the private part under `./files/id_rsa_andes`.

# Playbooks
## 1. `init_andes_monolithic.yml`
Use with host variable `-e host=MISSION_CODE`, for example `MISSION_CODE=IML-2024-012` to create:
- a container called MISSION_CODE
- a profile (reverse proxy) called MISSION_CODE

## 4. `config_andes_monolithic.yml`
Use with host variable `-e host=MISSION_CODE`, for example `MISSION_CODE=IML-2024-012` to install and configure andes in the contianer having the name `IML-2024-012`.

# Droits et Licence

Sa Majesté le Roi du chef du Canada représentée par le ministre du ministère des Pêches et des Océans, 2025.

MIT
[Guide pour la publication du code source libre](https://www.canada.ca/fr/gouvernement/systeme/gouvernement-numerique/innovations-gouvernementales-numeriques/logiciels-libres/guide-pour-la-publication-du-code-source-libre.html)
