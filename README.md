# talos-proxmox-cluster

[![Terraform](https://github.com/Naman1997/talos-proxmox-cluster/actions/workflows/terraform.yml/badge.svg)](https://github.com/Naman1997/talos-proxmox-cluster/actions/workflows/terraform.yml)
[![GitHub license](https://img.shields.io/github/license/Naereen/StrapDown.js.svg)](https://github.com/Naman1997/talos-proxmox-cluster/blob/main/LICENSE)

Automated talos cluster with system extensions

## Dependencies

| Dependency | Location |
| ------ | ------ |
| [Proxmox](https://www.proxmox.com/en/proxmox-ve) | Proxmox node |
| [xz](https://en.wikipedia.org/wiki/XZ_Utils) | Proxmox node |
| [jq](https://stedolan.github.io/jq/) | Client |
| [arp-scan](https://linux.die.net/man/1/arp-scan) | Client |
| [talosctl](https://www.talos.dev/latest/learn-more/talosctl/) | Client |
| [Terraform](https://www.terraform.io/) | Client |
| [HAproxy](http://www.haproxy.org/) | Raspberry Pi |
| [Docker](https://docs.docker.com/) | Client |

`Client` refers to the node that will be executing `terraform apply` to create the cluster. The `Raspberry Pi` can be replaced with a VM or a LXC container.

Docker is mandatory on the `Client` as this projects builds a custom talos image with system extensions using the [imager](https://github.com/siderolabs/talos/pkgs/container/installer) docker image on the `Client` itself.

## Create an HA Proxy Server

You can use the [no-lb](https://github.com/Naman1997/simple-talos-cluster/tree/no-lb) branch in case you do not want to use an external load-balancer. This branch uses the 1st master node that gets created as the cluster endpoint.

I've installed `haproxy` on my Raspberry Pi. You can choose to do the same in a LXC container or a VM.

You need to have passwordless SSH access to a user (from the Client node) in this node which has the permissions to modify the file `/etc/haproxy/haproxy.cfg` and permissions to run `sudo systemctl restart haproxy`. An example is covered in this [doc](docs/HA_Proxy.md).


## Create the terraform.tfvars file

The variables needed to configure this script are documented in this [doc](docs/Variables.md).

```
cp terraform.tfvars.example terraform.tfvars
# Edit and save the variables according to your liking
vim terraform.tfvars
```


## Creating the cluster

```
terraform init -upgrade
terraform plan
terraform apply --auto-approve
```

## Expose your cluster to the internet (Optional)

It is possible to expose your cluster to the internet over a small vps even if both your vps and your public ips are dynamic. This is possible by setting up dynamic dns for both your internal network and the vps using something like duckdns
and a docker container to regularly monitor the IP addresses on both ends. A connection can be then made using wireguard to traverse the network between these 2 nodes. This way you can hide your public IP while exposing services to the internet.

Project Link: [wireguard-k8s-lb](https://github.com/Naman1997/wireguard-k8s-lb)


## Known Issue(s)

### Proxmox in KVM

Currently this only happens if you're running this inside on a proxmox node that itself is virtualized inside kvm. This is highly unlikely, but I'll make a note of this for anyone stuck on this.

This project uses `arp-scan` to scan the local network using arp requests. In case your user does not have proper permissions to scan using the `virbr0` interface, then the talos VMs will not be found.

To fix this, either you can update the permissions for that socket interface or you can just use `sudo`, in case you opt for solution 2, make sure to run the `talosctl kubeconfig` command generated for you in `talos_setup.sh` after `terraform apply` finishes.