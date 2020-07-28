---
lab:
    title: '6: Implementing Azure SQL Database-Based Applications'
    module: 'Module 6: Design a Solution for Databases'
---

# Lab: Implementing Azure SQL Database-Based Applications
# Student lab manual

## Lab scenario

Adatum Corporation has a number two tier applications with .NET Core-based front end and SQL Server-based backend. The Adatum Enterprise Architecture team is exploring the possibility of implementing these applications by leveraging Azure SQL Database as the data tier. Given intermittent, unpredictable usage of the existing SQL Server backend and relatively high tolerance for latency built into the front-end apps, Adatum is considering the serverless tier of Azure SQL Database. 

Serverless is a compute tier for individual Azure SQL Databases instances that automatically scales compute based on workload demand and bills for compute used per second. The serverless compute tier is also capable of automatically pausing databases during inactive periods when only storage is billed and automatically resumes databases when activity returns.

The Adatum Enterprise Architecture team is also interested in evaluating network-level security provided by the Azure SQL Databases, in order to ensure that it is possible to restrct inbound connections to specific ranges of IP addresses, in scenarios where the apps must be able to connect from its on-premises locations without relying on hybrid connectivity via Site-to-Site VPN or ExpressRoute. 
  
To accomplish these objectives, the Adatum Architecture team will test Azure SQL Database-based applications, including:

-  Implementing serverless tier of Azure SQL Database

-  Implementing .NET Core console apps that use Azure SQL Database as their data store


## Objectives
  
After completing this lab, you will be able to:

-  Implement serverless tier of Azure SQL Database

-  Configure .NET Core-based console apps that use Azure SQL Database as their data store


## Lab Environment
  
Windows Server admin credentials

-  User Name: **Student**

-  Password: **Pa55w.rd1234**

Estimated Time: 60 minutes


## Lab Files

-  None


### Exercise 1: Implement Azure SQL Database
  
The main tasks for this exercise are as follows:

1. Create Azure SQL Database 

1. Connect to and query Azure SQL Database


#### Task 1: Create Azure SQL Database 

1. From your lab computer, start a web browser, navigate to the [Azure portal](https://portal.azure.com), and sign in by providing credentials of a user account with the Owner role in the subscription you will be using in this lab.

1. In the Azure portal, search for and select **SQL database** and, on the **SQL databases** blade, select **+ Add**.

1. On the **Basics** tab of the **Create SQL Database** blade, specify the following settings (leave others with their default values):

    | Setting | Value | 
    | --- | --- |
    | Subscription | the name of the Azure subscription you will be using in this lab |
    | Resource group | the name of a new resource group **az30303a-labRG** |
    | Database name | **az30303a-db1** | 

1. Directly below the **Server** drop down list, select the **Create new** and, on the **New server** blade, specify the following settings and select **OK** (leave others with their default values):

    | Setting | Value | 
    | --- | --- |
    | Server name | any valid, globally unique name | 
    | Server admin login | **sqladmin** |
    | Password | **Pa55w.rd1234** |
    | Location | the name of an Azure region where you can provision SQL databases |
 
1. Next to the **Compute + storage** label, select the **Configure database** link.

1. On the **Configure** blade, select **Serverless**, review the corresponding hardware configuration and auto-pause delay settings, leave the **Enable auto-pause** checkbox enabled, and select **Apply**.

1. Back on the **Basics** tab of the **Create SQL Database** blade, select **Next: Networking >**. 

1. On the **Networking** tab of the **Create SQL Database** blade, specify the following settings (leave others with their default values):

    | Setting | Value | 
    | --- | --- |
    | Connectivity method | **Public endpoint** |    
    | Allow Azure services and resources to access this server | **Yes** |
    | Add current client IP address | **No** |

1. Select **Next: Additional settings >**. 

1. On the **Additional settings** tab of the **Create SQL Database** blade, specify the following settings (leave others with their default values):

    | Setting | Value | 
    | --- | --- |
    | Use existing data | **Sample** |
    | Enable advanced data security | **Not now** |

1. Select **Review + create** and then select **Create**. 

    >**Note**: Wait for the SQL database to be created. Provisioning should take about 2 minutes. 


#### Task 2: Connect to and query Azure SQL Database

1. In the Azure portal, search for and select **SQL database** and, on the **SQL databases** blade, select the entry representing the newly created **az30303a-db1** Azure SQL database.

1. On the SQL database blade, select **Query editor (preview)**.

1. In the **SQL Server authentication** section, in the **Password** textbox, type **Pa55w.rd1234** and select **OK**.

1. In the **Query editor (preview)** pane, on the **Query 1** tab, enter the following query and select **Run**:

    ```SQL
    SELECT TOP 20 pc.Name as CategoryName, p.name as ProductName
    FROM SalesLT.ProductCategory pc
    JOIN SalesLT.Product p
    ON pc.productcategoryid = p.productcategoryid;
    ```

1. Review the **Results** tab to verify that the query completed successfully.


### Exercise 2: Implement a .NET Core console app that uses Azure SQL Database as their data store
  
The main tasks for this exercise are as follows:

1. Identify ADO.NET connection information of Azure SQL Database

1. Create and configure a .NET Core console app

1. Test the .NET Core console app

1. Configure Azure SQL database firewall

1. Verify the functionality of the .NET Core console app

1. Remove Azure resources deployed in the lab


#### Task 1: Identify ADO.NET connection information of Azure SQL Database

1. In the Azure portal, on the blade of the Azure SQL database you deployed in the previous exercise, in the **Settings** section, select **Connection strings**.

1. On the **ADO.NET** tab, note the ADO.NET connection string for SQL authentication.


#### Task 2: Create and configure a .NET Core console app

1. In the Azure portal, open the **Cloud Shell** pane by selecting on the toolbar icon directly to the right of the search textbox.

1. If prompted to select either **Bash** or **PowerShell**, select **Bash**. 

    >**Note**: If this is the first time you are starting **Cloud Shell** and you are presented with the **You have no storage mounted** message, select the subscription you are using in this lab, and select **Create storage**. 

1. From the Cloud Shell pane, run the following to create a new folder named **az30303a1** and set it as your current directory:

   ```sh
   mkdir az30303a1
   cd az30303a1/
   ```

1. From the Cloud Shell pane, run the following to create a new app project file for a .NET Core-based app based on the desktop template:

   ```sh
   dotnet new console
   ```

1. In the Cloud Shell pane, use the built in editor to open and modify the **az30303a1.csproj** file by adding the following XML element between the `<Project>` tags: 

   ```xml
   <ItemGroup>
       <PackageReference Include="System.Data.SqlClient" Version="4.6.0" />
   </ItemGroup>
   ```

1. Save and close the **az30303a1.csproj** file.

1. In the Cloud Shell pane, use the built in editor to open and modify the **Program.cs** file by replacing its content with the following code: 

   ```cs
   using System;
   using System.Data.SqlClient;
   using System.Text;

   namespace sqltest
   {
       class Program
       {
           static void Main(string[] args)
           {
               try 
               { 
                   SqlConnectionStringBuilder builder = new SqlConnectionStringBuilder();
                   builder.ConnectionString="<your_ado_net_connection_string>";

                   using (SqlConnection connection = new SqlConnection(builder.ConnectionString))
                   {
                       Console.WriteLine("\nQuery data example:");
                       Console.WriteLine("=========================================\n");
        
                       connection.Open();       
                       StringBuilder sb = new StringBuilder();
                       sb.Append("SELECT TOP 20 pc.Name as CategoryName, p.name as ProductName ");
                       sb.Append("FROM [SalesLT].[ProductCategory] pc ");
                       sb.Append("JOIN [SalesLT].[Product] p ");
                       sb.Append("ON pc.productcategoryid = p.productcategoryid;");
                       String sql = sb.ToString();

                       using (SqlCommand command = new SqlCommand(sql, connection))
                       {
                           using (SqlDataReader reader = command.ExecuteReader())
                           {
                               while (reader.Read())
                               {
                                   Console.WriteLine("{0} {1}", reader.GetString(0), reader.GetString(1));
                               }
                           }
                       }                    
                   }
               }
               catch (SqlException e)
               {
                   Console.WriteLine(e.ToString());
               }
               Console.WriteLine("\nDone. Press enter.");
               Console.ReadLine(); 
           }
       }
   }
   ```

1. Leave the editor window open. 

1. In the Azure portal, on the blade displaying the connection strings for the **az30303a-db1** database, copy the ADO.NET connection string. 

1. Switch back to the editor window and replace the placeholder `<your_ado_net_connection_string>` with the value of the connection string you copied in the previous step.

1. In the connection string you copied into the editor window, replace the placeholder `{your_password}` with **Pa55w.rd1234**.

1. Save and close the **Program.cs** file.


#### Task 3: Test the .NET Core console app

1. From the Cloud Shell pane, run the following to compile the newly created .NET Core-based console app:

   ```sh
   dotnet restore
   ```

1. From the Cloud Shell pane, run the following to execute the newly created .NET Core-based console app:

   ```sh
   dotnet run
   ```

1. Note that the execution of the console app will trigger an error. 

    >**Note**: This is expected, since the connection from IP address assigned to the virtual machine running the Cloud Shell session must be explicitly allowed.


#### Task 4: Configure Azure SQL database firewall

1. From the Cloud Shell pane, run the following to identify the public IP address of the virtual machine running the Cloud Shell session:

   ```sh
   curl -s checkip.dyndns.org | sed -e 's/.*Current IP Address: //' -e 's/<.*$//'
   ```

1. In the Azure portal, on the blade displaying the connection strings for the **az30303a-db1** database, select **Overview** and, in the toolbar, select **Set server firewall**.

1. On the **Firewall settings** blade, set the following entries and select **Save**:

    | Setting | Value | 
    | --- | --- |
    | Rule name | **cloudshell** |
    | Start IP | the IP addres you identified earlier in this task |
    | End IP | the IP addres you identified earlier in this task |

    >**Note**: Obviously this is meant for the lab purposes only, since that IP address will change after you restart the Cloud Shell session.


#### Task 5: Verify the functionality of the .NET Core console app

1. From the Cloud Shell pane, run the following to execute the newly created .NET Core-based console app:

   ```sh
   dotnet run
   ```

1. Note that the execution of the console app will this time be successful and that it returns the same results as those displayed in the query editor within the Azure portal SQL database blade. 


#### Task 6: Remove Azure resources deployed in the lab

1. From the Cloud Shell pane, run the following to list the resource group you created in this exercise:

   ```sh
   az group list --query "[?starts_with(name,'az30303')]".name --output tsv
   ```

    > **Note**: Verify that the output contains only the resource group you created in this lab. This group will be deleted in this task.

1. From the Cloud Shell pane, run the following to delete the resource group you created in this lab

   ```sh
   az group list --query "[?starts_with(name,'az30303')]".name --output tsv | xargs -L1 bash -c 'az group delete --name $0 --no-wait --yes'
   ```

1. Close the Cloud Shell pane.
