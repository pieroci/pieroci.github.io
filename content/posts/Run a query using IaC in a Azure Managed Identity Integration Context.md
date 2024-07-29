---
title: "Automating Managed Identity Integration for SQL Server with Terraform and Azure DevOps"
description: "Implementing an Azure Sql Server comunication with managed identity and IaC"
summary: If your security requirements mandate integrating managed identities on your Azure resources with a SQL Server, achieving this with Terraform can be challenging due to limitations reported in the Microsoft's documentation... but let me show you how can achieve that!
date: 2024-07-23
tags: [azure, terraform, sqlserver, managedidentity, Azure Devops,  run query with terraform, powershell and Terraform]
categories: ["terraform", "DevOps"]
series: ["Terraform, Devops"]
author: ["Pieroci"]
ShowToc: false
TocOpen: false
draft: true
weight: 5
cover:
    image: "../../assets/img/8rpnb0ns.png" # image path/url
    alt: "Automating Managed Identity Integration for SQL Server with Terraform and Azure DevOps" 
    caption: "Automating Managed Identity Integration for SQL Server with Terraform and Azure DevOps"
    relative: true # when using page bundles set this to true
    hidden: true # only hide on current single page
editPost:
    URL: "https://github.com/pieroci/content"
    Text: "Suggest Changes" # edit text
    appendFilePath: true # to append file path to Edit link
---

Managing an infrastructure with IaC can be very convenient.  

IaC can enhance your productivity through automation, reduce manual errors, eliminate configuration drift and have many more benefits. 

In some cases, it can be hard to automate certain tasks, and we prefer to make a mnemonic effort rather than automating every single step. 

> A good example can be: *run a query with terraform*. 

Yes, it’s possible to run a query in terraform and in some scenarios it’s not only a devops duty but it’s required! 

If your security requirements mandate integrating managed identities on your Azure resources with a SQL Server, achieving this with Terraform can be challenging due to limitations reported in the Microsoft's documentation https://learn.microsoft.com/en-us/sql/relational-databases/security/authentication-access/azure-ad-authentication-sql-server-overview?view=sql-server-ver16#azure-active-directory-managed-identity .  


In this tutorial we will use the following technologies:  

- Terraform 
- Azure DevOps Pipelines 
- Powershell

**Some context:** 
Our target is to allow a resource (an azure function app) to query our sql server database in a complex security scenario, in which the authentication between our client and the database is only allowed via [managed identity](https://learn.microsoft.com/en-us/entra/identity/managed-identities-azure-resources/overview). 

>
> Prerequisites: 
>
> - For this tutorial, I already created the “sql-admin” Entra ID Group: this group contains who I want to be the Sql Server Administrator. You must add your Service Connection’s Service Principal ID to this group, to enable launching a CREATE USER/LOGIN queries from your Azure DevOps pipelines.  In Short: if your sqlServer is setted with the Ms Entra Administrator , only an Admin can add users to dbs then this step it's required in this article. After the terraform apply you'll see the Entra group in your sql server in portal configurations like the below image. 
>
>  ![image-20240729094905330](C:\Users\piero.ciula\AppData\Roaming\Typora\typora-user-images\image-20240729094905330.png)
>
> - I also created the “sql-reader” Group, with Directory Reader role. You should add to this group all the Azure SQL Server resources that need to use the external Entra ID identity provider. (https://learn.microsoft.com/en-us/azure/azure-sql/database/authentication-aad-directory-readers-role?view=azuresql#assigning-the-directory-readers-role ). In short: A “FROM EXTERNAL PROVIDER” query indicates to SQL Server it must search the user’s identity in the Microsoft Entra directories. But your Sql Server need reader privilege on Entra Directory. 

 

**Let’s start coding!** 

We are going to use Terraform to provision a function app with system assigned managed identity: 

```terraform {linenos=true}

resource "azurerm_resource_group" "myrg" { 

 name   = "myrg" 

 location = "West Europe" 

} 

resource "azurerm_storage_account" "mystorage" { 

 name              = "mystorage" 

 resource_group_name       = azurerm_resource_group.myrg.name 

 location            = azurerm_resource_group.myrg.location 

 account_tier          = "Standard" 

 account_replication_type    = "LRS" 

 allow_nested_items_to_be_public = false 

 public_network_access_enabled  = false 

} 

resource "azurerm_service_plan" "myappsrvplan" { 

 name        = "myappsrvplan" 

 location      = azurerm_resource_group.myrg.location 

 resource_group_name = azurerm_resource_group.myrg.name 

 os_type       = "Linux" 

 sku_name      = "P1v3" 

} 

resource "azurerm_linux_function_app" "myfunctionapp" { 

 name        = "myfunctionapp" 

 location      = azurerm_resource_group.myrg.location 

 resource_group_name = azurerm_resource_group.myrg.name 

 storage_account_name     = azurerm_storage_account.mystorage.name 

 storage_uses_managed_identity = true 

 service_plan_id        = azurerm_service_plan.myappsrvplan.id 

 public_network_access_enabled = false 

 https_only          = true 

 site_config { 

  vnet_route_all_enabled = true 

 } 

 identity { 

  type = "SystemAssigned" 

 } 

} 
```



*Now, let’s provision SQL Server and a SQL Database*

```terraform {linenos=true}

resource "random_password" "generapassrandom" { 

 length = 32 

 special = true 

} 

 

resource "azurerm_mssql_server" "mssql" { 

 name             = "mssql" 

 location      = azurerm_resource_group.myrg.location 

 resource_group_name = azurerm_resource_group.myrg.name 

 administrator_login      = "admin" 

 administrator_login_password = random_password.generapassrandom.result 

 minimum_tls_version      = "1.2" 

 version            = "12.0" 

 public_network_access_enabled = false 

 azuread_administrator { 

  login_username = "sql-admins"            #The login username of the Azure AD Administrator of this SQL Server 

  object_id   = "object-id-of-sql-admins-group" #The object id of the Azure AD Administrator of this SQL Server 

 } 

 identity { 

  type = "SystemAssigned" 

 } 

} 

 

resource "azurerm_mssql_database" "mydb" { 

 name     = "mydb" 

 server_id  = azurerm_mssql_server.mssql.id 

 max_size_gb = 30 

 sku_name   = "P1" 

 license_type = "LicenseIncluded" 

} 

```

*At this point, we will try to assign a role to our function app to the Sql Server... but without success.* 

```terraform {linenos=true}
resource "azurerm_role_assignment" "role_mssql_to_myfunctionapp" { 

 scope        = azurerm_mssql_server.mssql.id 

 role_definition_name = "Contributor" 

 principal_id     = azurerm_linux_function_app.myfunctionapp.identity[0].principal_id 

} 
```

*That's it! Unfortunately, assigning the role to our Function App towards a SQL Server is not enough to allow the Function App to be able to authenticate to our SQL Server. See MSFT’s documentation for more information*: https://learn.microsoft.com/en-us/azure/azure-sql/database/authentication-azure-ad-logins-tutorial?view=azuresql#create-microsoft-entra-login . 

 

 

*The right way to achieve this is to launch a query in our database as follows:* 
```sql
CREATE LOGIN [myfunctionapp] FROM EXTERNAL PROVIDER;` 

ALTER ROLE db_datareader ADD MEMBER [myfunctionapp];` 

ALTER ROLE db_datawriter ADD MEMBER [myfunctionapp];` 
```


*It's annoying to manually launch this query every time we need to allow managed identity authentication towards a SQL Server.* 

*Let's automate this with Terraform and Azure DevOps Pipelines!* 

Create a task like the following one on your DevOps Pipeline: 
```powershell
  - task: AzurePowerShell@5
    displayName: Get Access Token
    inputs:
      azureSubscription: $(SERVICE_CONNECTION_NAME)
      ScriptType: 'InlineScript'
      azurePowerShellVersion: 'LatestVersion'
      preferredAzurePowerShellVersion: '3.1.0'
      Inline: |
        $VerbosePreference = 'SilentlyContinue'
        $DebugPreference = 'SilentlyContinue'
        $ErrorActionPreference = 'Stop'
        $requiredModules = 'Az'
        foreach ($module in $requiredModules) {
            if (-not (Get-Module -ListAvailable -Name $module -ErrorAction Stop)) {
                Install-Module -Name $module -Repository PSGallery -Force -AllowClobber -ErrorAction Stop
            }
            else 
            {
                # Write-Output "Module $module already installed..." 
            }
            
            Import-Module $module -ErrorAction Stop

            $modulePath = (Get-Module -Name $module -ErrorAction Stop).ModuleBase

            $env:PSModulePath = $env:PSModulePath + ":$modulePath"
        }
        $access_token = (Get-AzAccessToken -ResourceUrl https://database.windows.net -ErrorAction Stop).Token
        Write-Host "##vso[task.setvariable variable=TF_VAR_ACCESS_TOKEN]$access_token"

```



*This script allows you to get the Access token with SQL Database scope using our Service Connection’s Principal Id. The access token will be saved in a TF_VAR_ACCESS_TOKEN pipeline variable.* 

Pass the value to the terraform apply DevOps pipeline task like a TF_VAR environment variable: 

```yaml
            - task: TerraformTaskV3@3
              displayName: Terraform APPLY
              inputs:
                provider: 'azurerm'
                command: 'apply'
                workingDirectory: $(working_dir)
                commandOptions: "planfile"
                environmentServiceNameAzureRM: $(SERVICE_CONNECTION_NAME)
              env:
                TF_VAR_ACCESS_TOKEN: $(TF_VAR_ACCESS_TOKEN)

```


> On your terraform code, you must expose the ACCESS_TOKEN variables and will automatically filled at runtime when you run your pipeline with the TF_VAR_ACCESS_TOKEN value.

```yaml
variable "ACCESS_TOKEN" { 

  type = string 

} 
```

*Now we can create a reusable Terraform module (run_sql_query) to run sql queries, and a powershell script.* 



###### *terraform\run_sql_query\powershell_templates\ sqlcmd.ps1: *

```powershell 

param([String]$AccessToken = "AccessToken") 

try{ 

  $requiredModules = 'Az.Accounts', 'SqlServer' 

  foreach ($module in $requiredModules) { 

    if (-not (Get-Module -ListAvailable -Name $module)) { 

      Write-Output "Installing module $module ..." 

      Install-Module -Name $module -Force -AllowClobber 

      Write-Output "Module $module installed correctly..." 

    } 

    else 

    { 

      Write-Output "Module $module already installed..." 

    } 

    Import-Module $module 

 

    $modulePath = (Get-Module -Name $module).ModuleBase 

 

    $env:PSModulePath = $env:PSModulePath + ":$modulePath" 

  } 

  $serverName = '${serverName}' 

  $databaseName = '${databaseName}' 

  $query = '${query}' 

  Write-Host "${serverName} - ${databaseName}: Executing '${query}'" 

  Invoke-Sqlcmd -ServerInstance "${serverName}.database.windows.net" -Database $databaseName -AccessToken $Access_token -Query $query -IncludeSqlUserErrors 

  exit 0 

} 

catch { 

  Write-Error $_ 

  exit 1 

} 
```



*Yeah! Terraform can’t launch queries but PowerShell can do it with the Invoke-Sqlcmd cmdlet.* 

**The trick is to add a PowerShell script to your Terraform code and run it using a local-exec provider!**  

###### *terraform\run_sql_query\ main.tf* 

```terraform {linenos=true}

locals { 

 unique_suffix = "${random_uuid.randomuuid.result}" 

 temp_dir = "${path.module}/temp/${local.unique_suffix}" 

 sqlcmdtemplate = templatefile("${path.module}/powershell_templates/sqlcmd.ps1", { 

  serverName  = var.sql_server_name, 

  databaseName = var.sql_database_name, 

  query    = var.query_string_to_exec # local.querytemplate 

 }) 

} 

 

resource "random_uuid" "randomuuid" { 

} 

resource "local_file" "create_temp_dir" { 

 content = "" 

 filename = "${local.temp_dir}/.keep" 

} 

resource "local_file" "sqlcommandpshellscript" { 

 content = local.sqlcmdtemplate 

 filename = "${local.temp_dir}/sqlcmd.ps1" 

 depends_on = [local_file.create_temp_dir] 

} 

resource "null_resource" "database_setup_local_exec" { 

 provisioner "local-exec" { 

  command = "pwsh -File ./${local_file.sqlcommandpshellscript.filename} -AccessToken ${var.ACCESS_TOKEN}" 

  environment = { 

   ACCESS_TOKEN = var.ACCESS_TOKEN 

  } 

 } 

 triggers = { 

  fileId = var.triggerId # join(",", [local_file.sqlscript.content_md5, local_file.sqlcommandpshellscript.content_md5]) 

 } 

} 
```





###### *terraform\run_sql_query\ variables.tf* 

```terraform {linenos=true}
variable "ACCESS_TOKEN" { 

 type = string 

} 

variable "sql_server_name"{ 

 type = string 

} 

variable "sql_database_name"{ 

 type = string 

} 

variable "query_string_to_exec" { 

  type = string 

} 

variable "triggerId" { 

  type = string 

  default = "" 

} 
```



Now invoke your module from your terraform main.tf file.

###### *terraform\main.tf* 

```terraform {linenos=true}
locals{ 

 query = "CREATE LOGIN [myfunctionapp] FROM EXTERNAL PROVIDER; ALTER ROLE db_datareader ADD MEMBER [myfunctionapp]; ALTER ROLE db_datawriter ADD MEMBER [myfunctionapp];" 

} 

module "run_query" { 

 source = "/run_sql_query" 

 ACCESS_TOKEN = var.ACCESS_TOKEN 

 sql_database_name = azurerm_mssql_database.mydb.name 

 sql_server_name = azurerm_mssql_server.mssql.name 

 query_string_to_exec = local.query 

 triggerId = local.query 

} 
```

The code I used in the article can be found here: https://github.com/pieroci/devopsncode/tree/main/azure_run_sql_query_man_identity 

This code could be improved, and has some Open Points and limitations due to the use of Terraform *local_file* resources, which have some quirks. 

> Notably, local_file resources produce noise in subsequent terraform runs, as described in the official local provider’s documentation here: https://registry.terraform.io/providers/hashicorp/local/latest/docs/resources/file  

This brings a little chain problem: 

On your next terraform pipeline run, for the problem i mentioned above , Terraform logs show you a message: Hey! This resource already exists ( Don’t worry , the terraform apply doesn't fail as well!). 

![img](data:image/png;base64,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)

![img](data:image/png;base64,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)

Another security improvement could be treating the “AccessToken” variable as a secret in Azure DevOps, and using the sensitive flag in Terraform to avoid it being output in your apply log (beware though, depending on your usage it may remain visible in your agent’s pipeline logs). 

Maybe I'll make these improvements and search for a new strategy in a future article... 

*For now: enjoy automation laziness and go grab a cup of coffee (or a beer, if you’re into that)!* 