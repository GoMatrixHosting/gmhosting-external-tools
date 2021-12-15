
# Recovers a Matrix Server

Used to recover a Matrix server connected to AWX. This tool should be run outside of AWX.


## Instructions:

1) Edit the variables in `./inventory/hosts/localhost/vars.yml` for this recovery job.

2) List borg backups for that server:
`$ ansible-playbook -v -i ./inventory/hosts --extra-vars "view_borg_backup=true" recover_matrix_server.yml`

3) Observe the output, if you don't want the latest backup to be restored then copy the lines you want to recover:

```
TASK [recover-matrix-server : Print borg backup history for /matrix] ***********************************************************************************************************
Sunday 21 November 2021  09:32:19 +0800 (0:00:00.775)       0:00:14.154 ******* 
ok: [localhost] => {
    "msg": [
        "T-6HAX1LZIJHX9-aveng-2021-11-19T02:08:20 Fri, 2021-11-19 10:08:24 [b4a5ed078e228d3ad0c24458305e3dba49e38752418db46e9c18d66875266348]",
        "T-6HAX1LZIJHX9-aveng-2021-11-20T10:23:46 Sat, 2021-11-20 18:23:51 [01075b0015a7c1aff02e511fdacabc7d4ed73d4cd7e0f7d61788cc405257733b]",
        "T-6HAX1LZIJHX9-aveng-2021-11-21T00:02:25 Sun, 2021-11-21 08:02:30 [8b02b290700a419acca73b696808606eef37691272a54c5e1d0b93bd070a902e]"
    ]
}

TASK [recover-matrix-server : Print borg backup history for /database] *********************************************************************************************************
Sunday 21 November 2021  09:32:19 +0800 (0:00:00.037)       0:00:14.191 ******* 
ok: [localhost] => {
    "msg": [
        "T-6HAX1LZIJHX9-aveng-2021-11-19T02:08:51 Fri, 2021-11-19 10:08:56 [056cc8a0a8d68b182cce2496ebecae2ea31763ae9411877f3bc1993b806e2325]",
        "T-6HAX1LZIJHX9-aveng-2021-11-20T10:24:12 Sat, 2021-11-20 18:24:17 [9d2ffcb1c878b60619d6871c72ee04542077a6c31232ccd0e91300705e607bb7]",
        "T-6HAX1LZIJHX9-aveng-2021-11-21T00:03:03 Sun, 2021-11-21 08:03:08 [6b7138917fa4b4071e28fbfe6d77e5cb9ba860633e19341e3eda3e8ccaa5492c]"
    ]
}
```


4A) Run the playbook and recover the latest backup (made after the servers shutdown by this playbook):

Need to include:
```
matrix_domain
new_plan_title <see below>
AND
new_do_droplet_region_long <see below>
OR
new_server_ipv4
new_server_ipv6
```
Optionally you can change the member_id by specifying it:
```
member_id
```
Possible new_plan_title titles:
```
Micro DigitalOcean Server
Small DigitalOcean Server
Medium DigitalOcean Server
Large DigitalOcean Server
Jumbo 500 DigitalOcean Server
Jumbo 1000 DigitalOcean Server
Jumbo 2000 DigitalOcean Server
Jumbo 5000 DigitalOcean Server
Micro On-Premises Server
Small On-Premises Server
Medium On-Premises Server
Large On-Premises Server
Jumbo 500 On-Premises Server
Jumbo 1000 On-Premises Server
Jumbo 2000 On-Premises Server
Jumbo 5000 On-Premises Server
```
Possible new_do_droplet_region_long values:
```
New York City (USA)
San Francisco (USA)
Amsterdam (NLD)
Frankfurt (DEU)
Singapore (SGP)
London (GBR)
Toronto (CAN)
Balgalore (IND)
```

Example, recover to DO:
```
$ ansible-playbook -v -i ./inventory/hosts \
--extra-vars 'matrix_domain="aveng.xyz" \
new_plan_title="Micro DigitalOcean Server" \
new_do_droplet_region_long="New York City (USA)"' \
recover_matrix_server.yml
```

Example, recover to OP and change member_id:
```
ansible-playbook -v -i ./inventory/hosts \
--extra-vars 'matrix_domain="aveng.xyz" \
member_id="65" \
new_plan_title="Micro On-Premises Server" \
new_server_ipv4="188.166.223.116"' \
recover_matrix_server.yml
```

4B) Run the playbook and recover a specific backup:
```
$ ansible-playbook -v -i ./inventory/hosts \
--extra-vars 'matrix_domain="aveng.xyz" \
new_plan_title="Small DigitalOcean Server" \
new_do_droplet_region_long="New York City (USA)"' \
borg_backup_matrix_input="T-6HAX1LZIJHX9-aveng-2021-11-20T10:23:46 Sat, 2021-11-20 18:23:51 [01075b0015a7c1aff02e511fdacabc7d4ed73d4cd7e0f7d61788cc405257733b]" \
borg_backup_database_input="T-6HAX1LZIJHX9-aveng-2021-11-20T10:24:12 Sat, 2021-11-20 18:24:17 [9d2ffcb1c878b60619d6871c72ee04542077a6c31232ccd0e91300705e607bb7]"' \
recover_matrix_server.yml
```


5) Alter the DNS record and wait for it to propagate.


6A) Run the playbook and raise and test the new server, then finally delete the old server and subscription:
`$ ansible-playbook -v -i ./inventory/hosts --extra-vars 'matrix_domain="aveng.xyz"' raise_matrix_server.yml`

6B) If you're expecting the self-check to fail add this extra variable to skip it (dangerous):
`$ ansible-playbook -v -i ./inventory/hosts --extra-vars 'matrix_domain="aveng.xyz" skip_self_check="true"' raise_matrix_server.yml`


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
