
~~ Setup Wireguard Server ~~

A guide for configuring a wireguard proxy for a Matrix server with networking/firewall issues. The AWX tower will be able to SSH into the wireguard client via port 2222 afterwards and deploy the Matrix server.

1) Create a Debian 11 machine for the wireguard server, it only needs 1GB of RAM. Add the AWX hosts public SSH key to it.


2) Create a Debian 11 machine for the wireguard client. Add the AWX hosts public SSH key to it.


3) Enter these details into the vars.yml:

```
# WireGuard Settings
server_ip : 192.168.1.100
server_hostname: wireguard.gomatrixhosting.com
```


4) Run 'setup_wireguard_server.yml' playbook:

`$ ansible-playbook -i ./inventory/hosts_server setup_wireguard_server.yml`


5) Run 'setup_wireguard_clients.yml' playbook:

`$ ansible-playbook -i ./inventory/hosts_clients --tags "setup-server, wireguard" setup_wireguard_clients.yml`
