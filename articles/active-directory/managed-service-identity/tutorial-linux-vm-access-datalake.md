---
title: Use Managed Service Identity for a Linux VM to access Azure Data Lake Store
description: A tutorial that shows you how to use Managed Service Identity (MSI) for a Linux VM to access Azure Data Lake Store.
services: active-directory
documentationcenter: 
author: daveba
manager: mtillman
editor: 

ms.service: active-directory
ms.component: msi
ms.devlang: na
ms.topic: article
ms.tgt_pltfrm: na
ms.workload: identity
ms.date: 11/20/2017
ms.author: skwan
---

# Use Managed Service Identity for a Linux VM to access Azure Data Lake Store

[!INCLUDE[preview-notice](../../../includes/active-directory-msi-preview-notice.md)]

This tutorial shows you how to use Managed Service Identity for a Linux virtual machine (VM) to access Azure Data Lake Store. Azure automatically manages identities that you create through MSI. You can use MSI to authenticate to services that support Azure Active Directory (Azure AD) authentication, without needing to insert credentials into your code. 

In this tutorial, you learn how to:

> [!div class="checklist"]
> * Enable MSI on a Linux VM. 
> * Grant your VM access to Azure Data Lake Store.
> * Get an access token by using the VM identity and use it to access Azure Data Lake Store.

## Prerequisites

[!INCLUDE [msi-qs-configure-prereqs](../../../includes/active-directory-msi-qs-configure-prereqs.md)]

[!INCLUDE [msi-tut-prereqs](../../../includes/active-directory-msi-tut-prereqs.md)]

## Sign in to Azure

Sign in to the [Azure portal](https://portal.azure.com).

## Create a Linux virtual machine in a new resource group

For this tutorial, we create a new Linux VM. You can also enable MSI on an existing VM.

1. Select the **New** button in the upper-left corner of the Azure portal.
2. Select **Compute**, and then select **Ubuntu Server 16.04 LTS**.
3. Enter the virtual machine information. For **Authentication type**, select **SSH public key** or **Password**. The created credentials allow you to log in to the VM.

   !["Basics" pane for creating a virtual machine](../media/msi-tutorial-linux-vm-access-arm/msi-linux-vm.png)

4. In the **Subscription** list, select a subscription for the virtual machine.
5. To select a new resource group that you want the virtual machine to be created in, select **Resource group** > **Create new**. When you finish, select **OK**.
6. Select the size for the VM. To see more sizes, select **View all** or change the **Supported disk type** filter. In the settings pane, keep the defaults and select **OK**.

## Enable MSI on your VM

A VM MSI enables you to get access tokens from Azure AD without you needing to put credentials into your code. Enabling Managed Service Identity on a VM, does two things: registers your VM with Azure Active Directory to create its managed identity, and it configures the identity on the VM.

1. For **Virtual Machine**, select the virtual machine that you want to enable MSI on.
2. In the left pane, select **Configuration**.
3. You see **Managed service identity**. To register and enable MSI, select **Yes**. If you want to disable it, select **No**.
   !["Register with Azure Active Directory" selection](../media/msi-tutorial-linux-vm-access-arm/msi-linux-extension.png)
4. Select **Save**.

## Grant your VM access to Azure Data Lake Store

Now you can grant your VM access to files and folders in Azure Data Lake Store. For this step, you can use an existing Data Lake Store instance or create a new one. To create a Data Lake Store instance by using the Azure portal, follow the [Azure Data Lake Store quickstart](https://docs.microsoft.com/azure/data-lake-store/data-lake-store-get-started-portal). There are also quickstarts that use Azure CLI and Azure PowerShell in the [Azure Data Lake Store documentation](https://docs.microsoft.com/azure/data-lake-store/data-lake-store-overview).

In Data Lake Store, create a new folder and grant MSI permission to read, write, and execute files in that folder:

1. In the Azure portal, select **Data Lake Store** in the left pane.
2. Select the Data Lake Store instance that you want to use.
3. Select **Data Explorer** on the command bar.
4. The root folder of the Data Lake Store instance is selected. Select **Access** on the command bar.
5. Select **Add**.  In the **Select** box, enter the name of your VM--for example, **DevTestVM**. Select your VM from the search results, and then click **Select**.
6. Click **Select Permissions**.  Select **Read** and **Execute**, add to **This folder**, and add as **An access permission only**. Select **Ok**.  The permission should be added successfully.
7. Close the **Access** pane.
8. For this tutorial, create a new folder. Select **New Folder** on the command bar, and give the new folder a name--for example **TestFolder**.  Select **Ok**.
9. Select the folder that you created, and then select **Access** on the command bar.
10. Similar to step 5, select **Add**. In the **Select** box, enter the name of your VM. Select your VM from the search results, and then click **Select**.
11. Similar to step 6, select **Select Permissions**. Select **Read**, **Write**, and **Execute**, add to **This folder**, and add as **An access permission entry and a default permission entry**. Select **Ok**.  The permission should be added successfully.

MSI can now perform all operations on files in the folder that you created. For more information on managing access to Data Lake Store, see [Access Control in Data Lake Store](https://docs.microsoft.com/azure/data-lake-store/data-lake-store-access-control).

## Get an access token and call the Data Lake Store file system

Azure Data Lake Store natively supports Azure AD authentication, so it can directly accept access tokens obtained via MSI. To authenticate to the Data Lake Store file system, you send an access token issued by Azure AD to your Data Lake Store file system endpoint. The access token is in an authorization header in the format "Bearer \<ACCESS_TOKEN_VALUE\>".  To learn more about Data Lake Store support for Azure AD authentication, see [Authentication with Data Lake Store using Azure Active Directory](https://docs.microsoft.com/azure/data-lake-store/data-lakes-store-authentication-using-azure-active-directory).

In this tutorial, you authenticate to the REST API for the Data Lake Store file system by using cURL to make REST requests.

> [!NOTE]
> The client SDKs for the Data Lake Store file system do not yet support Managed Service Identity.

To complete these steps, you need an SSH client. If you are using Windows, you can use the SSH client in the [Windows Subsystem for Linux](https://msdn.microsoft.com/commandline/wsl/about). If you need assistance configuring your SSH client's keys, see [How to use SSH keys with Windows on Azure](../../virtual-machines/linux/ssh-from-windows.md) or [How to create and use an SSH public and private key pair for Linux VMs in Azure](../../virtual-machines/linux/mac-create-ssh-keys.md).

1. In the portal, browse to your Linux VM. In **Overview**, select **Connect**.  
2. Connect to the VM by using the SSH client of your choice. 
3. In the terminal window, by using cURL, make a request to the local MSI endpoint to get an access token for the Data Lake Store file system. The resource identifier for Data Lake Store is "https://datalake.azure.net/".  It's important to include the trailing slash in the resource identifier.
    
   ```bash
   curl 'http://169.254.169.254/metadata/identity/oauth2/token?api-version=2018-02-01&resource=https%3A%2F%2Fdatalake.azure.net%2F' -H Metadata:true   
   ```
    
   A successful response returns the access token that you use to authenticate to Data Lake Store:

   ```bash
   {"access_token":"eyJ0eXAiOiJ...",
    "refresh_token":"",
    "expires_in":"3599",
    "expires_on":"1508119757",
    "not_before":"1508115857",
    "resource":"https://datalake.azure.net/",
    "token_type":"Bearer"}
   ```

4. By using cURL, make a request to your Data Lake Store file system's REST endpoint to list the folders in the root folder. This is a simple way to check that everything is configured correctly. Copy the value of the access token from the previous step. It's important that the string "Bearer" in the Authorization header has a capital "B." You can find the name of your Data Lake Store instance in the **Overview** section of the **Data Lake Store** pane in the Azure portal.

   ```bash
   curl https://<YOUR_ADLS_NAME>.azuredatalakestore.net/webhdfs/v1/?op=LISTSTATUS -H "Authorization: Bearer <ACCESS_TOKEN>"
   ```
    
   A successful response looks like this:

   ```bash
   {"FileStatuses":{"FileStatus":[{"length":0,"pathSuffix":"TestFolder","type":"DIRECTORY","blockSize":0,"accessTime":1507934941392,"modificationTime":1508105430590,"replication":0,"permission":"770","owner":"bd0e76d8-ad45-4fe1-8941-04a7bf27f071","group":"bd0e76d8-ad45-4fe1-8941-04a7bf27f071"}]}}
   ```

5. Now you can try uploading a file to your Data Lake Store instance. First, create a file to upload.

   ```bash
   echo "Test file." > Test1.txt
   ```

6. By using cURL, make a request to your Data Lake Store file system's REST endpoint to upload the file to the folder that you created earlier. The upload involves a redirect, and cURL follows the redirect automatically. 

   ```bash
   curl -i -X PUT -L -T Test1.txt -H "Authorization: Bearer <ACCESS_TOKEN>" 'https://<YOUR_ADLS_NAME>.azuredatalakestore.net/webhdfs/v1/<FOLDER_NAME>/Test1.txt?op=CREATE' 
   ```

    A successful response looks like this:

   ```bash
   HTTP/1.1 100 Continue
   HTTP/1.1 307 Temporary Redirect
   Cache-Control: no-cache, no-cache, no-store, max-age=0
   Pragma: no-cache
   Expires: -1
   Location: https://mytestadls.azuredatalakestore.net/webhdfs/v1/TestFolder/Test1.txt?op=CREATE&write=true
   x-ms-request-id: 756f6b24-0cca-47ef-aa12-52c3b45b954c
   ContentLength: 0
   x-ms-webhdfs-version: 17.04.22.00
   Status: 0x0
   X-Content-Type-Options: nosniff
   Strict-Transport-Security: max-age=15724800; includeSubDomains
   Date: Sun, 15 Oct 2017 22:10:30 GMT
   Content-Length: 0
       
   HTTP/1.1 100 Continue
       
   HTTP/1.1 201 Created
   Cache-Control: no-cache, no-cache, no-store, max-age=0
   Pragma: no-cache
   Expires: -1
   Location: https://mytestadls.azuredatalakestore.net/webhdfs/v1/TestFolder/Test1.txt?op=CREATE&write=true
   x-ms-request-id: af5baa07-3c79-43af-a01a-71d63d53e6c4
   ContentLength: 0
   x-ms-webhdfs-version: 17.04.22.00
   Status: 0x0
   X-Content-Type-Options: nosniff
   Strict-Transport-Security: max-age=15724800; includeSubDomains
   Date: Sun, 15 Oct 2017 22:10:30 GMT
   Content-Length: 0
   ```

By using other APIs for the Data Lake Store file system, you can append to files, download files, and more.

Congratulations! You've authenticated to the Data Lake Store file system by using MSI for a Linux VM.

## Related content

- For an overview of MSI, see [Managed Service Identity overview](overview.md).
- For management operations, Data Lake Store uses Azure Resource Manager.  For more information on using MSI to authenticate to Resource Manager, see [Use a Linux VM Managed Service Identity (MSI) to access Resource Manager](https://docs.microsoft.com/azure/active-directory/msi-tutorial-linux-vm-access-arm).
- Learn more about [authentication with Data Lake Store by using Azure Active Directory](https://docs.microsoft.com/azure/data-lake-store/data-lakes-store-authentication-using-azure-active-directory).
- Learn more about [file system operations on Azure Data Lake Store by using the REST API](https://docs.microsoft.com/azure/data-lake-store/data-lake-store-data-operations-rest-api) or the [WebHDFS FileSystem APIs](https://docs.microsoft.com/rest/api/datalakestore/webhdfs-filesystem-apis).
- Learn more about [access control in Data Lake Store](https://docs.microsoft.com/azure/data-lake-store/data-lake-store-access-control).

Use the following comments section to provide feedback and help us refine and shape our content.
