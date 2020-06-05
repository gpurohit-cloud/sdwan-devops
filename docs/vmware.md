# Building the SD-WAN Control Plane on VMware

The steps below will build and configure the SD-WAN control plane and edges on VMware.

> Note: the only tested reference topology for VMware is the hq2 topology.

## Create the SD-WAN images

The VMware playbooks make use of terraform to provision the control plane and edge VNFs.  Prior to running the playbooks, ensure you have the correct SD-WAN images deployed to your VMware cluster.  Follow the directions for creating the VMware images in the [terraform-sdwan](https://github.com/CiscoDevNet/terraform-sdwan) repo.

> Note: You only need to create the images.  You do not need to do any of the other steps outlined in the terraform-sdwan repo for manually provisioning using terraform.  The playbooks will run terraform automatically using the data specified in Ansible inventory.

## Configure the licensing requirements

Set the organization name.

```bash
export VMANAGE_ORG=myorgname
```

Make sure that you copied a valid license file to this location: `licenses/serialFile.viptela` 

## Create the local CA

Create the local CA used for certificate signing.

```bash
./play.sh build-ca.yml
```

## Build the SDWAN control plane

Build the control plane on VMWare.

```bash
./play.sh build-vmware.yml
```

> Note: this playbook can take a long time to run.  You can verify activity by using the vCenter UI to verify provisioning and watch the boot process.

## Create/update the inventory data

In `groupvars/all/local.yml` set the following variables to reflect your environment:
* `vsphere_user`: the vCenter username
* `vsphere_password`: the vCenter password
* `vsphere_server`: the vCenter FQDN or IP address
* `datacenter`: the vCenter datacenter to use
* `cluster`: the vCenter cluster to use
* `folder`: the folder to create in vCenter.  Provisioned VMs will be placed in this folder.
* `resource_pool`: the vCenter resource pool to use.  Leave blank if none.
* `datastore`: the datastore to use for VMs
* `iso_datastore`: the datastore to use for cloud-init ISOs
* `iso_path`: the path on the datastore for cloud-init ISOs
* `vmanage_template`: the name of the vManage template in vCenter
* `vbond_template`: the name of the vBond template in vCenter
* `vsmart_template`: the name of the vSmart template in vCenter
* `vedge_template`: the name of the vEdge template in vCenter
* `cedge_template`: the name of the cEdge template in vCenter

In `inventory/hq2/sdwan.yml` set the following variables to reflect your environment:

* `sdwan_vbond`: DNS name/IP address of the vbond server
* `vpn0_portgroup`: the name of the vCenter port group to use for vpn0
* `vpn0_gateway`: the default gateway to use for the vpn0 network
* `vpn512_portgroup`: the name of the vCenter port group to use for vpn512
* `vpn0_ip`: for each control plane component, set this to it's assigned static IP address
* `servicevpn_portgroup`: the name of the vCenter port group to use for the service VPN (VPN 1)

## Configure the SD-WAN Control Plane

Configure the SD-WAN control plane using the supplied inventory data.

```bash
./play.sh config-vmware.yml
```

> Note: sometimes this playbook will fail because an incorrect IP address was detected for vBond.  If this happens, simply re-run the `build-vmware.yml` playbook as shown above.  This appears to be a bug in the way vBond reports IP addressing to VMware.

## Deploy the SD-WAN edges

Provision and bootstrap the edges.

```bash
./play.sh deploy-vmware.yml
```

> Note: this playbook can take a long time to run.  You can verify activity by using the vCenter UI to verify provisioning and watch the boot process.

## Wait for the edges to sync in vManage

The `waitfor-sync.yml` playbook is useful when we want to wait until all edges are in sync and the service VPN is active.  We need to do this before validating the simulation, otherwise the validation check will fail.

```bash
./play.sh waitfor-sync.yml
```

## Validate the topology

Run the `check-sdwan.yml` playbook to validate the topology.

```bash
./play.sh check-sdwan.yml
```

## Clean the topology

Delete the entire simulation, including the control plane and edges.

```bash
./play.sh clean-vmware.yml
```

If you want to delete only the edges, use the `edges` tag.

```bash
./play.sh clean-vwmare.yml --tags="edges"
```

If you want to delete only the control plane, use the `control` tag.

```bash
./play.sh clean-vmware.yml --tags="control"
```
