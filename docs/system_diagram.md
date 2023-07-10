# System architecture

The Andes stack is completely containerized.
The services are broken up into three LXC containers:
 - database server: `andes-db`
 - webserver: `andes-web`
 - reverse proxy: `gninx-proxy`
The containers all run on the same server hardware running a CDOS manage Linux image.

The architecture is show in the [![system diagram](system-diagram.svg)](system-diagram.svg)
The lxc containers are shown are circles and lxc profiles (attached to the containers) are shown as rectangles.
Ansible creates an manages the containers and profiles.
The Ansible playbooks can either be executed from a controller (option 1) or directly from the server (options 2). 
In both cases there has to be an SSH connection, i.e., ansible uses SSH for option 1.

The diagram shows that other services may exist on the server, (e.g., another web app stack (called Specify) or some other database sever).
These are shown as lxc containers as well, but are not managed by the Ansible scripts containes in this repository.
