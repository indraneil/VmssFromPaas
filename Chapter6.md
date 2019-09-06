# Porting your existing PAAS service - specifics
This chapter will talk about some specific topics that should help set up a VMSS cluster and port your existing application.
Think of this chapter as a way to get started on what to build next.

## Start with an ARM template and a test subscription
If possible, start by picking a template from the [quick-start templates](https://github.com/Azure/azure-quickstart-templates) on github. Also designate a test subscription and/or resource group for experimentation. Stick to your testing boundary because you are about to do a bunch of somewhat-insecure things, so be sure you aren't polluting your production services somehow.

## Create a prerequisite script
As part of bootstrapping, you will run a few high privilege commands somewhat infrequently. It is best to script them out so that they can be reliably run with a set of privileges that aren't needed for the main deployment. Some examples are below
#### Create keyvaults and generate certificates
If your service uses certificates, the script can precreate the key vault(s) and generate/rotate certificates in it. Since most certificates can not be generated during a deployment, you will want to run this step ahead of time.

Similarly, if the key vault needs to be bootstrapped with secrets (e.g. the VMSS OS Password) then now would be the time to procure/generate them.
#### Create or update AAD application
If you start using RBAC, you will need to have AAD applications created, client certificates/secrets generated and published to AAD. You should do those in the prereq script as well.

## Modify your build
There are a few new things you should consider doing as part of your build.

### Create a build script that can generate your parameter files
Your ARM templates will need [parameter files](https://docs.microsoft.com/en-us/azure/azure-resource-manager/resource-group-template-deploy#pass-parameter-values) which are analogous to [cscfg files](https://docs.microsoft.com/en-us/azure/cloud-services/cloud-services-model-and-package#serviceconfigurationcscfg) except that these are json files. You could use a templating library (e.g. something like [T4](https://en.wikipedia.org/wiki/Text_Template_Transformation_Toolkit)) to generate different parameter files to target different VMSS deployments.

### Zip your service binaries so that they are deployable via a custom extension
Your build may be outputting a [cspkg file](https://docs.microsoft.com/en-us/azure/cloud-services/cloud-services-model-and-package#servicepackagecspkg) to deploy a service. If you already know how to unpack and use those, that is good, else wise you should create a standalone zip file for VMSS

You can also choose to upload your template and zipped service binaries to some private storage at this point (to be deployed from later).

## Create a runtime library that abstracts the differences between PAAS and VMSS
Your service likely makes a bunch of calls to RoleEnvironment class to discover RoleName/Instance Id/Configuration values etc. In stead of directly calling the RoleEnvironment class, provide a facade that detects Cloud Service or VMSS (for example, maybe look for [metadata service](https://docs.microsoft.com/en-us/azure/virtual-machines/windows/instance-metadata-service) and then abstract the calls by providing your own implementation that you deem suitable for VMSS). You may already be doing something similar for mocking the calls for unit testing, so this isn't that big a leap.

### What to do if you use cscfg files today
There isn't a good answer here, because the cscfg file was used in 2 ways
* Instance information, e.g. role name and count --> The [Metadata service](https://docs.microsoft.com/en-us/azure/virtual-machines/windows/instance-metadata-service) is a drop-in replacement for this
* Application configuration, which there does not seem to have good equivalence in the VMSS world.

To handle the application specific things, 
* One option here is to upload the exisiting .cscfg file into the VMs, and let this runtime library intercept calls to GetConfigurationSettingValue and read it off this local file.
* Similarly, if you use local resources, then you can get your library to intercept calls to GetLocalResourceRootPath and dole out directories from some well known location in the VM.

One interesting thing is that if you are looking for instance Id, you should read [this page](https://docs.microsoft.com/en-us/azure/virtual-machine-scale-sets/virtual-machine-scale-sets-instance-ids#scale-set-vm-computer-name) since VMSS uses hexatrigesimal (base36) numbering for VMs.

## Create an entrypoint script to be invoked as a custom extension
As part of the deployment, your zipped up service, related configuration, and this entry point script will be pushed into the azure VMs. 
* The entry point script will then be invoked as a custom extension, at which point, plan on unzipping your service and launching it passing it the configuration.
* Also remember to place your configuration at a place that is accessible to your runtime library

### Create a hosting/launching process for your DLL
Cloud service uses WaWorkerHost.exe or WaIISHost.exe to host and launch your DLLs, and you now need something new to do it for you. For your web roles, IIS can handle this, if you are installing that ([sample code](https://github.com/MicrosoftDocs/azure-docs/blob/master/articles/virtual-machines/windows/tutorial-automate-vm-deployment.md)). However you will need something similar for worker roles. Whatever you build, you should configure it such that it is relaunched if the VM reboots (note that custom extensions don't run on reboots).

## Decide on how you are deploying your build
You should have a formal way to deploy your official build. 
* It can be as simple as a [script that deploys your templates](https://docs.microsoft.com/en-us/azure/azure-resource-manager/resource-group-template-deploy) with AAD credentials.
* It can be something a lot more complicated like a [CI/CD Azure pipeline](https://docs.microsoft.com/en-us/azure/devops/pipelines/get-started/what-is-azure-pipelines?view=azure-devops) posting updates to [teams channels](https://docs.microsoft.com/en-us/azure/devops/pipelines/integrations/microsoft-teams?view=azure-devops)

## Create some helpful debugging scripts
If your service has exisiting logging/telemetry that should keep working after being deployed here, you should plan on building some new debugging scripts i.e.
* A script that can enumerate the extension status for each scale set in your resource group
  * If a particular extension has failed, it can drill into the instances and enumerate them and their status
* A script that can enumerate each scale set in your resource group and create an RDG file for you to remote into using [rdcman](https://www.microsoft.com/en-us/download/details.aspx?id=44989)
  * It should be able to get the VMSS password from the key vault created as part of prereq script

> If you have gotten this far, you are well on your way to successfully transition to VMSS. Congratulations!

**Navigation**: <a href="Introduction.md">First<a> | <a href="Chapter5.md">Prev</a> | Next | Last
