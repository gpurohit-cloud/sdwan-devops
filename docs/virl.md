# CML

Both the control plane and the edges can be deployed in CML.

>Note: The repo is designed for VIRL2/CML2.  The previous VIRL1/CML1 version can be found [here](https://github.com/CiscoDevNet/sdwan-devops/tree/virl1)

## Configure the licensing requirements

Set the organization name.

```bash
export VMANAGE_ORG=myorgname
```

Make sure that you copied a valid license file to this location: `licenses/serialFile.viptela` 

## Dependencies

Set the name of the lab, e.g.:

```bash
export VIRL_HOST=myvirlhost
export VIRL_USERNAME=myusername
export VIRL_PASSWORD=mypasword
export VIRL_LAB=myusername_sdwan
```

These values can be set permanently in the `virl.yml` file in the inventory by adding:

```yaml
host: myvirlhost
username: myusername
password: mypasword
lab: myusername_sdwan
```

## Lab files

The files that define the labs are located in `{{ playbook_dir }}/files`.  The `build-virl.yml` playbooks builds the lab specified in the `virl_lab_file` fact located in `inventory/<topology_name>/group_vars/all/virl.yml`

To save changes that you make in the VIRL GUI, download the lab and override the existing file.

## Create a local CA

Create the local CA used for certificate signing.

```bash
./play.sh build-ca.yml
```

## Build the topology

Build out the underlay and control plane.

```bash
./play.sh build-virl.yml
```

## Configure the SD-WAN Control Plane

Configure the SD-WAN control plane using the supplied inventory data.

```bash
./play.sh config-virl.yml
```

## Deploy the SD-WAN edges

Provision and bootstrap the edges.

```bash
./play.sh deploy-virl.yml
```

## Wait for the edges to sync in vManage

The `waitfor-sync.yml` playbook is useful when we want to wait until all edges are in sync and the service VPN is active.  We need to do this before validating the simulation, otherwise the validation check will fail.

```bash
./play.sh waitfor-sync.yml
```

## Get inventory information

```bash
./play.sh virl-inventory.yml
```

## Get detailed inventory information for a single host

```bash
./play.sh virl-inventory.yml --tags=detail --limit=vmanage1
```

## Validate the topology

Run the `check-sdwan.yml` playbook to validate the topology.

```bash
./play.sh check-sdwan.yml
```

## Clean the topology

Stop and wipe all of the nodes in the lab.

```bash
./play.sh clean-virl.yml
```

`--limit` can be used to clean individual nodes:
```bash
./play.sh clean-virl.yml --limit=site1-cedge1
```

To remove the lab completely from the VIRL server:
```bash
./play.sh clean-virl.yml --tags=delete
```
