# ARM and RBAC
If you set up your resources a while ago and haven't paid attention to what has changed since then, you may not know ARM. So lets take a quick detour to look at it, since it is the management service for Azure. Similarly, if you use connection strings/access keys/management certificates everywhere, it helps to understand how to use RBAC to manage access.

## Understanding Azure Resource Manager or ARM
You really should just go read up [this document](https://docs.microsoft.com/en-us/azure/azure-resource-manager/resource-group-overview) which 
explains it well. At a high level, 
* all objects that you create in Azure are described as resources
* which can be collectively wrapped into resource groups. All create-deploy-delete operations can be done at the resource group scope using [declarative](https://stackoverflow.com/questions/1784664/what-is-the-difference-between-declarative-and-imperative-programming) sytax in [templates](https://docs.microsoft.com/en-us/azure/azure-resource-manager/resource-group-authoring-templates)
* Each resource is backed by a [resource provider](https://docs.microsoft.com/en-us/azure/azure-resource-manager/resource-manager-supported-services), e.g. a request to create a load balancer will be handled by the load balancer service in networking. 
* All commands to the portal, SDKs or from the command line essentially go to ARM, which detects the resources being described and farms out the request to he appropriate provider
* Letting the appropriate provider handle the request frees you from a lot of tactical decisions. You simply describe your resources, and the appropriate experts handle it for you (often including upgrades).

## ARM templates
While deployments can still be done via portal or powershell, it starts to make sense to understand the json template syntax. 
### Dipping your toes
There are a few ways to get started with ARM templates
* Use the portal to create a resource and explore the resulting template: Start [here](https://docs.microsoft.com/en-us/azure/azure-resource-manager/resource-manager-quickstart-create-templates-use-the-portal)
* Use visual studio code to create and do [similar things](https://docs.microsoft.com/en-us/azure/azure-resource-manager/resource-manager-quickstart-create-templates-use-visual-studio-code?tabs=CLI)
### Authoring ARM templates
Once you get a feel of the templates, spend some time tweaking them and [deploying](https://docs.microsoft.com/en-us/azure/azure-resource-manager/resource-group-template-deploy) to see how the changes take effect.
* Microsoft has published a large number of [quick-start templates](https://github.com/Azure/azure-quickstart-templates) on github, so look there for inpiration.
* Then read up on the [syntax](https://docs.microsoft.com/en-us/azure/azure-resource-manager/resource-group-authoring-templates) and dive into the [builtin functions](https://docs.microsoft.com/en-us/azure/azure-resource-manager/resource-group-template-functions)

If you got this far, you have understood a lot of ARM and you should be able to find other pointers on your own

## Understanding role based access control or RBAC
RBAC is the idea that every resource has common usage/access patterns, which can be formalized and granted to specific identities. 
* With ARM, everything (storage/sql servers/key vaults/VMs) is a resource
  * For every resource, Azure teams have created well known access patterns, and called them roles.
* With [Azure Active Directory](https://docs.microsoft.com/en-us/azure/active-directory/fundamentals/active-directory-whatis), all users and even applications can be given an identity and that identity can have a specific type of access. Access can be granted or denied to a user/app 
  * individually
  * as a group
  * as a combined identity, where by the app can only get access if launched by a specific user or group of users

Start [here](https://docs.microsoft.com/en-us/azure/role-based-access-control/overview) if you want to learn more about RBAC

**Navigation**: <a href="Introduction.md">First<a> | <a href="Chapter3.md">Prev</a> | <a href="Chapter5.md">Next</a> | <a href="Chapter6.md">Last</a>
