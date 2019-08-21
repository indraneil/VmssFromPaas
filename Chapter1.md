# Analyse your existing PAAS cloud service
To recreate the cloud service in VMSS, you need to have a good sense of what you are building. So start by analysing what you currently do. Once you have a checklist of those things, you have a good sense of what you are about to emulate in the VMSS world.

## Web and worker roles
The difference between the 2 types of roles in cloud service is the lack of [IIS being enabled](https://stackoverflow.com/questions/7118942/in-windows-azure-what-are-web-role-worker-role-and-vm-role) in Worker role. However, you should enumerate how many of each role types your service has (e.g. you may have a cloud service with 3 worker role types). Also check the following related information
* Discover the [size](https://docs.microsoft.com/en-us/azure/cloud-services/cloud-services-sizes-specs) and count of VMs per role
* Confirm the [OS family](https://docs.microsoft.com/en-us/azure/cloud-services/cloud-services-guestos-update-matrix) being used.

## Discover traffic rules for your service
* Find out the number of public and internal [endpoints](https://docs.microsoft.com/en-us/azure/cloud-services/cloud-services-enable-communication-role-instances) in your service.
* Confirm the protocol that is used for those ports (TCP/UDP/HTTPS etc.)
* Are you enforcing communication patterns inside your service between the roles?
* Do you have [remote desktop](https://docs.microsoft.com/en-us/azure/cloud-services/cloud-services-role-enable-remote-desktop-visual-studio) enabled for your service?
* Do you use an Azure [Traffic Manager](https://docs.microsoft.com/en-us/azure/traffic-manager/traffic-manager-overview)?
* Do you use [reserved IPs](https://azure.microsoft.com/en-us/blog/reserved-ip-addresses/)?

## What lifecycle events are you handling today?
* Do you have handlers for [onStart and onStop](https://docs.microsoft.com/en-us/azure/cloud-services/cloud-services-role-lifecycle-dotnet) events?
* What have you done to extend [RoleEntryPoint class](https://msdn.microsoft.com/library/azure/microsoft.windowsazure.serviceruntime.roleentrypoint.aspx)
* Do you use [startup tasks](https://docs.microsoft.com/en-us/azure/cloud-services/cloud-services-startup-tasks)? 
* Are you depending on [RoleEnvironment](https://msdn.microsoft.com/library/azure/microsoft.windowsazure.serviceruntime.roleenvironment.aspx) variables?

## How are storages being accessed?
* Do you use external storages and/or caches? 
* Do you have a concept of leader elections to decide which processes should write to storage or do you support putting your service to readonly mode?
* Are you connection strings encrypted in your [.cscfg](https://docs.microsoft.com/en-us/azure/cloud-services/cloud-services-model-and-package#serviceconfigurationcscfg) files?

## What certificates are you using?
* Do you know what [certificates](https://docs.microsoft.com/en-us/azure/cloud-services/cloud-services-certs-create#what-are-service-certificates) are being installed with your service?
  * Do you install older thumbprints of the same subject name?
* Do you use [management certificates](https://docs.microsoft.com/en-us/azure/cloud-services/cloud-services-certs-create#what-are-management-certificates)?
* Do you have a certificate [bound](https://docs.microsoft.com/en-us/azure/cloud-services/cloud-services-configure-ssl-certificate-portal) to SSL?
* Are the certificates kept in an azure [key vault](https://docs.microsoft.com/en-us/azure/key-vault/key-vault-overview)?

## How do you develop and deploy your service?
* Do you use Azure [emulator express](https://docs.microsoft.com/en-us/visualstudio/azure/vs-azure-tools-emulator-express-debug-run?view=vs-2017), or the [storage emulator](https://docs.microsoft.com/en-us/azure/storage/common/storage-use-emulator)?
* Do you use [azure powershell commandlets](https://docs.microsoft.com/en-us/powershell/azure/overview?view=azps-1.1.0) or something similar?
* Do you use [management certificates](https://docs.microsoft.com/en-us/azure/cloud-services/cloud-services-certs-create#what-are-management-certificates) or [RBAC](https://docs.microsoft.com/en-us/azure/role-based-access-control/overview) for your deployment?
* Do you use [VIP swap](https://docs.microsoft.com/en-us/azure/cloud-services/cloud-services-how-to-manage-portal#swap-deployments-to-promote-a-staged-deployment-to-production) or [in-place upgrade](https://docs.microsoft.com/en-us/azure/cloud-services/cloud-services-update-azure-service#how-an-upgrade-proceeds)?

**Navigation**: <a href="Introduction.md">First<a> | <a href="Introduction.md">Prev</a> | <a href="Chapter2.md">Next</a> | <a href="Chapter6.md">Last</a>
