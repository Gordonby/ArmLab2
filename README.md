# Infrastructure as Code Lab

## 2nd in the Series
This lab assumes you have completed the earlier IaaS lab https://aka.ms/armlab
It takes you to a point when you have a single VM deployed in a new vnet, using a storage account for disk storage.

## Abstract

During this lab, you will build on the infrastructure design from the Arm Lab.
You will extend a single VM environment to use a load balancer, availability set and with another VM sitting alongside.
We will introduce the concept of nested templates and a more advanced use of parameters including "Parameters as Objects".

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

1. In Visual Studio, create a new Resouce project and proceed using a **Blank Template** and click **OK**.

    ![image](./media/image3.png)

1. In the **JSON Outline** window, right-click **Resources** and select **Add New Resource**.

1. In the **Add Resource** window, select **Windows Virtual Machine**.

    - Set the resource **Name** to *vm*.

    - Set the **Storage account** field to *tmpstorage*, we will be removing this reference from the template in a later step.

    - Select from the **Virtual network/subnet** dropdown the Virtual Network that you created in the portal, and the Windows subnet

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
1. Now remove all references to the storage account from the template as it is no longer needed.

1. Make sure to change the API version in the Virtual Machine definition to
    ```json
        "apiVersion": "2016-04-30-preview"
    ...
   We need to do this because the older API versions don't know about managed disks.

1. Add *2016-Datacenter* to the list of Windows Os Versions and make it the default

1. Reduce the list of tempalte parameters to
    1. vmName
    1. vmAdminPassword
    1. vmWindowsOSVersion
You'll do this by creating variables with the same name, then changing the reference to them throughout the template from parameters('name') to variables('name')

You should have something that looks like this
    ![image](./media/StageOneParamsVariables.png) 

1. Deploy the VM to Azure and make sure creation completed successfully.

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


# Exercise : Parameters

1. 

# Exercise : Nested templates

1. 