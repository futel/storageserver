## Requirements

- Have assets subdirectory populated with assets.
- Don't have any volumes named assetsbuild.
- Don't have any droplets named assetsbuild.phu73l.net.

## Validate requirements and configuration locally

    ansible-playbook -i deploy/hosts deploy/requirements_conf_prod_playbook.yml

## Build volume

    ansible-playbook -i deploy/hosts deploy/deploy_digitalocean_playbook.yml --vault-password-file=conf/vault_pass_digitalocean.txt

- Note IP address printed.
- Wait for DNS to match IP address with "nslookup assetsbuild.phu73l.net".

    ansible-playbook -i deploy/hosts deploy/secure_playbook.yml
    ansible-playbook -i deploy/hosts deploy/deploy_playbook.yml --vault-password-file=conf/vault_pass_digitalocean.txt
    ansible-playbook -i deploy/hosts deploy/provision_playbook.yml

XXX from here

    packer build -var-file=conf/variables.pkr.hcl deploy/deploy_template.pkr.hcl

## Create snapshot of volume and clean up

- In digitalocean console, create snapshot of assetsbuild volume with default name.
- In digitalocean console, destroy assetsbuild volume.
- In digitalocean console, destroy all assetsbuild volume snapshots but most recent.
- In digitalocean console, destroy assetsbuild.phu73l.net droplet.

## Notes

community.digitalocean.digital_ocean_snapshot is the ansible collection to snapshot volume

volume snapshot downgrade:
- check out appropriate release to match asteriskserver
- create volume snapshot

asteriskserver stage deploy:
- create, provision, etc droplet
- list assetsbuild volume snapshots, find most recent
- create, mount assets volume from assetsbuild snapshot

asterisksever stage promote:
- decommission, delete etc futel-prod-back droplet
- destroy assets volume mounted to futel-prod-back
- delete all assetsbuild volume snapshots but most recent

asteriskserver prod downgrade:
- make snapshot of current futel-prod droplet
- delete futel-prod droplet
- create droplet from futel-prod-back
- XXX create, mount volume of appropriate revision



# Create a mount point for your volume:
$ mkdir -p /mnt/assets_tmp

# Mount your volume at the newly-created mount point:
$ mount -o discard,defaults,noatime /dev/disk/by-id/scsi-0DO_Volume_assets-tmp /mnt/assets_tmp

# Change fstab so the volume will be mounted after a reboot
$ echo '/dev/disk/by-id/scsi-0DO_Volume_assets-tmp /mnt/assets_tmp ext4 defaults,nofail,discard 0 0' | sudo tee -a /etc/fstab
