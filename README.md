# Build assets storage on Digital Ocean

## Steps

- Validate, etc.
- Update asset source
- Create asset package, droplet, volume
- Create volume snapshot
- Clean up everything but volume snapshot

## Requirements

- Don't have any volumes named assetsbuild.
- Don't have any droplets named assetsbuild.phu73l.net.

## Validate requirements and configuration locally

    ansible-playbook -i deploy/hosts deploy/requirements_conf_playbook.yml
    ansible-playbook -K -i deploy/hosts deploy/requirements_packages_playbook.yml    

## Update asset source

Checkout the content repository into the assets subdirectory, or update it to the latest revision.

https://gitlab.com/futel/futel-content

## Create asset package, droplet, volume

    ansible-playbook -i deploy/hosts deploy/update_assets_playbook.yml
    ansible-playbook -i deploy/hosts deploy/deploy_digitalocean_playbook.yml --vault-password-file=conf/vault_pass_digitalocean.txt

- Note IP address printed.
- Wait for DNS to match IP address with "nslookup assetsbuild.phu73l.net".

    ansible-playbook -i deploy/hosts deploy/secure_playbook.yml
    ansible-playbook -i deploy/hosts deploy/deploy_playbook.yml --vault-password-file=conf/vault_pass_digitalocean.txt
    ansible-playbook -i deploy/hosts deploy/provision_playbook.yml

## Create snapshot of volume and clean up

- In digitalocean console, create snapshot of assetsbuild volume with default name.
- In digitalocean console, destroy assetsbuild.phu73l.net droplet and the associated assetsbuild volume.
- In digitalocean console, destroy old assetsbuild.phu73l.net domain name.
- In digitalocean console, destroy old assetsbuild volume snapshots.
  - Keep the most recent (which we just created), we will attach it to the next stage/prod.
  - Keep the most recent before that as a backup.


## Notes

community.digitalocean.digital_ocean_snapshot is the ansible collection to snapshot volume

volume snapshot downgrade:
- check out appropriate release to match asteriskserver
- create volume snapshot

asteriskserver prod downgrade:
- make snapshot of current futel-prod droplet
- delete futel-prod droplet
- create droplet from futel-prod-back
- XXX create, mount volume of appropriate revision
