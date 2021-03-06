> Azure Resource Manager Community Documentation - Beta Version

> Work in progress - This community driven documentation is considered to be in preview stage at this time. Documentation might contain errors, might not cover every aspect or might lack complete parts, even important parts. Please help us make this documentation better by [contributing](CONTRIBUTING.md) anything that you think would make it better.


---

# Azure Resource Manager Templates Basics
With Resource Manager, you can create a simple template (in JSON format) that defines deployment and configuration of your application. This template is known as a Resource Manager Template and provides a declarative way to define deployment. By using a template, you can repeatedly deploy your application throughout the app lifecycle and have confidence your resources are deployed in a consistent state.

**Template size is limted to 1MB, and each parameter file to 64KB**. The 1 MB limit applies to the final state of the template after it has been expanded with iterative resource definitions, and values for variables and parameters.

## Template Structure
A tempalte consists of 4 basics parts, that constracts the resources needed to deploy the application, the parameters and their valus and any output values.

Basic template structure:
```json
{
   "$schema": "http://schema.management.azure.com/schemas/2015-01-01/deploymentTemplate.json#",
   "contentVersion": "",
   "parameters": {  },
   "variables": {  },
   "resources": [  ],
   "outputs": {  }
}
```
|Element | Required | Description |
|--------|----------|-------------|
|$schema | Yes      |location of the JSON schema file that describes the version of the template language|
|contentVersion | Yes | Version of the template (such as 1.0.0.0). When deploying resources using the template, this value can be used to make sure that the right template is being used|
|parameters | No | Values that are provided when deployment is executed to customize resource deployment |
|variables| No | Values that are used as JSON fragments in the template to simplify template language expressions |
|resources | Yes | Types of services that are deployed or updated in a resource group | 
|outputs | No | Values that are returned after deployment |

### Parameters
The parameters section defined the values a user can input while deploying the resources. The values of the parameters can be used thoughout the template.

```json
"parameters": {
   "<parameterName>" : {
     "type" : "<type-of-parameter-value>",
     "defaultValue": "<optional-default-value-of-parameter>",
     "allowedValues": [ "<optional-array-of-allowed-values>" ],
     "minValue": <optional-minimum-value-for-int-parameters>,
     "maxValue": <optional-maximum-value-for-int-parameters>,
     "minLength": <optional-minimum-length-for-string-secureString-array-parameters>,
     "maxLength": <optional-maximum-length-for-string-secureString-array-parameters>,
     "metadata": {
         "description": "<optional-description-of-the parameter>" 
     }
   }
}
```
|ELEMENT NAME	|REQUIRED	|DESCRIPTION|
|-------------|---------|-----------|
|parameterName|	Yes	|Name of the parameter. Must be a valid JavaScript identifier.|
|type|	Yes	|Type of the parameter value. See the list below of allowed types.|
|defaultValue|	No	|Default value for the parameter, if no value is provided for the parameter.|
|allowedValues|	No	|Array of allowed values for the parameter to make sure that the right value is provided.|
|minValue|	No	|The minimum value for int type parameters, this value is inclusive.|
|maxValue|	No	|The maximum value for int type parameters, this value is inclusive.|
|minLength|	No	|The minimum length for string, secureString and array type parameters, this value is inclusive.|
|maxLength|	No	|The maximum length for string, secureString and array type parameters, this value is inclusive.|
|description|	No	|Description of the parameter which will be displayed to users of the template through the portal custom template interface.|

**Allowed Values**

* string or secureString - any valid JSON string
* int - any valid JSON integer
* bool - any valid JSON boolean
* object or secureObject - any valid JSON object
* array - any valid JSON array

**Optional Parameter**

To specify a parameter as optional, set its defaultValue to an empty string.

**Secure Parameter Values**

secureString - all passwords, keys, and other secrets should use the **secureString** type. Template parameters with the secureString type cannot be read after resource deployment.

**Parameter File**

Parameters can be passed to the deployment using a parameter file. This is also a JSON file, with the following structure:
```json
{
   "$schema": "http://schema.management.azure.com/schemas/2015-01-01/deploymentTemplate.json#",
   "contentVersion": "",
   "parameters": {
    "<parameterName>" : {
      <Parameters structure - same as in template file>
    }
   }
}
```

### Variables
Variables are used to simplify template language expressions. Typically, these variables will be based on values provided from the parameters. Basic variables section structure:
```json
"variables": {
   "<variable-name>": "<variable-value>",
   "<variable-name>": { 
       <variable-complex-type-value> 
   }
}
```

Follwing is an example of a variable constructed from two parameters, one that is custructed from other variables and a varialbe that is a complex JSON object:
```json
"parameters": {
   "username": {
     "type": "string"
   },
   "password": {
     "type": "secureString"
   },
   "environmentName": {
     "type": "string",
     "allowedValues": [
       "test",
       "prod"
     ]
   }
 },
 "variables": {
   "connectionString": "[concat('Name=', parameters('username'), ';Password=', parameters('password'))]",
   "environmentSettings": {
     "test": {
       "instancesSize": "Small",
       "instancesCount": 1
     },
     "prod": {
       "instancesSize": "Large",
       "instancesCount": 4
     }
   },
   "currentEnvironmentSettings": "[variables('environmentSettings')[parameters('environmentName')]]",
   "instancesSize": "[variables('currentEnvironmentSettings').instancesSize",
   "instancesCount": "[variables('currentEnvironmentSettings').instancesCount"
}
```
### Resources
This section defines the resouces to create or update in the deployment. Each resource is defined seperatly. If there are dependencies between resources, they must be described in the resource definision. For example, if a Virtual Machine depends on a Storage Account, this will be defined in the Virtual Machine resource decleration. Azure Resource Manager analyzes dependencies to ensure resources are created in the correct order, and there is no meaning to the order in which the resources are defined in the template. 

```json
"resources": [
   {
     "apiVersion": "<api-version-of-resource>",
     "type": "<resource-provider-namespace/resource-type-name>",
     "name": "<name-of-the-resource>",
     "location": "<location-of-resource>",
     "tags": "<name-value-pairs-for-resource-tagging>",
     "comments": "<your-reference-notes>",
     "dependsOn": [
       "<array-of-related-resource-names>"
     ],
     "properties": "<settings-for-the-resource>",
     "resources": [
       "<array-of-dependent-resources>"
     ]
   }
]
```
|ELEMENT NAME	|REQUIRED	|DESCRIPTION    |
|--------------|-----------|---------------|
|apiVersion	|Yes	|Version of the API that supports the resource|
|type	|Yes	|Type of the resource. This value is a combination of the namespace of the resource provider and the resource type that the resource provider supports|
|name	|Yes	|Name of the resource. The name must follow URI component restrictions defined in RFC3986|
|location	|No	|Supported geo-locations of the provided resource|
|tags	|No	|Tags that are associated with the resource|
|comments	|No	|Your notes for documenting the resources in your template|
|dependsOn	|No	|Resources that the resource being defined depends on. The dependencies between resources are evaluated and resources are deployed in their dependent order. When resources are not dependent on each other, they are attempted to be deployed in parallel. The value can be a comma separated list of a resource names or resource unique identifiers|
|properties	|No	|Resource specific configuration settings|
|resources	|No	|Child resources that depend on the resource being defined|

### Output
This is an optional section, were you can specify the values to be returned from the deployment.
```json
"outputs": {
   "<outputName>" : {
     "type" : "<type-of-output-value>",
     "value": "<output-value-expression>",
   }
}
```
|ELEMENT NAME	|REQUIRED	|DESCRIPTION|
|--------------|-----------|-----------|
|outputName	|Yes	|Name of the output value. Must be a valid JavaScript identifier|
|type	|Yes	|Type of the output value. Output values support the same types as template input parameters|
|value	|Yes	|Template language expression which will be evaluated and returned as output value|

## Template Sample - simple compute deployment
The following template creates a virtual network with 2 subnets, a network interface, storage account and a Windows virtual machine. The json file can be found in this directoy [simple-win-vm-sample.json](json/simple-win-vm-sample.json)
```json
{
  "$schema": "http://schema.management.azure.com/schemas/2015-01-01/deploymentTemplate.json#",
  "contentVersion": "1.0.0.0",
  "parameters": {
    "storageAccountNameName": {
      "type": "string"
    },
    "storageAccountNameType": {
      "type": "string",
      "defaultValue": "Standard_LRS",
      "allowedValues": [
        "Standard_LRS",
        "Standard_ZRS",
        "Standard_GRS",
        "Standard_RAGRS",
        "Premium_LRS"
      ]
    },
    "vmNameAdminPassword": {
      "type": "securestring"
    },
    "vmNameAdminUserName": {
      "type": "string",
      "minLength": 1
    },
    "vmNameName": {
      "type": "string",
      "minLength": 1
    },
    "vmNameWindowsOSVersion": {
      "type": "string",
      "defaultValue": "2012-R2-Datacenter",
      "allowedValues": [
        "2008-R2-SP1",
        "2012-Datacenter",
        "2012-R2-Datacenter",
        "Windows-Server-Technical-Preview"
      ]
    }
  },
    "variables": {        
        "virtualNetworkNamePrefix": "10.0.0.0/16",
        "virtualNetworkNameSubnet1Name": "Subnet-1",
        "virtualNetworkNameSubnet1Prefix": "10.0.0.0/24",
        "virtualNetworkNameSubnet2Name": "Subnet-2",
        "virtualNetworkNameSubnet2Prefix": "10.0.1.0/24",
        "vmNameImagePublisher": "MicrosoftWindowsServer",
        "vmNameImageOffer": "WindowsServer",
        "vmNameOSDiskName": "vmNameOSDisk",
        "vmNameVmSize": "Standard_D1",
        "vmNameVnetID": "[resourceId('Microsoft.Network/virtualNetworks', 'virtualNetworkName')]",
        "vmNameSubnetRef": "[concat(variables('vmNameVnetID'), '/subnets/', variables('virtualNetworkNameSubnet1Name'))]",
        "vmNameStorageAccountContainerName": "vhds",
        "vmNameNicName": "[concat(parameters('vmNameName'), 'NetworkInterface')]"
    },
    "resources": [
        {
            "name": "[parameters('storageAccountNameName')]",
            "type": "Microsoft.Storage/storageAccounts",
            "location": "[resourceGroup().location]",
            "apiVersion": "2015-05-01-preview",
            "dependsOn": [ ],
            "tags": {
                "displayName": "storageAccountName"
            },
            "properties": {
                "accountType": "[parameters('storageAccountNameType')]"
            }
        },
        {
            "name": "virtualNetworkName",
            "type": "Microsoft.Network/virtualNetworks",
            "location": "[resourceGroup().location]",
            "apiVersion": "2015-05-01-preview",
            "dependsOn": [ ],
            "tags": {
                "displayName": "virtualNetworkName"
            },
            "properties": {
                "addressSpace": {
                    "addressPrefixes": [
                        "[variables('virtualNetworkNamePrefix')]"
                    ]
                },
                "subnets": [
                    {
                        "name": "[variables('virtualNetworkNameSubnet1Name')]",
                        "properties": {
                            "addressPrefix": "[variables('virtualNetworkNameSubnet1Prefix')]"
                        }
                    },
                    {
                        "name": "[variables('virtualNetworkNameSubnet2Name')]",
                        "properties": {
                            "addressPrefix": "[variables('virtualNetworkNameSubnet2Prefix')]"
                        }
                    }
                ]
            }
        },
        {
            "name": "[variables('vmNameNicName')]",
            "type": "Microsoft.Network/networkInterfaces",
            "location": "[resourceGroup().location]",
            "apiVersion": "2015-05-01-preview",
            "dependsOn": [
                "[concat('Microsoft.Network/virtualNetworks/', 'virtualNetworkName')]"
            ],
            "tags": {
                "displayName": "vmNameNic"
            },
            "properties": {
                "ipConfigurations": [
                    {
                        "name": "ipconfig1",
                        "properties": {
                            "privateIPAllocationMethod": "Dynamic",
                            "subnet": {
                                "id": "[variables('vmNameSubnetRef')]"
                            }
                        }
                    }
                ]
            }
        },
        {
            "name": "[parameters('vmNameName')]",
            "type": "Microsoft.Compute/virtualMachines",
            "location": "[resourceGroup().location]",
            "apiVersion": "2015-05-01-preview",
            "dependsOn": [
                "[concat('Microsoft.Storage/storageAccounts/', parameters('storageAccountNameName'))]",
                "[concat('Microsoft.Network/networkInterfaces/', variables('vmNameNicName'))]"
            ],
            "tags": {
                "displayName": "vmName"
            },
            "properties": {
                "hardwareProfile": {
                    "vmSize": "[variables('vmNameVmSize')]"
                },
                "osProfile": {
                    "computerName": "[parameters('vmNameName')]",
                    "adminUsername": "[parameters('vmNameAdminUsername')]",
                    "adminPassword": "[parameters('vmNameAdminPassword')]"
                },
                "storageProfile": {
                    "imageReference": {
                        "publisher": "[variables('vmNameImagePublisher')]",
                        "offer": "[variables('vmNameImageOffer')]",
                        "sku": "[parameters('vmNameWindowsOSVersion')]",
                        "version": "latest"
                    },
                    "osDisk": {
                        "name": "vmNameOSDisk",
                        "vhd": {
                            "uri": "[concat('http://', parameters('storageAccountNameName'), '.blob.core.windows.net/', variables('vmNameStorageAccountContainerName'), '/', variables('vmNameOSDiskName'), '.vhd')]"
                        },
                        "caching": "ReadWrite",
                        "createOption": "FromImage"
                    }
                },
                "networkProfile": {
                    "networkInterfaces": [
                        {
                            "id": "[resourceId('Microsoft.Network/networkInterfaces', variables('vmNameNicName'))]"
                        }
                    ]
                }
            }
        }
    ]
}
```

## Resources and References

https://azure.microsoft.com/en-us/documentation/articles/resource-group-authoring-templates/

[MVA course](https://mva.microsoft.com/en-us/training-courses/deep-dive-into-azure-resource-manager-scenarios-and-patterns-13793?l=i1m06ZJYB_7001937557)




