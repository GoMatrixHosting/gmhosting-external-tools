
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

# Starting again:

To start from the beginning:
1) move the backup back in place:
```
$ ssh backup.topgunmatrix.com 
...
root@backup:~# mv /mnt/mfs/GMH-Backups/Clients/gomatrixhostingisawesome.xyz-old/ /mnt/mfs/GMH-Backups/Clients/gomatrixhostingisawesome.xyz
```
2) delete temp files on controller:
```
$ ls /tmp/gomatrixhostingisawesome.xyz*
/tmp/gomatrixhostingisawesome.xyz-borg_backup.yml  /tmp/gomatrixhostingisawesome.xyz_lock_3
/tmp/gomatrixhostingisawesome.xyz-extra_vars.json  /tmp/gomatrixhostingisawesome.xyz-matrix_vars.yml
/tmp/gomatrixhostingisawesome.xyz_lock_1           /tmp/gomatrixhostingisawesome.xyz-organisation.yml
/tmp/gomatrixhostingisawesome.xyz_lock_2           /tmp/gomatrixhostingisawesome.xyz-server_vars.yml
$ rm /tmp/gomatrixhostingisawesome.xyz*
```
3) delete the domains inventory file:
```
/recover-matrix-server$ rm ./inventory/gomatrixhostingisawesome.xyz.yml
```


# Issues:

- regenerated digitalocean servers use the same server name!! [fixed]
- how should on-premises servers be recovered?

