Deploy new MySQL database using our playbooks
=============================================

This ansible repository deploys mysql server on OpenStack provider.

Create an OpenStack environment
-------------------------------

1. Create a volume for data, as big as you need. It should be empty.
2. Create a security group for database servers. It should have following rules:
    1. Incoming traffic is allowed only for port 22.
    2. Outgoing traffic should be allowed.
    3. When you provision servers using the database allow them using their IP.
3. Create an instance using an appropriate image, such as Ubuntu 16.04. We suggest you use vanilla image from
   [ubuntu page](https://cloud-images.ubuntu.com/). This image might be small; MySQL
   data will be stored elsewhere.
4. Attach the data volume that you created earlier to the VM (go to `Volumes` -> data volume -> expand dropdown next to "Edit Volume" ->
   `Edit Attachments`).
5. Manual step: SSH into the database instance and format `/dev/vdb` using `ext4`.
   Run `fdisk /dev/vdb` to do this and then issue:
    1. `o` --- creates new DOS partition table
    2. `n p` --- creates primary partition, and then `1 <enter> <enter>` will create partition in slot 1 that will span the whole disk.
    3. `w` --- writes changes to the disk
    4. Create the file system: `mkfs.ext4 -j /dev/vdb1`
6. Go to info page and note the instance public key, ssh to the instance and check whether public key matches,
   save public key to your ssh config.

Generate secrets
----------------

1. Create `private-extra-vars.yml`, you'll put all the generated variables there
1. Generate root password, put it in the `private.yaml` for your host, under `MYSQL_ROOT_PASSWORD`.
2. Create a key, but store it somewhere safe (for example keepassx database), this key shouldn't end on the mysql server.
3. Generate tarsnap read write key from the master key, see [tarsnap-keymngmt](http://www.tarsnap.com/man-tarsnap-keymgmt.1.html),
   save this key in `MYSQL_TARSNAP_KEY`. **Note**: this key won't be able to delete the backups.
4. Go to dead man's snitch at https://deadmanssnitch.com/. Save it under `TARSNAP_BACKUP_SNITCH` in the `private.yml`.

Perform deployment
------------------

    ansible-galaxy install -r requirements.yml -f && ansible-playbook deploy-all.yml -u ubuntu --extra-vars @private-extra-vars.yml

Post deployment checkups
------------------------

1. Check that backups are saved to tarsnap
2. Check contents of these backups.
