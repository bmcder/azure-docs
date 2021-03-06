---
title: Data Science Virtual Machine pools - Azure | Microsoft Docs
description: Deploying pools of Data Science VMs as a shared resource for a team
keywords: deep learning, AI, data science tools, data science virtual machine, geospatial analytics, team data science process
services: machine-learning
documentationcenter: ''
author: gopitk
manager: cgronlun
ms.custom: seodec18

ms.assetid: 
ms.service: machine-learning
ms.subservice: data-science-vm
ms.workload: data-services
ms.tgt_pltfrm: na
ms.devlang: na
ms.topic: article
ms.date: 12/10/2018
ms.author: gokuma

---

# Create a shared pool of Data Science Virtual Machines

This article discusses how you can create a shared pool of Data Science Virtual Machines (DSVMs) for a team to use. The benefits of using a shared pool are better resource utilization, facilitation of sharing and collaboration, and more effective management of DSVM resources. 

You can use many methods and technologies to create a pool of DSVMs. This article focuses on pools for interactive VMs. An alternative managed compute infrastructure is Azure Machine Learning Compute. See [Set up compute targets](../service/how-to-set-up-training-targets.md#amlcompute) for more information.

## Interactive VM pool

A pool of interactive VMs that are shared by the whole AI/data science team allows users to log in to an available instance of the DSVM instead of having a dedicated instance for each set of users. This setup helps with better availability and more effective utilization of resources. 

The technology that you use to create an interactive VM pool is [Azure virtual machine scale sets](https://docs.microsoft.com/azure/virtual-machine-scale-sets/). You can use scale sets to create and manage a group of identical, load-balanced, and autoscaling VMs. 

The user logs in to the main pool's IP or DNS address. The scale set automatically routes the session to an available DSVM in the scale set. Because users want a similar environment regardless of the VM they're logging in to, all instances of the VM in the scale set mount a shared network drive, like an Azure Files share or an NFS share. The user's shared workspace is normally kept on the shared file store that's mounted on each of the instances. 

You can find a sample Azure Resource Manager template that creates a scale set with Ubuntu DSVM instances on [GitHub](https://raw.githubusercontent.com/Azure/DataScienceVM/master/Scripts/CreateDSVM/Ubuntu/dsvm-vmss-cluster.json). A sample of the [parameter file](https://raw.githubusercontent.com/Azure/DataScienceVM/master/Scripts/CreateDSVM/Ubuntu/dsvm-vmss-cluster.parameters.json)  for the Azure Resource Manager template is in the same location. 

You can create the scale set from the Azure Resource Manager template by specifying values for the parameter file in Azure CLI. 

```
az group create --name [[NAME OF RESOURCE GROUP]] --location [[ Data center. For eg: "West US 2"]
az group deployment create --resource-group  [[NAME OF RESOURCE GROUP ABOVE]]  --template-uri https://raw.githubusercontent.com/Azure/DataScienceVM/master/Scripts/CreateDSVM/Ubuntu/dsvm-vmss-cluster.json --parameters @[[PARAMETER JSON FILE]]
```
The preceding commands assume you have:
* A copy of the parameter file with the values specified for your instance of the scale set.
* The number of VM instances.
* Pointers to the Azure Files share.
* Credentials for the storage account that will be mounted on each VM. 

The parameter file is referenced locally in the commands. You can also pass parameters inline or prompt for them in your script.  

The preceding template enables the SSH and the JupyterHub port from the front-end scale set to the back-end pool of Ubuntu DSVMs. As a user, you just log in to the VM on SSH or on JupyterHub in the normal way. Because the VM instances can be scaled up or down dynamically, any state needs to be saved in the mounted Azure Files share. You can use the same approach to create a pool of Windows DSVMs. 

The [script that mounts the Azure Files share](https://raw.githubusercontent.com/Azure/DataScienceVM/master/Extensions/General/mountazurefiles.sh) is also available in the Azure DataScienceVM repository in GitHub. The script mounts the Azure Files share at the specified mount point in the parameter file. The script also creates soft links to the mounted drive in the initial user's home directory. A user-specific notebook directory within the Azure Files share is soft linked to the `$HOME/notebooks/remote` directory so that users can access, run, and save their Jupyter notebooks. You can use the same convention when you create additional users on the VM to point each user's Jupyter workspace to the Azure Files share. 

Virtual machine scale sets support autoscaling. You can set rules on when to create additional instances and when to scale down instances. For example, you can scale down to zero instances to save on cloud hardware usage costs when the VMs are not used at all. The documentation pages of virtual machine scale sets provide detailed steps for [autoscaling](https://docs.microsoft.com/azure/virtual-machine-scale-sets/virtual-machine-scale-sets-autoscale-overview).

## Next steps

* [Set up a common Identity](dsvm-common-identity.md)
* [Securely store credentials to access cloud resources](dsvm-secure-access-keys.md)















