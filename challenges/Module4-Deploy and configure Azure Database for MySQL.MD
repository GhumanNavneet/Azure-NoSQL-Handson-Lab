# Deploy and configure Azure Database for MySQL
### Before you start

* Please use  PowerBI-VM to execute this lab.

* Throughout this hands on lab there are many cli commands given. Some of these contain values that are listed with like this [vaule], in instances where this square bracket notation is used you must provide an apropriate value.


### Provision the Azure Resources 

1. Simply click the Deploy to Azure button below and follow the wizard to create the resources. You will need to log in to the Azure Portal, Use the lab azure credentials to login.<br/>
                                                                     
  [![Deploy to Azure](https://azuredeploy.net/deploybutton.png)](https://portal.azure.com/#create/Microsoft.Template/uri/https%3A%2F%2Fraw.githubusercontent.com%2FSpektraSystems%2FAzure-CosmosDB-and-Mysql-Postgres-Hands-on-Lab%2Fmaster%2Ftemplate%2Fmysql-resources.azuredeploy.json)

2. Select **existing Resource Group** for deployment. Provide unique name for **Site_name**, any name for **Serverfarms_name** and left other as default value.<br/>
3. Check-In **terms and condition** and click on **Purchase.**<br/>
<img src="images/template.jpg"/>

### Scenarios ###

Azure Database for MySQL
```
Scenario 1: Provision Azure Database for MySQL

Scenario 2: Deplopy website code

Scenario 3: Popluate MySQL with data

Scenario 4: Scale up

Conclusion
```

# Azure Database for MySQL

Azure Database for MySQL provides a fully managed database service for application development and deployment allowing you to provision a MySQL database instance in minutes and scale it on the fly.
With comprehensive pricing models, and configurable sizing, you get all the features you need such as high-availability, monitoring and security, and backup and recovery, all with reduced management overhead.  These features are built into the service and are aligned to a simple, predictable pricing model. 
<br>

# Scenario 1: Provision Azure Database for MySQ 

## Create a database using Azure CLI

1. Open a PowerShell command prompt, you can find a shortcut on the taskbar, and run:<br/>
```
  az login
```

2. When the prompt window instructs your to open [https://aka.ms/devicelogin](https://aka.ms/devicelogin) and enter the code shown in the console window.<br/>  
![Device Login code](images/az-login-code.png)

3. Sign in with your Microsoft Azure Credential.<br/>

4. You now have an Azure CLI session open against the Azure Subscription hosting your lab.<br/>

5. You will now use the CLI to provision an Azure Database for MySQL. In, open PowerShell  prompt, use the following command to provision an new Azure Database for MySQL:<br/>
    > Note: It would be best to provision this in the Resource Group created using the Azure Deploy tool.
```
az mysql server create --resource-group [MyResourceGroupName] --name [mysqlDatabaseName] --location westus --admin-user mysqlAdminUser --admin-password mysqlAdminPassw0rd! --performance-tier Basic --compute-units 50 --ssl-enforcement Disabled
```

6. By default the database is completely locked down and cannot be accessed, so we need to add a firewall rule to allow us to connect to the database server. The rule we're creating here allows all traffic, in production scenarios the rule would be much more restricted.<br/>
```
az mysql server firewall-rule create --resource-group [MyResourceGroupName] --server [mysqlDatabaseName]--name AllowAllIps --start-ip-address 0.0.0.0 --end-ip-address 255.255.255.255
```
7. Congratuations. You now have somewhere to store your data.

<br>

## Scenario 2: Deplopy website code

We now need to change some configuration code for the WordPress website so that it is able to consume data from the database which we have just provisioned. Once we have updated the code we'll deploy it to an Azure App Service which has already been provisioned.<br/>

1. First we need to acquire a copy of the code for the website. To do this we will clone it from an existing GitHub repository. In the open PowerShell command prompt run the following commands one by one:<br/>
```
    cd \  
    mkdir code  
    cd code  
    git clone https://github.com/gavinbarron/bikeshop.git
 ```

2. Next we need to set some git configuration values so that we can push any changes which are made to remote repositories. To do this run the following commands in the command prompt:<br/>
```
    cd bikeshop  
    git config user.name "holuser"  
    git config user.email "me@me.net"  
```

3. Now we need to update the code to use the MySQL databse which was previously provisioned. To do this, Open Visual Studio Code.<br/>

4. Using the file menu choose Open File.<br/>

 ![Open File](images/vscode-open-file.png)

5. Open the `wp-config.php` file at `C:\code\bikeshop\`.<br/>

6. On line 44 replace `[Username]` with **mysqlAdminUser@[mysqlDatabaseName]**, the value of the username for your database administrator account. [previously provisioned](1.md)<br/>

7. On line 47 replace `[Password]` with **mysqlAdminPassw0rd!**, the value of the password for your database administrator account. [previously provisioned](1.md)<br/>

8. On line 51 replace `[Servername]` with **[mysqlDatabaseName]**, the server name for your database. [previously provisioned](1.md)<br/>

![wp_config changes](images/wp_config.PNG "wp config changes")

9. Save the changes.<br/>

10. To deploy this code to the website that has been provisioned you will need to set up some deployment credentials for your user account. To do this, switch back to the open PowerShell prompt.<br/>

11. Execute this command to set your deployment **Username** and **Password**:<br/>
    > Note that the username here needs to be globally unique for all of Azure, so you do need to choose your own values.
```
 az webapp deployment user set --user-name [myDeploymentUserName] --password [MyGitDeployPassw0rd!]
```

12. Next we need the URL for the git repository which is configured for the website which will host the WordPress code. Open [https://portal.azure.com](https://portal.azure.com.<br/>

13. Open the **Resource Groups** blade and then open the resource group available to you.<br/>

14. Verify that there are:
    - An Azure Database for MySQL
    - An Azure App Service
    - An App Service Plan<br/>
 
 ![ResourceGroup Items](images/resourcegroup_items.png "Resource Group Items")

15. Open the App Service blade.<br/>

16. In overview section, copy the **Url** and add **scm.** after website name as show below:
```
https://website986.scm.azurewebsites.net
```

17. Now we need to configure the local git repository to know about the website. To do this we will add a new remote to the repository:<br/><br/>
```
git remote add website [copiedGitUrl]
```
18. Commit your changes to the local git repository:<br/>
```
git add .  
git commit -m "Updating config to point at new Azure Database for MySQL"
```
19. Deploy your changes to the **Azure App Service**:<br/>
```
git push website
```

20. In the pop-up enter the deployment credentials password that you set previously and click **OK**.<br/>

![Git Credentials](images/git-credentials-pop-up.png)

21. Wait for the push to the website to complete.<br/>

22. The website is now updated and just needs data.<br/>

<br>

## Scenario 3: Popluate MySQL with data

Before our WordPress website will work we need to populate the database with data.  To do this, we are going to use The MySQL Workbench to connect to our ``Azure Database for MySQL`` that we already have up and running and use a script to restore data into the database.

1. Open **MySQL Workbench**. <br/>

![Open MySQL Workbench](images/open-mysql-workbench.png)

2. Click **OK** and ignore the unsupported operating system warning.<br/><br/>

3. Click on the **+** icon to add a new MySQL Connection.<br/>

![Add new connection](images/add-mysql-connection.png)

4. Setup the new connection to MySQL using the server name, username, and password. [previously provisioned](1.ProvisionServer.md)

5. Click on **"Store in Vault..."** to enter password, then click OK.<br/>

![Workbench DB Connection](images/wb_db_connection.png "Workbench DB Connection")

6. Click on **Test Connection** to verify that you have configured the connection correctly. If you have issues check that you have provided the all of the details correctly. Once the test connection action succeeds click on **OK**.<br/>

7. Double click on the **connection name** to open a session against your MySQL database.<br/>

![DB connection](images/wb_open_connection.png "DB connection")

8. Choose **File > Run SQL Script**.<br/> 

![Run SQL Script](images/mysql-run-script.png)

9. Open the sql file from `C:\code\bikeshop\bikeshop.sql` and click the **Run**.<br/>

![Workbench run query](images/running_query_create_schema.png "Run Query in Workbench")

10. Once the script has been executed the database schema "bikestore" will have been created, click the **Close button**.<br/>

11. Click the refresh icon to see it listed in the set of availble schema.<br/>

![Schema created](images/workbench_schema_created.png "DB Schema Created")

12. Return to Azure portal and in the App Service blade click on the URL for the website.<br/>

![Open website](images/appservices.png)

13. The website should be up and running.<br/>

![Running website](images/running_website.png "Running website")

   
   
## Scenario 4: Scale up

Microsoft Azure offers a variety of pricing tiers for the Azure Database for MySQL service. In the earlier stages of this hands on lab, the lowest priced teir was used. In the real world, there may be times the application which relies upon your MySQL database will come under heavy load. In turn, this load may consume all of the resources available to your MySQL database instance, causing degraded performance. Scaling up your Azure Database for MySQL will help to noticeably improve performance in many of these cases, allowing you to keep processing your data efficiently.

Refer to these documentation articles for more infomation on [Pricing Tiers](https://docs.microsoft.com/en-us/azure/mysql/concepts-service-tiers) and [Compute Units](https://docs.microsoft.com/en-us/azure/mysql/concepts-compute-unit-and-storage) should you want to get more information on scaling your MySQL databases.

There are two options to scale up or scale down the Azure Database for MySQL service, either using the portal or via CLI.

### Using the Azure Portal

1. Open [https://portal.azure.com](https://portal.azure.com).<br/>

2. Open All Resources blade and click on the MySQL database that you want to scale.<br/>

3. Click **Pricing Tier** option.<br/>

 ![Pricing Tier](images/mysql-pricing-option.png)

4. Change the **Compute Units** up or down as need and click OK to save changes.<br/>
    > Note that currently only the Basic level at either 50 or 100 compute units is available  

![Compute units](images/mysql-compute.png)

### Using the Azure CLI 2.0

1. Using the PowerShell prompt that you have open and connected to the Azure subscription.<br/>

1. Using Azure CLI get information about the MySQL databases that you have access to:<br/>
```
 az mysql server list
```
1. The cli will provide a list of your MySQL databases, there should only be one. Note the resourceGroup and name values.<br/> 
![MySQL Server List](images/az-mysql-server-list.png)

1. On the command prompt, run below command to scale up to 100 compute units.<br/>
```
 az mysql  server update --resource-group [MyResourceGroupName] --name [mysqlDatabaseName] --compute-units 100
```
1. Run below command to scale down to 50 compute units.<br/>
```
az mysql  server update [MyResourceGroupName] --name [mysqlDatabaseName] --compute-units 50
```
 
    
# CONCLUSION
   
As a fully integrated service, Azure Database for MySQL plugs you into Microsoft’s global network of value-add services, datacenters, security features, and round-the-clock monitoring.  Azure Database for MySQL is designed to deliver highly available MySQL Compatibility at scale to the forefront, all backed by a best of class SLA.   Trust Azure to keep your enterprise-ready MySQL applications up and running.
