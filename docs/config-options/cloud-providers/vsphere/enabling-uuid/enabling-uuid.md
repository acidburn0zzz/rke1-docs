---
title: Enabling Disk UUIDs for vSphere VMs
---

In order to provision nodes with RKE, all nodes must be configured with disk UUIDs. This is required so that attached VMDKs present a consistent UUID to the VM, allowing the disk to be mounted properly.

Depending on whether you are provisioning the VMs using the [vSphere node driver](https://ranchermanager.docs.rancher.com/pages-for-subheaders/launch-kubernetes-with-ranchernode-pools/vsphere) in Rancher or using your own scripts or third-party tools, there are different methods available to enable disk UUIDs for VMs:

- [Using the vSphere console](#using-the-vsphere-console)
- [Using the GOVC CLI tool](#using-the-govc-cli-tool)
- [Using a Rancher node template](#using-a-rancher-node-template)

### Using the vSphere Console

The required property can be set while creating or modifying VMs in the vSphere Console:

1. For each VM navigate to the tab **VM Options** and click on **Edit Configuration**.
2. Add the parameter `disk.EnableUUID` with a value of **TRUE**.

![vSphere Advanced Parameters](/img/vsphere-advanced-parameters.png)

### Using the GOVC CLI tool

You can also modify properties of VMs with the [govc](https://github.com/vmware/govmomi/tree/master/govc) command-line tool to enable disk UUIDs:

```sh
$ govc vm.change -vm <vm-path> -e disk.enableUUID=TRUE
```

### Using a Rancher Node Template

In Rancher v2.0.4+, disk UUIDs are enabled in vSphere node templates by default.

If you are using Rancher before v2.0.4, refer to the [vSphere node template documentation.](https://ranchermanager.docs.rancher.com/v2.0-v2.4/reference-guides/cluster-configuration/downstream-cluster-configuration/node-template-configuration/vsphere/prior-to-v2.0.4) for details on how to enable a UUID with a Rancher node template.
