# Infrastructure as Code Lab

## 2nd in the Series
This lab assumes you have completed the earlier IaaS lab https://aka.ms/armlab
It takes you to a point when you have a single VM deployed in a new vnet, using a storage account for disk storage.

## Learning Objectives

1. Working with existing Azure resources

1. Introduction to managed disks instead of storage accounts for VM disk storage

1. Advanced use of parameters to capture the right level of detail

1. Using conditional logic

1. Best practice use of variables

1. Nested templates

1. Passing parameters to nested templates

**Estimated time to complete this lab: *120* minutes**

# Exercise : Add a VM to an existing template

1. In the Azure Portal, create a virtual network in a new Resource Group.

1. Create two subnets, "Windows-Sandbox" and "Linux-Sandbox"

1. In Visual Studio, create a new *Azure Resource Group* project and proceed using a **Blank Template** and click **OK**.

    ![image](./media/image3.png)

1. In the **JSON Outline** window, right-click **Resources** and select **Add New Resource**.

1. In the **Add Resource** window, select **Windows Virtual Machine**.

    - Set the resource **Name** to *vm*.

    - For the **Storage account**, select create new and name it *tmpstorage*, we will be removing this reference from the template in a later step.

    - Select from the **Virtual network/subnet** dropdown the Virtual Network that you created in the portal, and the *Windows subnet*

    - Click **Add**.

    ![image](./media/image7.png)


1. In the editor window for azuredeploy.json, find the definition for the osDisk.  Replace it with this statement.

    ```json
        "osDisk": {
            "createOption": "fromImage",
            "managedDisk": {
              "storageAccountType": "Standard_LRS"
            }
          },
    ```
1. Now remove all references to the storage account from the template as it is no longer needed.  Visual Studio can help this process, in the JSON Outline, find the *Storage* resource 

1. We need to change the API version for the *Virtual Machine*  because the older API versions don't know about managed disks.  
    ```json
        "apiVersion": "2016-04-30-preview"  

1. Add *2016-Datacenter* to the parameter list of Windows OS Versions and make it the default

1. Now we want tor educe the list of template parameters to really just being the ones we want the user to supply;
    1. vmName
    1. vmAdminPassword
    1. vmWindowsOSVersion
You'll do this by creating variables with the same name, then changing the reference to them throughout the template from parameters('name') to variables('name')

You should have something that looks like this
    ![image](./media/StageOneParamsVariables.png) 

1. At this stage several changes have been made, all of which we haven't tested out by deploying the VM.  Therefore we should deploy the VM to Azure and make sure creation completes successfully before proceeding too far.

# Exercise : Parameters

1. Add a new parameter to allow the user to select the VmSize

    ```json
    "VmSize": {
      "type": "string",
      "defaultValue": "Small",
      "allowedValues": [
        "Small",
        "Large"
      ]
    }
    ...

1. Add a variable to interpret the t-shirt size parameter to the appropriate Azure VM size
    ```json
        "vmVmSize": "[if(equals(parameters('VmSize'), 'Small'), 'Standard_A0', 'Standard_D2_V2')]",
    ...

1. Change the VMName parameter and deploy to Azure again.

1. Extend the VmSize Parameter to have a third option 

    ```json
        "VmSize": {
            "type": "string",
            "defaultValue": "Small",
            "allowedValues": [
                "Small",
                "Medium",
                "Large"
            ]
            }
    ...

1. Add a new Variable that will help us translate between the friendly VM Size value and the actual Azure VM Size we need for the template.

    ```json
        "VmShirtSize": {
            "Small": "Standard_A0",
            "Medium": "Standard_A3",
            "Large": "Standard_D2_V2"
        },
    ...

1. Now we can changed the *vmVmSize* variable, replacing the If statement with something a little more clever.

    ```json
        "vmVmSize": "[Variables('VmShirtSize')[Parameters('VmSize')]]",
    ...

# Exercise : Nested templates

1. Create a folder within your *Resource Group Project* called *nested*

1. Download the file from here: https://raw.githubusercontent.com/Gordonby/ArmLab2/master/scripts/Ex3/nested/Shutdown.json and put it in the *nested folder*

1. If you examine the file, you'll see it's exactly the same template format that you're used to.  This template takes 1 parameter, the *Virtual Machine Name* and creates a scheduled shutdown of it at 7pm.  

1. In order to call this template from our main template file, we have to add a resource of *Microsoft.Resources/deployments*

1. Add the resouce to your template, noticing that we're 
    1. Passing a parameter down to the template for the VM Name that comes from the main template Parameters
    1. Specifying a variable to be used as the path to the Nested Template.  This could be a local path or an http one.

    ```json
        {
        "apiVersion": "2017-05-10",
        "name": "Shutdown-policy",
        "type": "Microsoft.Resources/deployments",
        "properties": {
            "mode": "incremental",
            "templateLink": {
            "uri": "[concat(variables('nestedTemplateRoot') ,'Shutdown.json')]",
            "contentVersion": "1.0.0.0"
            },
            "parameters": {
            "virtualMachineName": { "value": "[parameters('vmName')]" }
            }
        },
        "dependsOn": [
            "[resourceId('Microsoft.Compute/virtualMachines', parameters('vmName'))]"
        ]
        }

    ```json
        "nestedTemplateRoot": "https://raw.githubusercontent.com/Gordonby/ArmLab2/master/scripts/Ex3/nested/"