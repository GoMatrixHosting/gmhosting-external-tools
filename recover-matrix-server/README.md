
# Recovers a Matrix Server

Used to recover a Matrix server connected to AWX. This tool should be run outside of AWX.


# Instructions:

1) On the controller create a SSH config entry for the old servers IP:
```
Host 128.199.235.90
    HostName 128.199.235.90
    User root
    Port 22
    IdentityFile ~/.ssh/matrixtesting6_ed25519
    IdentitiesOnly=yes
```

2) Edit the variables in `./inventory/hosts/localhost/vars.yml` for this recovery job.

3) Run the playbook:
`$ ansible-playbook -v -i ./inventory/hosts recover_server_1.yml`


- wait for DNS to propogate
- deploy again!



# Issues:

- regenerated digitalocean servers use the same server name!! [fixed]
- how should on-premises servers be recovered?

