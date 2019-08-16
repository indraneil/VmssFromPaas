# Porting your existing PAAS service - general concepts
We discovered aspects of your cloud service when running through the checklist exercise in <a href="Chapter1.md">Chapter 1</a>. We then took a detour through the VMSS landscape to get a feel of the new options. Now we are ready to discus strategies to convert your service to the new paradigm.

## Web and worker roles
Since worker roles are just windows server machines, they don't need anything special beyond VM instances being created. Web roles has IIS installed and you will need to explicitly enable that role on top.
* Plan on creating 1 VMSS per role in your service
  * Map your existing [OS choice](https://docs.microsoft.com/en-us/azure/cloud-services/cloud-services-guestos-update-matrix) to an equivalent <a href="https://docs.microsoft.com/en-us/azure/virtual-machines/windows/cli-ps-findimage#table-of-commonly-used-windows-images">OS for VMSS</a>. Just know that unlike earlier, you can now set every role to a different OS choice.
  * If you haven't thought about VM sizes in a while, then you may need to pick newer [sizes](https://docs.microsoft.com/en-us/azure/virtual-machines/windows/sizes).
* To install IIS for web roles, you will likely need to write a custom extension which can enable that role. Here is a <a href="https://github.com/MicrosoftDocs/azure-docs/blob/master/articles/virtual-machines/windows/tutorial-automate-vm-deployment.md">sample</a> from github doing the same thing using powershell.
  * We talked about extensions in <a href="Introduction.md">Introduction</a>. We will delve into custom extensions a bit more soon.
* Your application itself will likely need to be installed on the VMs as a custom extension. We will get to that in a bit as well.

## Enforcing traffic rules for your service
The general patterns for VMSS are to create 
* a VNet that encapsulates the whole service
* a Subnet with associated NSG that encapsulates each VMSS (or role in cloud service)
* A load balancer and public IP address for every **public endpoint** in the cloud service. 
  * So if you have 2 public endpoints in your service paired with 2 roles, create 2 IP addresses, pair them with 2 load balancers, and associate the respective VMSS with each load balancer.
* You can enforce the kind of traffic (UDP/TCP etc.) on the kind of ports (443/8008 etc.) you allow via NSG rules. 
  * **Internal endpoints** can be created using <a href="https://docs.microsoft.com/en-us/azure/virtual-network/security-overview#service-tags">Service tags</a> when creating rules.
  * **Remote desktop** can be similarly enabled. You can even lock down incoming IP addresses using NSG rules. You can also use nontraditional ports (besides 3389) for RDP (by configuring the load balancer)
* If you had specific firewall rules enforced on your VMs, you will need to wrap them as custom extensions

## Handling lifecycle events
* If you have extended [RoleEntryPoint class](https://msdn.microsoft.com/library/azure/microsoft.windowsazure.serviceruntime.roleentrypoint.aspx)
  * [onStart and onStop](https://docs.microsoft.com/en-us/azure/cloud-services/cloud-services-role-lifecycle-dotnet) events can not be handled yet though I am told the support for onStop is likely coming.
  * While worker roles had no choice but to use RoleEntryPoint, if you are using web roles, please consider using the Asp.Net lifecycle events like [Application_Start or Application_End](https://docs.microsoft.com/en-us/azure/cloud-services/cloud-services-role-lifecycle-dotnet). This will making porting your app to VMSS easier. For now, it appears that your code will have to handle "disgraceful" shutdowns on its own.
* If you depend on [RoleEnvironment](https://msdn.microsoft.com/library/azure/microsoft.windowsazure.serviceruntime.roleenvironment.aspx) variables, most of the azure specific ones have been ported to [Azure Instance Metadata Service](https://docs.microsoft.com/en-us/azure/virtual-machines/windows/instance-metadata-service)
  * However, there is no support for user provided values coming from cscfg files. There are a few things you can do to mitigate this. We shall revisit this problem at a later point.
* If you use [startup tasks](https://docs.microsoft.com/en-us/azure/cloud-services/cloud-services-startup-tasks), you should plan to switch over to custom extensions.

## Using external storages with your service
Using external storages with your service will look pretty much the same except for the part where you locate your conection string.
* If you are currently using certificates to authenticate, you can continue to do so. 
* If there are encrypted secrets in your cscfg file, this will need to be handled differently, and we shall revisit this again soon.
* One important new possibility is the use of RBAC (see <a href="Chapter4.md">Chapter 4</a>) to connect to your storages. An example of how to grant identities access to azure storage is [here](https://docs.microsoft.com/en-us/azure/storage/common/storage-auth-aad-rbac-portal?toc=%2fazure%2fstorage%2fblobs%2ftoc.json) and then how your code can leverage that access is [here](https://docs.microsoft.com/en-us/azure/storage/common/storage-auth-aad-app?toc=%2fazure%2fstorage%2fblobs%2ftoc.json)

## Using certificates with your service
If you use certificates, then the most important change is the use of Azure key vaults, which we talked about in <a href="Chapter2.md">Chapter 2</a>. A lot of usage details can be found in the [FAQ]("https://docs.microsoft.com/en-us/azure/virtual-machine-scale-sets/virtual-machine-scale-sets-faq#certificates") for VMSS

The only gotcha is that the key vault and VMSS must be in the same region. (See <a href="https://stackoverflow.com/questions/38856285/scale-set-using-keyvault-in-another-region">this discussion</a>), which means that if you have several VMSS instances in different region, and they share certificates, you will need to create that many key vaults and replicate your secrets to that many regions.

With RBAC (see <a href="Chapter4.md">Chapter 4</a>), there is very little need for management certificates (in fact their use is discouraged), and binding to SSL can be accomplished with custom extensions.

## Deploying and developing the service
* If you use [azure powershell commandlets](https://docs.microsoft.com/en-us/powershell/azure/overview?view=azps-1.1.0) or something similarly imperative-command based to deploy, that is very much an option.
  * If you are ready to switch to declarative deployments (which are safer), please consider ARM. The [quick-start templates](https://github.com/Azure/azure-quickstart-templates) on github are a very convenient way to get started with this
* VMSS has several possible automation options. For a good list, please read [this article](https://docs.microsoft.com/en-us/azure/virtual-machines/windows/infrastructure-automation) which introduces options for continuous integration and delivery (CI/CD) [Azure pipelines](https://docs.microsoft.com/en-us/azure/devops/pipelines/get-started/?view=azure-devops), and other similar option.
* You have several options to upgrade your VMs or your code.
  * [In place upgrade options](https://docs.microsoft.com/en-us/azure/virtual-machine-scale-sets/virtual-machine-scale-sets-upgrade-scale-set)
  * If you prefer VIP swaps, then read [this article](https://msftstack.wordpress.com/2017/02/24/vip-swap-blue-green-deployment-in-azure-resource-manager/) which suggests how to go about this.
* When developing, while the [storage emulator](https://docs.microsoft.com/en-us/azure/storage/common/storage-use-emulator) is still useful, there isn't a corresponding emulator for VMSS.
  * Since the code you are writing is either a traditional ASP.NET app or a windows service, you can mock it using standard techniques. You no longer need explicit Visual Studio templates for cloud services, but those are still technically [available](https://docs.microsoft.com/en-us/visualstudio/azure/vs-azure-tools-azure-project-create?view=vs-2019)

## Custom extensions
We first talked about extensions in <a href="Introduction.md">Introduction</a> and have since suggested custom extensions as a way to solve numerous problems, so it makes sense to explain them in more detail here.

VMSS supports an [extension framework](https://docs.microsoft.com/en-us/azure/virtual-machines/extensions/overview). Any application that supports [unattended-installation](https://en.wikipedia.org/wiki/Installation_(computer_programs)#Unattended_installation) can be wrapped into an extension which can be pushed into VMSS as part of a deployment. You can even arrange for multiple extensions to be installed based on user-defined sequence. You may discover the [available list of extensions](https://docs.microsoft.com/en-us/azure/virtual-machines/extensions/features-windows#discover-vm-extensions) or build your own as a custom extension.

[Custom script extensions](https://docs.microsoft.com/en-us/azure/virtual-machines/extensions/custom-script-windows) allows us to push scripts into VMs to install and configure the VMs to suit our needs. It is a convenient way to do common operations in PAAS with the VMSS framework, e.g. run startup tasks, bind certs to SSL, install and launch your application, etc.
* There is 1 important caveat that holds true in the summer of 2019. Initiating reboots via custom extensions isn't supported.