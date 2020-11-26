# PackerAzureRM-2020 - Examples to create Azure VM Images with Packer (en-us) 

Packer is an open source tool for creating identical machine images for multiple platforms from a single source configuration. Packer is lightweight, runs on every major operating system, and is highly performant, creating machine images for multiple platforms in parallel. Packer does not replace configuration management like Chef or Ansible. In fact, when building images, Packer is able to use tools like Chef or Ansible to install and configure software onto the image. Packer only builds images. It does not attempt to manage them in any way. After they're built, it is up to you to launch or destroy them as you see fit..


---------------------------------------------------------------------------------------------------------
The purpose of this repository is to help you to start using Packer to build your custom VM images on Microsoft Azure Cloud.

--------------------------------------------------------------------------------------------------------

**Step 1 : Download and install Packer**

Packer binaries are available here  : </br>[https://www.packer.io/downloads.html]

Official documentation to install Packer </br> [https://learn.hashicorp.com/tutorials/packer/getting-started-install] 

**Step 2 : Prepare Azure prerequisite to connect Packer to Microsoft Azure**

You can connect Packer to Azure using your Azure Credential but it's better (my opinion) to use an Azure Service Principal Name (SPN).
You need first an Azure Active Directory, then 3 ways to do that : 
- Use portal to create an Azure Active Directory application and service principal that can access resources : </br>[https://docs.microsoft.com/en-us/azure/azure-resource-manager/resource-group-create-service-principal-portal](https://docs.microsoft.com/en-us/azure/azure-resource-manager/resource-group-create-service-principal-portal)
- Create an Azure service principal with Azure PowerShell : </br>
[https://docs.microsoft.com/en-us/powershell/azure/create-azure-service-principal-azureps?view=azurermps-4.2.0](https://docs.microsoft.com/en-us/powershell/azure/create-azure-service-principal-azureps?view=azurermps-4.2.0)
- Create an Azure service principal with Azure CLI 2.0 : </br>[https://docs.microsoft.com/en-us/cli/azure/create-an-azure-service-principal-azure-cli?toc=%2fazure%2fazure-resource-manager%2ftoc.json](https://docs.microsoft.com/en-us/cli/azure/create-an-azure-service-principal-azure-cli?toc=%2fazure%2fazure-resource-manager%2ftoc.json)

     az ad sp create-for-rbac --name ServicePrincipalName –role contributor –o jsonc

Create a Resource Group for VM Images that will be build with Packer

**Step 3 : Create a JSON file for Packer**

JSON File section used by Packer are the following: 
- ``"variables": ["..."],``        ==> variables
- ``"builders": ["..."],``         ==> provider where image is build, Connection information, VM size to prepare image...
- ``"provisioners": ["..."]``      ==> add functionalities, install applications, execute configuration scripts, Configuration Manager tools (Chef, Puppet, Ansible..)...

All Packer-name.json files in this repo are example you can use and adapt.

>/!\ Attention : there are difference between a JSON for a Linux VM and a JSON for a Windows VM: for Windows VM you need to provide the SPN ObjectID 
>>==> To obtain this ObjectID, use :
>>- the following Powershell command with SPN name : Get-AzureRmADServicePrincipal -SearchString "NameofyourSPN"
>>- or the Azure CLI 2.0 command (Thanks to **Etienne Deneuve** who provides me this command):
>>az ad sp list --query "[?displayName=='NameofyourSPN'].{ name: displayName, objectId: objectId }" <br/>
>>
>> **You can now use also : ``az ad sp list --spn http://Packer``**
>>```Bash
>>az ad sp list --spn http://Packer
>>[
>>  {
>>    "appId": "852e8410-5762-4890-XXXX-36b3f002377c",
>>    "displayName": "Packer",
>>    "objectId": "a5a66fab-913a-455a-XXXX-d0eb534cb90e",
>>    "objectType": "ServicePrincipal",
>>    "servicePrincipalNames": [
>>      "http://Packer",
>>      "852e8410-5762-4890-XXXX-36b3f002377c"
>>    ]
>>  }
>>]
>>```
- To check the name of a SPN with an ObjectID : `` az ad sp show --id XXXXXXXX-XXXX-XXXX-XXXX-XXXXXXXXXXXXX  <-- here objectid of Service Principal Name.`` Note : the az command can take many minutes to provide a result, depending on the size of your Azure AD. PowerShell command is really quicker (filtering is done on server side)

I tried to use as much as possible variables in my Packer template json file. So you will need an additional json file that contains the values of variables : variables-Packer.json

---------------------------------------------------------------------------------------------------------

**Step 4: Validate your JSON fichier with Packer**

When your Packer's JSON file is ready, validate it with the following Packer command : 
``packer validate -var-file=variables-Packer.json nameofjsonfileforPacker.json ``

---------------------------------------------------------------------------------------------------------
**Step 5 : Build your image in Azure with Packer.**

Execute the following command : ``packer build -var-file=variables-Packer.json nameofjsonfileforPacker.json``

The basic steps performed by Packer to create a Linux image build are:
1. Create a resource group.
1. Validate and deploy a VM template.
1. Execute provision - defined by the user; typically shell commands.
1. Power off and capture the VM.
1. Delete the resource group.
1. Delete the temporary VM's OS disk.

The basic steps performed by Packer to create a Windows image build are:
1. Create a resource group.
1. Validate and deploy a KeyVault template.
1. Validate and deploy a VM template.
1. Execute provision - defined by the user; typically shell commands.
1. Power off and capture the VM.
1. Delete the resource group.
1. Delete the temporary VM's OS disk.

The output from the Packer build process is a virtual hard disk (VHD) in the specified storage account or an image disk (Azure Managed Disk) depending on your choice in Packer's JSON file. 

---------------------------------------------------------------------------------------------------------
**Step 6 : Create a new VM in Azure based on the image built by Packer**

Using the Azure Portal or PowerShell  </br> 
cf [https://docs.microsoft.com/en-us/azure/virtual-machines/windows/create-vm-generalized-managed?toc=/azure/virtual-machines/windows/toc.json] 


---------------------------------------------------------------------------------------------------------
**Global Summary** 

The following step must be follow to create a custom image in Azure:
1. Install Packer
1. in a folder, copy and customize Packer .json files
1. Create a resource group and a Service Principal Name in Azure AD
1. Validate Packer package. Example: ``packer validate -var-file=variables-Packer.json nameofjsonfileforPacker.json``
1. Build Packer package. Example: ``packer build -var-file=variables-Packer.json nameofjsonfileforPacker.json``
1. Create a new VM from the managed image built by Packer
