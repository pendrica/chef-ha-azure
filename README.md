# chef-ha-azure

Deploy a highly available Chef Server system in Azure, designed to be used in conjunction with https://docs.chef.io/install_server_ha.html

This template does not install the Chef Server software, it only deploys the required infrastructure, see below for some notes installing the rest of the software.

<a href="https://portal.azure.com/#create/Microsoft.Template/uri/https%3A%2F%2Fraw.githubusercontent.com%2Fpendrica%2Fchef-ha-azure%2Fmaster%2Farm-template%2Fazuredeploy.json" target="_blank"><img src="http://azuredeploy.net/deploybutton.png"/></a>

The template is intended for demo purposes and is a work-in-progress but by default includes all the features required to operate a highly-available system in Azure, such as:

- 2x Frontend nodes
- 3x Backend nodes
- Frontend and backend in separate availability zones, update and fault domains
- Azure Load balancer used for frontend
- Separate subnets for frontend and backend
- Network Security groups (like firewall rules) that apply to the subnet and each host machine that only allow the specific application traffic to communicate. 

#Deployment Instructions

Modify the parameters as required and then deploy using Azure-cli or PowerShell:

## Azure-cli example

```bash
azure login
azure group create --name "yourcompany-chef-ha" --location "westeurope"
azure group deployment create --name "deploy-chef-ha" --resource-group "yourcompany-chef-ha" --template-file azuredeploy.json --parameters-file azuredeploy.parameters.json
```

## PowerShell example

```powershell
PS> Login-AzureRmAccount
PS> New-AzureRmResourceGroup -Name "yourcompany-chef-ha" -Location "westeurope"
PS> New-AzureRmResourceGroupDeployment -ResourceGroupName "yourcompany-chef-ha" -TemplateFile azuredeploy.json -TemplateParameterFile azuredeploy.parameters.json
```

# Getting started - Chef Server HA 2.0 installation

* You can sign up for the Beta program for Chef Server HA 2.0 [here](https://pages.chef.io/201605-HighAvailabilityBeta_Request.html)
* Some notes about running the HA 2.0 installation on Azure are [here](INSTALLNOTES.md)

# Notes/Considerations

- By default, SSH is open to the front end via the frontend Load Balancer
- No traffic is permitted from the backend to the frontend subnets - all SSH access must go via the frontend machine.
- Production installs of this will need to consider enabling diagnostics, Premium storage, VM sizes etc. - this template is intended for demo purposes only.
- Currently access to the nodes is via username and password only (specified in the parameters).  SSH configuration will come in time...

# Bugs/Enhancements etc.

Please feel free to raise issues/pull requests etc. via the usual mechanisms on GitHub.
