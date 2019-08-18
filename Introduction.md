# Moving from Azure cloud services to Azure VMSS
Running a service on azure cloud has been synonymous with the [PAAS](https://en.wikipedia.org/wiki/Platform_as_a_service) offering called [CloudService](https://docs.microsoft.com/en-us/azure/cloud-services/cloud-services-choose-me) for a long while.
While newer offerings like [Webjobs](https://docs.microsoft.com/en-us/azure/app-service/webjobs-create) and [Functions](https://docs.microsoft.com/en-us/azure/azure-functions/functions-overview) showed up, people who needed control of the VM hosting the services, had no choice but to stick to CloudServices.

However, a few years ago, Azure unveiled [Virtual Machine Scale Sets](https://docs.microsoft.com/en-us/azure/virtual-machine-scale-sets/overview) (VMSS) and it became a contender to CloudService, so it is worth understanding if you should switch and how would you go about it.

## What is VMSS?
VMSS is an [IAAS](https://en.wikipedia.org/wiki/Infrastructure_as_a_service) play from Microsoft and allows VM operations (create/restart etc.) to be applied at a full set of VMs (upto 1000 right now) with 1 command.

* Unlike cloud services, you can create [Linux based](https://docs.microsoft.com/en-us/azure/virtual-machines/linux/tutorial-create-vmss) VMSS.
* You can also create [Service Fabric](https://docs.microsoft.com/en-us/azure/service-fabric/service-fabric-overview) or [Kubernetes](https://docs.microsoft.com/en-us/azure/aks/intro-kubernetes) clusters and deploy your applications to those, in stead of as direct PAAS.
* You get better control on the networking with [VNets](https://docs.microsoft.com/en-us/azure/virtual-network/virtual-networks-overview), [NSGs](https://docs.microsoft.com/en-us/azure/virtual-network/security-overview), [load balancers](https://docs.microsoft.com/en-us/azure/load-balancer/), [IP addresses](https://docs.microsoft.com/en-us/azure/virtual-network/virtual-network-public-ip-address) etc. being supported as first class objects
* You can configure the VMs in numerous ways using [extensions](https://docs.microsoft.com/en-us/azure/virtual-machines/extensions/features-windows) like PowerShell Desired State configurations ([DSC](https://docs.microsoft.com/en-us/azure/virtual-machines/extensions/dsc-overview)) or [Bitlocker](https://docs.microsoft.com/en-us/azure/security/azure-security-disk-encryption-overview)
* Like cloud services, you can get regular [OS upgrades](https://docs.microsoft.com/en-us/azure/virtual-machine-scale-sets/virtual-machine-scale-sets-automatic-upgrade)
* However, at the end of it all, you just get a set of VMs, and you decide how to deploy and run your application on it. It neither blocks you from running anything, nor suggests a specific way to run something.

All things considered, if you decide you need more control - and not less - VMSS is a good choice.

## What to consider when porting a service from cloud service to VMSS
Cloud services were an opinionated implementation of how to design a service. You had to code to a certain paradigm and in exchange, a lot of details were abstracted from you.
In VMSS, you get to decide on all those aspects, so you will have to reverse engineer some of the decisions from cloud services.

In broad terms, you have to decide
* On the kind of network boundary that will replace your existing setup
* What new features you would want to use
* How to replace the [RoleEnvironment](https://docs.microsoft.com/en-us/previous-versions/azure/reference/ee773173(v=azure.100)) class
* How to pull in certificates
* How you would want to deploy

Once you have done that all, you will have your service running on VMSS and a world of customization-options open up for you from there on

## How this booklet is laid out
* Introduction: You are here right now
* <a href="Chapter1.md">Analyse your existing PAAS cloud service</a>
* <a href="Chapter2.md">Concepts about creating a sample VMSS</a>
* <a href="Chapter3.md">Concepts about exploring a running VMSS</a>
* <a href="Chapter4.md">Understanding ARM and RBAC, which didn't exist when cloud services were created</a>
* <a href="Chapter5.md">General concepts to consider when porting your PAAS application</a>
* <a href="Chapter6.md">Specific things to build when porting your PAAS application</a>

**Navigation**: First | Prev | <a href="Chapter1.md">Next</a> | <a href="Chapter6.md">Last</a>
