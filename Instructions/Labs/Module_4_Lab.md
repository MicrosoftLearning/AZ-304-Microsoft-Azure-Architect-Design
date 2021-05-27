---
lab:
    title: '4: Managing Azure AD Authentication and Authorization'
    module: 'Module 4: Design Authentication and Authorization'
---

# Lab: Managing Azure AD Authentication and Authorization
# Student lab manual

## Lab scenario

As part of its migration to Azure, Adatum Corporation needs to define its identity strategy. Adatum has a single domain Active Directory forest named adatum.com and owns the corresponding, publicly registered DNS domain. As the Adatum Enterprise Architecture team is exploring the option of transitioning some of the on-premises workloads to Azure, it intends to evaluate integration between its Active Directory Domain Services (AD DS) environment and the Azure Active Directory (Azure AD) tenant associated with the target Azure subscription as the core component of its longer-term authentication and authorization model.

The new model should facilitate single sign-on, along with per-application step-up authentication that leverages multi-factor authentication capabilities of Azure AD. To implement single sign-on, the Architecture team plans to deploy Azure AD Connect and configure it for password hash synchronization, resulting in matching user objects in both identity stores. Choosing the optimal authentication method is the first concern for organizations wanting to move to the cloud. Azure AD password hash synchronization is the simplest way to implement single sign-on authentication for on-premises users when accessing Azure AD-integrated resources. This method is also required by some premium Azure AD features, such as Identity Protection.

To implement step-up authentication, the Adatum Enterprise Architecture team intends to take advantage of Azure AD Conditional Access policies. Conditional Access policies support enforcement of multi-factor authentication depending on the type of application or resource being accessed. Conditional Access policies are enforced after the first-factor authentication has been completed. Conditional Access can be based on a wide range of factors, including:

- User or group membership. Policies can be targeted to specific users and groups giving administrators fine-grained control over access.
- IP Location information. Organizations can create trusted IP address ranges that can be used when making policy decisions. Administrators can specify entire countries/regions IP ranges to block or allow traffic from.
- Device. Users with devices of specific platforms or marked with a specific state can be used when enforcing Conditional Access policies.
- Application. Users attempting to access specific applications can trigger different Conditional Access policies.
- Real-time and calculated risk detection. Signals integration with Azure AD Identity Protection allows Conditional Access policies to identify risky sign-in behavior. Policies can then force users to perform password changes or multi-factor authentication to reduce their risk level or be blocked from access until an administrator takes manual action.
- Microsoft Cloud App Security (MCAS). Enables user application access and sessions to be monitored and controlled in real time, increasing visibility and control over access to and activities performed within your cloud environment.

To accomplish these objectives the Adatum Enterprise Architecture team intends to test integration of its Active Directory Domain Services (AD DS) forest with its Azure Active Directory (Azure AD) tenant and evaluate the conditional access functionality for its pilot users.

## Objectives
  
After completing this lab, you will be able to:

 - Deploy an Azure VM hosting an AD DS domain controller

 - Create and configure an Azure AD tenant

 - Integrate an AD DS forest with an Azure AD tenant


## Lab Environment
  
Windows Server admin credentials

-  User Name: **Student**

-  Password: **Pa55w.rd1234**

Estimated Time: 120 minutes


## Lab Files

-  \\\\AZ304\\AllFiles\\Labs\\10\\azuredeploy30410suba.json


## Instructions

### Exercise 0: Prepare the lab environment

The main tasks for this exercise are as follows:

1. Identify an available DNS name for an Azure VM deployment

1. Deploy an Azure VM running an AD DS domain controller by using an Azure Resource Manager QuickStart template


#### Task 1: Identify an available DNS name for an Azure VM deployment

1. From your lab computer, start a web browser, navigate to the [Azure portal](https://portal.azure.com), and sign in by providing credentials of a user account with the Owner role in the subscription you will be using in this lab.

1. In the Azure portal, open **Cloud Shell** pane by selecting on the toolbar icon directly to the right of the search textbox.

1. If prompted to select either **Bash** or **PowerShell**, select **PowerShell**. 

    >**Note**: If this is the first time you are starting **Cloud Shell** and you are presented with the **You have no storage mounted** message, select the subscription you are using in this lab, and select **Create storage**. 

1. In the Cloud Shell pane, run the following to identify an available DNS name you will need to provide in the next task (substitute the placeholder `<custom-label>` with any valid DNS hostname which is likely to be globally unique and the placeholder `<Azure region>` with the name of the Azure region into which you want to deploy the Azure VM that will host an Active Directory domain controller):

    ```powershell
    Test-AzDnsAvailability -DomainNameLabel <custom-label> -Location '<location>'
    ```
      > **Note**: To identify Azure regions where you can provision Azure VMs, refer to [https://azure.microsoft.com/en-us/regions/offers/](https://azure.microsoft.com/en-us/regions/offers/), you can also get the list of the regions using **Powershell cmdlet**
      ```powershell
      Get-AzLocation | FT
      ```

1. Verify that the command returned **True**. If not, rerun the same command with a different value of the `<custom-label>` until the command returns **True**.

1. Record the value of the `<custom-label>` that resulted in the successful outcome. You will need it in the next task.


#### Task 2: Deploy an Azure VM running an AD DS domain controller by using an Azure Resource Manager QuickStart template

1. In the Azure portal, in the toolbar of the Cloud Shell pane, select the **Upload/Download files** icon, in the drop-down menu select **Upload**, and upload the file **\\\\AZ304\\AllFiles\Labs\\10\\azuredeploy30410suba.json** into the Cloud Shell home directory.

1. From the Cloud Shell pane, run the following to create a resource groups (replace the `<Azure region>` placeholder with the name of the Azure region that you specified in the previous task):

   ```powershell
   $location = '<Azure region>'
   ```
   ```powershell
   New-AzSubscriptionDeployment `
     -Location $location `
     -Name az30410subaDeployment `
     -TemplateFile $HOME/azuredeploy30410suba.json `
     -rgLocation $location `
     -rgName 'az30410a-labRG'
   ```

1. In the Azure portal, close the **Cloud Shell** pane.

1. From your lab computer, open another browser tab and navigate to the [https://github.com/Azure/azure-quickstart-templates/tree/master/application-workloads/active-directory/active-directory-new-domain](https://github.com/Azure/azure-quickstart-templates/tree/master/application-workloads/active-directory/active-directory-new-domain).


1. On the **Create a new Windows VM and create a new AD Forest, Domain and DC** page, select **Deploy to Azure**. This will automatically redirect the browser to the **Create an Azure VM with a new AD Forest** blade in the Azure portal.

1. On the **Create an Azure VM with a new AD Forest** blade, select **Edit parameters**.

1. On the **Edit parameters** blade, select **Load file**, in the **Open** dialog box, select **\\\\AZ304\\AllFiles\Labs\\10\\azuredeploy30410rga.parameters.json**, select **Open**, and then select **Save**. 

1. On the **Create an Azure VM with a new AD Forest** blade, specify the following settings (leave others with their existing values):

    | Setting | Value | 
    | --- | --- |
    | Subscription | the name of the Azure subscription you are using in this lab |
    | Resource group | **az30410a-labRG** |
    | Dns Prefix | the DNS hostname you identified in the previous task| 

1. On the **Create an Azure VM with a new AD Forest** blade, select **Review + create** and select **Create**.

    > **Note**: Do not wait for the deployment to complete but instead proceed to the next exercise. The deployment might take about 15 minutes. You will use the virtual machine deployed in this task in the third exercise of this lab.


### Exercise 1: Create and configure an Azure AD tenant
  
The main tasks for this exercise are as follows:

1. Create an Azure AD tenant

1. Create and configure Azure AD users

1. Activate and assign Azure AD Premium P2 licensing


#### Task 1: Create an Azure AD tenant

1. In the Azure portal, search for and select **Azure Active Directory** and, on the Azure Active Directory blade, select **+ Create a tenant**.

1. On the **Basics** tab of the **Create a directory** blade, select the **Azure Active Directory** option and select **Next: Configuration >**.

1. On the **Configuration** tab of the **Create a directory** blade, specify the following settings (leave others with their existing values):

    | Setting | Value |
    | --- | --- |
    | Organization name | **Adatum Lab** |
    | Initial domain name | any valid DNS name consisting of lower case letters and digits and starting with a letter | 
    | Country/Region | **United States** |

   > **Note**: The green check mark in the **Initial domain name** text box will indicate that the domain name you typed in is valid and unique.

1. Select **Next: Review + create** and then select **Create**.

1. Refresh the browser page displaying the Azure portal, search for and select **Azure Active Directory** and, on the Azure Active Directory blade, select **Switch tenant**.

1. In the **Directory + subscription** blade, on the **Adatum Lab** card, click **Switch**.


#### Task 2: Create and configure Azure AD users

1. On the **Adatum Lab \| Overview** Azure Active Directory blade, in the **Manage** section, select **Users**, on the **Users | All users** blade, select your user account to display its **Profile** settings. 

1. On the profile blade of your user account, select **Edit**, in the **Settings** section, set **Usage location** to **United States** and select **Save** to save the change.

    >**Note**: This is necessary in order to assign an Azure AD Premium P2 license to your user account later in this lab.

1. Navigate back to the **Users - All users** blade, and then select **+ New user**.

1. On the **New user** blade, specify the following settings (leave others with their defaults):

    | Setting | Value |
    | --- | --- |
    | User name | **az30410-aaduser1** |
    | Name | **az30410-aaduser1** |
    | Auto-generate password | enabled |
    | Show password | enabled |
    | Roles | **Global administrator** |
    | Usage location | **United States** |

    >**Note**: Record the full user name (including the domain name) and the auto-generated password. You will need it later in this task.

1. On the **New user** blade, select **Create**

1. On the lab computer, open an **InPrivate** browser window and sign in to the [Azure portal](https://portal.azure.com) using the newly created **az30410-aaduser1** user account. When prompted to update the password, change the password to **Pa55w.rd1234**. 

1. Sign out as the **az30410-aaduser1** user from the Azure portal and close the InPrivate browser window.


#### Task 3: Activate and assign Azure AD Premium P2 licensing

1. Back in the browser window displaying the Azure portal, navigate to the **Overview** blade of the **Adatum Lab** Azure AD tenant and, in the **Manage** section, select **Licenses**.

1. On the **Licenses \| Overview** blade, select **All products**, select **+ Try/Buy**.

1. On the **Activate** blade, in the **Azure AD Premium P2** section, select **Free trial** and then select **Activate**. 

1. Refresh the browser window showing the **Licenses \| All products** blade to verify that the activation was successful. 

1. On the **Licenses - All products** blade, select the **Azure Active Directory Premium P2** entry. 

1. On the **Azure Active Directory Premium P2 \| Licensed users** blade, select **+ Assign**. 

1. On the **Assign license** blade, select **Add users and groups**, and on the **Add users and groups** blade, select both your account and the **az30410-aaduser1** user account and click **Select** for each.

1. Back on the **Assign license** blade, select **Assignment options**, review the options listed on the **License options** blade, and select **OK**.

1. On the **Assign license** blade, select **Review and Assign** the on the **Assign** page, select **Assign**. 


### Exercise 2: Integrate an AD DS forest with an Azure AD tenant
  
The main tasks for this exercise are as follows:

1. Assign a custom domain name to the Azure AD tenant

1. Configure AD DS in the Azure VM

1. Install Azure AD Connect

1. Configure properties of synchronized user accounts


#### Task 1: Assign a custom domain name to the Azure AD tenant

1. In the Azure portal, navigate to the **Azure Active Directory Adatum Lab | Overview** blade.

1. On the **Adatum Lab \| Overview** blade, select **Custom domain names**.

1. On the **Adatum Lab \| Custom domain names** blade, identify the primary, default DNS domain name associated with the Azure AD tenant. 

    >**Note**: Record the value of the primary DNS name of the Azure AD tenant. You will need it in the next task.

1. On the **Adatum Lab \| Custom domain names** blade, select **+ Add custom domain**.

1. On the **Custom domain name** blade, in the **Custom domain name** text box, type **adatum.com**, and select **Add domain**. 

1. On the **adatum.com** blade, review the information necessary to perform verification of the Azure AD domain name and close the blade.

    > **Note**: You will not be able to complete the validation process because you do not own the **adatum.com** DNS domain name. This will *not* prevent you from synchronizing the **adatum.com** Active Directory domain with the Azure AD tenant. You will use for this purpose the default primary DNS name of the Azure AD tenant (the name ending with the **onmicrosoft.com** suffix), which you identified earlier in this task. However, keep in mind that, as a result, the DNS domain name of the Active Directory domain and the DNS name of the Azure AD tenant will differ. This means that Adatum users will need to use different names when signing in to the Active Directory domain and when signing in to Azure AD tenant.


#### Task 2: Configure AD DS in the Azure VM

> **Note**: Make sure that the deployment of the Azure VM you initiated at the beginning of the lab has completed before you start this exercise.

1. In the Azure portal, search for and select **Azure Active Directory** and, on the Azure Active Directory blade, select **Switch tenant**.

1. On the **Switch tenant** blade, click the **Switch** button in the tile representing the Azure AD tenant associated with the Azure subscription into which you deployed the **az30410a-vm1** Azure VM in the previous exercise of this lab. 

1. In the Azure portal, search for and select **Virtual machines** and, on the **Virtual machines** blade, select **az30410a-vm1**.

1. On the **az30410a-vm1** blade, select **Connect**, in the drop-down menu, select **RDP**, on the **RDP** tab of the **az30410a-vm1 | Connect** blade, in the **IP address** drop-down list, select the **Load balancer public IP address** entry, select **Download RDP File** and open the downloaded RDP file.

1. When prompted, sign in with the following credentials:

    | Setting | Value | 
    | --- | --- |
    | User Name | **Student** |
    | Password | **Pa55w.rd1234** |

1. Within the Remote Desktop session to **az30410a-vm1**, in the Server Manager window, select **Local Server**, select the **On** link next to the **IE Enhanced Security Configuration** label, and, in the **IE Enhanced Security Configuration** dialog box, select both **Off** options.

1. Within the Remote Desktop session to **az30410a-vm1**, in the Server Manager window, select **Tools** and, in the drop-down menu, select **Active Directory Administrative Center**

1. In **Active Directory Administrative Center**, select **adatum (local)**, in the **Tasks** pane, select **New**, and, in the cascading menu, select **Organizational Unit**.

1. In the **Create Organizational Unit** window, in the **Name** text box, type **ToSync** and select **OK**.

1. Double-click the newly crated **ToSync** organizational unit such that it its content appears in the details pane of the Active Directory Administrative Center console. 

1. In the **Tasks** pane, within the **ToSync** section, select **New**, and, in the cascading menu, select **User**.

1. In the **Create User** window, create a new user account with the following settings (leave others with their existing values) and select **OK**:

    | Setting | Value | 
    | --- | --- |
    | Full Name | **aduser1** |
    | User UPN logon | **aduser1** |
    | User SamAccountName logon | **aduser1** |
    | Password | **Pa55w.rd1234** | 
    | Other password options | **Password never expires** |


#### Task 3: Install Azure AD Connect

1. Within the Remote Desktop session to **az30410a-vm1**, start Internet Explorer, navigate to the download page of [Microsoft Edge](https://www.microsoft.com/en-us/edge/business/download), download Microsoft Edge installer and perform the installation. 

1. Within the Remote Desktop session to **az30410a-vm1**, in Microsoft Edge, navigate to the [Azure portal](https://portal.azure.com), and sign in by using the **az30410-aaduser1** user account you created the previous exercise. When prompted, specify the full user name you recorded and the **Pa55w.rd1234** password.

1. In the Azure portal, search for and select **Azure Active Directory** and, on the **Adatum Lab | Overview** blade, select **Azure AD Connect**.

1. On the **Adatum Lab | Azure AD Connect** blade, select the **Download Azure AD Connect** link. You will be redirected to the **Microsoft Azure Active Directory Connect** download page.

1. On the **Microsoft Azure Active Directory Connect** download page, select **Download**.

1. When prompted, select **Run** to start the **Microsoft Azure Active Directory Connect** wizard.

1. On the **Welcome to Azure AD Connect** page of the **Microsoft Azure Active Directory Connect** wizard, select the checkbox **I agree to the license terms and privacy notice** and select **Continue**.

1. On the **Express Settings** page of the **Microsoft Azure Active Directory Connect** wizard, select the **Customize** option.

1. On the **Install required components** page, leave all optional configuration options deselected and select **Install**.

1. On the **User sign-in** page, ensure that only the **Password Hash Synchronization** is enabled and select **Next**.

1. On the **Connect to Azure AD** page, authenticate by using the credentials of the **az30410-aaduser1** user account you created in the previous exercise and select **Next**. 

1. On the **Connect your directories** page, select the **Add Directory** button to the right of the **adatum.com** forest entry.

1. In the **AD forest account** window, ensure that the option to **Create new AD account** is selected, specify the following credentials, and select **OK**:

    | Setting | Value | 
    | --- | --- |
    | User Name | **ADATUM\Student** |
    | Password | **Pa55w.rd1234** |

1. Back on the **Connect your directories** page, ensure that the **adatum.com** entry appears as a configured directory and select **Next**

1. On the **Azure AD sign-in configuration** page, note the warning stating **Users will not be able to sign-in to Azure AD with on-premises credentials if the UPN suffix does not match a verified domain name**, enable the checkbox **Continue without matching all UPN suffixes to verified domain**, and select **Next**.

    > **Note**: As explained earlier, this is expected, since you could not verify the custom Azure AD DNS domain **adatum.com**.

1. On the **Domain and OU filtering** page, select the option **Sync selected domains and OUs**, clear all checkboxes, select only the checkbox next to the **ToSync** OU, and select **Next**.

1. On the **Uniquely identifying your users** page, accept the default settings, and select **Next**.

1. On the **Filter users and devices** page, accept the default settings, and select **Next**.

1. On the **Optional features** page, accept the default settings, and select **Next**.

1. On the **Ready to configure** page, ensure that the **Start the synchronization process when configuration completes** checkbox is selected and select **Install**.

    > **Note**: Installation should take about 2 minutes.

1. Review the information on the **Configuration complete** page and select **Exit** to close the **Microsoft Azure Active Directory Connect** window.


#### Task 4: Configure properties of synchronized user accounts

1. Within the Remote Desktop session to **az30410a-vm1**, in the Microsoft Edge window displaying the Azure portal, navigate to the **Users - All users** blade of the Adatum Lab Azure AD tenant.

1. On the **Users \| All users** blade, note that the list of user objects includes the **aduser1** account, with the **Yes** entry appearing in the **Directory synced** column.

    > **Note**: You might have to wait a few minutes and select **Refresh** for the **aduser1** user account to appear.

1. On the **Users \| All users** blade, select the **aduser1** entry.

1. On the **aduser1 \| Profile** blade, note the full name of the user account.

    > **Note**: Record the full user name. You will need it in the next exercise.

1. On the **aduser1 \| Profile** blade, in the **Job info** section, note that the **Department** attribute is not set.

1. Within the Remote Desktop session to **az30410a-vm1**, switch to **Active Directory Administrative Center**, select the **aduser1** entry in the list of objects in the **ToSync** OU, and, in the **Tasks** pane, in the **ToSync** section, select **Properties**.

1. In the **aduser1** window, in the **Organization** section, in the **Department** text box, type **Sales**, and select **OK**.

1. Within the Remote Desktop session to **az30410a-vm1**, start **Windows PowerShell**.

1. From the **Administrator: Windows PowerShell** console, run the following to start Azure AD Connect delta synchronization:

   ```powershell
   Import-Module -Name 'C:\Program Files\Microsoft Azure AD Sync\Bin\ADSync\ADSync.psd1'

   Start-ADSyncSyncCycle -PolicyType Delta
   ```

1. Switch to the Microsoft Edge window displaying the **aduser1 \| Profile** blade, refresh the page and note that the **Department** property is set to **Sales**.

    > **Note**: You might need to wait for another minute and refresh the page again if the **Department** attribute remains not set.

1. On the **aduser1 \| Profile** blade, select **Edit**.

1. On the **aduser1 \| Profile** blade, in the **Settings** section, in the **Usage location** drop-down list, select **United States** and then select **Save**.

1. On the **aduser1 \| Profile** blade, select **Licenses**.

1. On the **aduser1 \| Licenses** blade, select **+ Assignments**.

1. On the **Update license assignments** blade, select the **Azure Active Directory Premium P2** checkbox and select **Save**.


### Exercise 3: Implement Azure AD conditional access
  
The main tasks for this exercise are as follows:

1. Disable Azure AD security defaults.

1. Create an Azure AD conditional access policy

1. Verify Azure AD conditional access

1. Remove Azure resources deployed in the lab


#### Task 1: Disable Azure AD security defaults.

1. Within the Remote Desktop session to **az30410a-vm1**, in the Microsoft Edge window displaying the Azure portal, navigate to the **Adatum Lab | Overview** blade of the Adatum Lab Azure AD tenant.

1. On the **Adatum Lab | Overview** blade, in the **Manage** section, select **Properties**.

1. On the **Adatum Lab | Properties** blade, select the **Manage Security defaults** link at the bottom of the page.

1. On the **Enable Security defaults** blade, set **Enable Security defaults** switch to **No**, select the checkbox **My organization is using Conditional Access**, and select **Save**. 


#### Task 2: Create an Azure AD conditional access policy

1. On the **Adatum Lab \| Properties** blade, in the **Manage** section, select the **Security**.

1. On the **Security \| Getting started** blade, select **Conditional Access**.

1. On the **Conditional Access \| Policies** blade, select **+ New policy**.

1. On the **New** blade, in the **Name** text box, type **Azure portal MFA enforcement**. 

1. On the **New** blade, in the **Assignments** section, select **Users and groups**, on the **Include** tab, select **Select users and groups**, select the **Users and groups** checkbox, on the **Select** blade, select **aduser1**, and confirm your choice by clicking **Select**.

1. Back on the **New** blade, in the **Assignments** section, select **Cloud apps or actions**, on the **Include** tab, select **Select apps**, click **Select**, on the **Select** blade, select **Microsoft Azure Management** checkbox, and confirm your choice by clicking **Select**.

1. Back on the **New** blade, in the **Access controls** section, select **Grant**, on the **Grant** blade, ensure that the **Grant** option is selected, select **Require multi-factor authentication**, and confirm your choice by clicking **Select**.

1. Back on the **New** blade, set the **Enable policy** switch to **On** and select **Create**.


#### Task 3: Verify Azure AD conditional access

1. Within the Remote Desktop session to **az30410a-vm1**, in the **Microsoft Edge** window, select **Settings** menu header, in the **Settings** menu, select **Safety**, in the cascading menu, select **InPrivate Browsing**, and, in the InPrivate Microsoft Edge window, navigate to the Access Panel Applications portal [https://myapplications.microsoft.com](https://myapplications.microsoft.com).

1. When prompted, sign in by using the synchronized Azure AD account of the **aduser1**, using the full user name you recorded in the previous exercise and the **Pa55w.rd1234** password. 

1. Verify that you can successfully sign in to the Access Panel Applications portal. 

1. In the same browser window, navigate to the [Azure portal](https://portal.azure.com).

1. Note that, this time, you are presented with the message **More information required**. Within the page displaying the message, select **Next**. 

1. At that point, you will be redirected to the **Additional security verification** page, which will step you through configuring multi-factor authentication.

    > **Note**: Completing the multi-factor authentication configuration is optional. If you proceed, you will need to designate your mobile device as an authentication phone or to use it to run a mobile app.


#### Task 4: Remove Azure resources deployed in the lab

1. Within the Remote Desktop session to **az30410a-vm1**, start Microsoft Edge and browse to the Microsoft Online Services Sign-In Assistant for IT Professionals RTW at [https://download.microsoft.com/download/7/1/E/71EF1D05-A42C-4A1F-8162-96494B5E615C/msoidcli_64bit.msi](https://download.microsoft.com/download/7/1/E/71EF1D05-A42C-4A1F-8162-96494B5E615C/msoidcli_64bit.msi). 

1. In the browsers **Downloads** dropdown, click **Open file**. 

1. On the **Open File - Security Warning** dialog box click run.

1. in the **Microsoft Online Services Sign-In Assistant Setup** wizard, click the check box to accept the license agreement, then click **Install**.

1. In the **User Account Control** dialog box select **Yes**.

1. Once the setup completes, click **Finish**.

1. Within the Remote Desktop session to **az30410a-vm1**, start **Windows PowerShell** console.

1. In the **Administrator: Windows PowerShell** window, run the following to install the required PowerShell module:

   ```powershell
   [Net.ServicePointManager]::SecurityProtocol = [Net.SecurityProtocolType]::Tls12
   Install-PackageProvider -Name NuGet -MinimumVersion 2.8.5.201 -Force
   Install-Module MSOnline -Force
   ```
1. In the **Administrator: Windows PowerShell** window, run the following to authenticate to the **Adatum Lab** Azure AD tenant:

   ```powershell
   Connect-MsolService
   ```

1. When prompted to authenticate, provide the credentials of the **az30410-aaduser1** user account.

1. In the **Administrator: Windows PowerShell** window, run the following to disable Azure AD Connect synchronization:

   ```powershell
   Set-MsolDirSyncEnabled -EnableDirSync $false -Force
   ```

    > **Note**: If you receive an error message at this point, you might have to wait for up to 12 hours and try again.

1. From the lab computer, in the browser window displaying the Azure portal, switch to the **Adatum Lab** tenant, navigate to the **Azure Active Directory Premium P2 - Licensed users** blade, select the user accounts to which you assigned licenses in this lab, select **Remove license**, and, when prompted to confirm, select **OK**.

1. In the Azure portal, navigate to the **Users - All users** blade and ensure that all user accounts you created in this lab are no longer listed as **Directory synced**.

1. On the **Users - All users** blade, select each user accounts you created in this lab and select **Delete** in the toolbar. 

1. Navigate to the **Adatum Lab - Overview** blade of the Adatum Lab Azure AD tenant, select **Delete tenant**, on the **Delete directory 'Adatum Lab'** blade, select the **Get permission to delete Azure resources** link, on the **Properties** blade of Azure Active Directory, set **Access management for Azure resources** to **Yes** and select **Save**.

1. Sign out from the Azure portal and sign in back. 

1. Navigate back to the **Delete directory 'Adatum Lab'** blade and select **Delete**.

1. On the lab computer, in the browser window displaying the Azure portal, make sure you are connected to the original Azure Active Directory tenant, and start a PowerShell session within the Cloud Shell pane.

1. From the Cloud Shell pane, run the following to list the resource group you created in this exercise:

   ```powershell
   Get-AzResourceGroup -Name 'az30410*'
   ```

    > **Note**: Verify that the output contains only the resource group you created in this lab. This group will be deleted in this task.

1. From the Cloud Shell pane, run the following to delete the resource group you created in this lab

   ```powershell
   Get-AzResourceGroup -Name 'az30410*' | Remove-AzResourceGroup -Force -AsJob
   ```

1. Close the Cloud Shell pane.
