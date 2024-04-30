# Andes

One type of deployment that was beneficial to automate was an andes monolithic stack. The playbook `init_andes_monolithic.yml`
will create an initialize (create container, create profile attache profile) an lxc container.
and the playbook `config_andes_monolithic.yml` will configure the container with all it needs (web server, mysql, Andes etc) for a functional Andes instance.

There will be a lxc contain with the name `CHOOSE_A_HOSTNAME` as well as an attached lxc profile with the name `CHOOSE_A_HOSTNAME` that will forward web and database traffic according to pre-defined ports.

you must first create a host inventory file similar to the following template:

``` yaml
#
# for CHOOSE_A_HOSTNAME

vars:
  hosts:
    CHOOSE_A_HOSTNAME:
      fixture_filename: andes_v2.11_0f05f31d_IML2024TBD_2024_04_04_21_02_59_mysql.sql
      web_port: 24011 # YEAR*1000 + MISSION NUM
      db_port: 24989 # countdown (YEAR+1)*1000 - MISSION NUM
      git_sha: 0f05f31d
      host: CHOOSE_A_HOSTNAME

local:
  hosts:
    CHOOSE_A_HOSTNAME:

andes_monolithic:
  hosts:
    CHOOSE_A_HOSTNAME:
```
where you can choose the hostname, web and database port, a starting fixture and the git SHA.
The other variables are inherited from the `andes_monolithic` inventory under `group_vars/andes_monolithic.yml`.
Variable can be overwritten in the host inventory and should ideally NOT be modifying in the `andes_monolithic` inventory.

playbooks are launched by specifing the host using the `-e` flag:
``` bash
ansible-playbook playbooks/init_andes_monolithic.yml -e 'host=CHOOSE_A_HOSTNAME'
ansible-playbook playbooks/config_andes_monolithic.yml -e 'host=CHOOSE_A_HOSTNAME'
```


## port numbers,
I try to choose predictable port numbers. I want to expose two services, i) web and ii) database.
Here is what we can work with:
 - The maximum port number is 65535.
 - Every missions should contain a notification number / mission number / year.

First, I can assume all missions are from the same institution (IML), so a contanenation of year and mission number would be nice.
It would be extra nice to concatenat the four-year digits (`2020`) with the three mission digits (`033`), and one of `80` (for web) or `3306` (for database. In this example, the resulting port number would be `20200333306` which is too long. 

Instead, I use a scheme that yields the numbers `20033` (for web) and `20077` (for database). For the web service, the template is a contatenation of year and misison number `<YY><NNN>`, for the database I go backwards and use `<YY><MMM>` where `XXX` is a countDOWN using the mission number (100-033=077).


