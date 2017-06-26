---
title: Manage Azure Data Lake Analytics using Azure .NET SDK | Microsoft Docs
description: 'Learn how to manage Data Lake Analytics jobs, data sources, users. '
services: data-lake-analytics
documentationcenter: ''
author: saveenr
manager: saveenr
editor: cgronlun

ms.assetid: 811d172d-9873-4ce9-a6d5-c1a26b374c79
ms.service: data-lake-analytics
ms.devlang: na
ms.topic: article
ms.tgt_pltfrm: na
ms.workload: big-data
ms.date: 06/18/2017
ms.author: saveenr

---
# Manage Azure Data Lake Analytics using Azure .NET SDK
[!INCLUDE [manage-selector](../../includes/data-lake-analytics-selector-manage.md)]

Learn how to manage Azure Data Lake Analytics accounts, data sources, users, and jobs using the Azure .NET SDK. 

## Prerequisites

* **Visual Studio 2015, Visual Studio 2013 update 4, or Visual Studio 2012 with Visual C++ Installed**.
* **Microsoft Azure SDK for .NET version 2.5 or above**.  Install it using the [Web platform installer](http://www.microsoft.com/web/downloads/platform.aspx).
* **Required NuGet Packages**

### Install NuGet packages

|Package|Version|
|-------|-------|
|[Microsoft.Rest.ClientRuntime.Azure.Authentication](https://www.nuget.org/packages/Microsoft.Rest.ClientRuntime.Azure.Authentication)| 2.3.1|
|[Microsoft.Azure.Management.DataLake.Analytics](https://www.nuget.org/packages/Microsoft.Azure.Management.DataLake.Analytics)|3.0.0|
|[Microsoft.Azure.Management.DataLake.Store](https://www.nuget.org/packages/Microsoft.Azure.Management.DataLake.Store)|2.2.0|
|[Microsoft.Azure.Management.ResourceManager](https://www.nuget.org/packages/Microsoft.Azure.Management.ResourceManager)|1.6.0-preview|
|[Microsoft.Azure.Graph.RBAC](https://www.nuget.org/packages/Microsoft.Azure.Management.ResourceManager)|3.4.0-preview|

You can install these packages via the NuGet command line with the following commands:

```
Install-Package -Id Microsoft.Rest.ClientRuntime.Azure.Authentication  -Version 2.3.1
Install-Package -Id Microsoft.Azure.Management.DataLake.Analytics  -Version 3.0.0
Install-Package -Id Microsoft.Azure.Management.DataLake.Store  -Version 2.2.0
Install-Package -Id Microsoft.Azure.Management.ResourceManager  -Version 1.6.0-preview
Install-Package -Id Microsoft.Azure.Graph.RBAC -Version 3.4.0-preview
```

## Common variables

```
string subid = "<Subscription ID>"; // a guid
string tenantid = "<Tenant ID>"; // tenantid. For example "contoso.inmicrosoft.com"
string rg == "<value>"; // resource  group name
string clientid = "1950a258-227b-4e31-a9cf-717495945fc2"; // Sample client ID (this will work, but you should pick your own)
```

## Authentication

You have multiple options for logging on to Azure Data Lake Analytics. The following snippet shows an example of authentication with interactive user authentication with a pop-up.

```
using System;
using System.IO;
using System.Threading;
using System.Security.Cryptography.X509Certificates;

using Microsoft.Rest;
using Microsoft.Rest.Azure.Authentication;
using Microsoft.Azure.Management.DataLake.Analytics;
using Microsoft.Azure.Management.DataLake.Analytics.Models;
using Microsoft.Azure.Management.DataLake.Store;
using Microsoft.Azure.Management.DataLake.Store.Models;
using Microsoft.IdentityModel.Clients.ActiveDirectory;
using Microsoft.Azure.Graph.RBAC;

public static Program
{
   public static string TENANT = "microsoft.onmicrosoft.com";
   public static string CLIENTID = "1950a258-227b-4e31-a9cf-717495945fc2";
   public static System.Uri ARM_TOKEN_AUDIENCE = new System.Uri( @"https://management.core.windows.net/");
   public static System.Uri ADL_TOKEN_AUDIENCE = new System.Uri( @"https://datalake.azure.net/" );
   public static System.Uri GRAPH_TOKEN_AUDIENCE = new System.Uri( @"https://graph.windows.net/" );

   static void Main(string[] args)
   {
      string MY_DOCUMENTS= System.Environment.GetFolderPath( System.Environment.SpecialFolder.MyDocuments);
      string TOKEN_CACHE_PATH = System.IO.Path.Combine(MY_DOCUMENTS, "my.tokencache");

      var tokenCache = GetTokenCache(TOKEN_CACHE_PATH);
      var armCreds = GetCreds_User_Popup(TENANT, ARM_TOKEN_AUDIENCE, CLIENTID, tokenCache);
      var adlCreds = GetCreds_User_Popup(TENANT, ADL_TOKEN_AUDIENCE, CLIENTID, tokenCache);
      var graphCreds = GetCreds_User_Popup(TENANT, GRAPH_TOKEN_AUDIENCE, CLIENTID, tokenCache);
   }
}
```

The source code for **GetCreds_User_Popup** and the code for other options for authentication are covered in [Data Lake Analytics .NET authentication options](https://github.com/Azure-Samples/data-lake-analytics-dotnet-auth-options)


## Create the client management objects

```
var resourceManagementClient = new ResourceManagementClient(armCreds) { SubscriptionId = subid };

var adlaAccountClient = new DataLakeAnalyticsAccountManagementClient(armCreds);
adlaAccountClient.SubscriptionId = subid;

var adlsAccountClient = new DataLakeStoreAccountManagementClient(armCreds);
adlsAccountClient.SubscriptionId = subid;

var adlaCatalogClient = new DataLakeAnalyticsCatalogManagementClient(adlCreds);
var adlaJobClient = new DataLakeAnalyticsJobManagementClient(adlCreds);

var adlsFileSystemClient = new DataLakeStoreFileSystemManagementClient(adlCreds);

var  graphClient = new GraphRbacManagementClient(graphCreds);
graphClient.TenantID = domain;

```


## Manage accounts

### Create an Azure Resource Group

If you haven't already created one, you must have an Azure Resource Group to create your Data Lake Analytics components. You will need your authentication credentials, subscription ID, and a location. The following code shows how to create a resource group:

```
var resourceGroup = new ResourceGroup { Location = location };
resourceManagementClient.ResourceGroups.CreateOrUpdate(groupName, rg);
```
For more information, see [Azure Resource Groups and Data Lake Analytics](#Azure-Resource-Groups-and-Data-Lake-Analytics).

### Create a Data Lake Store account

Ever ADLA account requires an ADLS account. If you don't already have one to use, you can create one with the following code:

```
var new_adls_params = new DataLakeStoreAccount(location: _location);
adlsAccountClient.Account.Create(rg, adls, new_adls_params);
```

### Create a Data Lake Analytics account

The following code creates an ADLS account

```
var new_adla_params = new DataLakeAnalyticsAccount()
{
   DefaultDataLakeStoreAccount = adls,
   Location = location
};

adlaClient.Account.Create(rg, adla, new_adla_params);
```

### List Data Lake Store accounts

```
var adlsAccounts = adlsClient.Account.List().ToList();
foreach (var adls in adlsAccounts)
{
   Console.WriteLine("ADLS: {0}", adls.Name);
}
```

### List Data Lake Analytics accounts

```
var adlaAccounts = adlaClient.Account.List().ToList();
for (var adla in AdlaAccounts)
{
   Console.WriteLine("ADLA: {0}, adla.Name");
}
```

### Checking if an account exists

```
bool exists = adlaClient.Account.Exists(rg, adla));
```

### Get information about an account

```
bool exists = adlaClient.Account.Exists(rg, adla));
if (exists)
{
   var adla_accnt = adlaClient.Account.Get(rg, adla);
}
```

### Delete an account

```
if (adlaClient.Account.Exists(rg, adla))
{
   adlaClient.Account.Delete(rg, adla);
}
```

### Get the default Data Lake Store account

Every Data Lake Analytics account requires a default Data Lake Store account. Use this code to determine the default Store account for an Analytics account.

```
if (adlaClient.Account.Exists(rg, adla))
{
  var adla_accnt = adlaClient.Account.Get(rg, adla);
  string def_adls_account = adla_accnt.DefaultDataLakeStoreAccount;
}
```

## Manage data sources

Data Lake Analytics currently supports the following data sources:

* [Azure Data Lake Store](../data-lake-store/data-lake-store-overview.md)
* [Azure Storage Account](../storage/storage-introduction.md)

### Link to an Azure Storage account

You can create links to Azure Storage accounts.

```
string storage_key = "xxxxxxxxxxxxxxxxxxxx";
string storage_account = "mystorageaccount";
var addParams = new AddStorageAccountParameters(storage_key);            
adlaClient.StorageAccounts.Add(rg, adla, storage_account, addParams);
```

### List Azure Storage data sources

```
var stg_accounts = adlaClient.StorageAccounts.ListByAccount(rg, adla);

if (stg_accounts != null)
{
  foreach (var stg_account in stg_accounts)
  {
      Console.WriteLine("Storage account: {0}", stg_account.Name);
  }
}
```

### List Data Lake Store data sources

```
var adls_accounts = adlsClient.Account.List();
if (adls_accounts != null)
{
  foreach (var adls_accnt in adls_accounts)
  {
      Console.WriteLine("ADLS account: {0}", adls_accnt.Name);
  }
}
```

### Upload and download folders and files
You can use the Data Lake Store file system client management object to upload and download individual files or folders from Azure to your local computer, using the following methods:

- UploadFolder
- UploadFile
- DownloadFolder
- DownloadFile

The first parameter for these methods is the name of the Data Lake Store Account, followed by parameters for the source path and the destination path.

The following example shows how to download a folder in the Data Lake Store.


```
adlsFileSystemClient.FileSystem.DownloadFolder(adls, sourcePath, destinationPath);
```


### Create a file in a Data Lake Store account

```
using (var memstream = new MemoryStream())
{
   using (var sw = new StreamWriter(memstream, UTF8Encoding.UTF8))
   {

      sw.WriteLine("Hello World");
      sw.Flush();

      adlsFileSystemClient.FileSystem.Create(adls, "/Samples/Output/randombytes.csv", memstream);
   }
}
```

### Verify Azure Storage account paths
The following code checks if an Azure Storage account (storageAccntName) exists in a Data Lake Analytics account (analyticsAccountName), and if a container (containerName) exists in the Azure Storage account.

```
string storage_account = "mystorageaccount";
string storage_container = "mycontainer";
bool accountExists = adlaClient.Account.StorageAccountExists(rg, adla, storage_account));
bool containerExists = adlaClient.Account.StorageContainerExists(rg, adla, storage_account, storage_container));
```

## Manage catalog and jobs
The DataLakeAnalyticsCatalogManagementClient object provides methods for managing the SQL database provided for each Azure Data Lake Store. The DataLakeAnalyticsJobManagementClient provides methods to submit and manage jobs run on the database with U-SQL scripts.

### List databases and schemas
Among the several things, you can list, the most common are databases and their schema. The following code obtains a collection of databases, and then enumerates the schema for each database.

```
var databases = adlaCatalogClient.Catalog.ListDatabases(adla);
foreach (var db in databases)
{
  Console.WriteLine($"Database: {db.Name}");
  Console.WriteLine(" - Schemas:");
  var schemas = adlaCatalogClient.Catalog.ListSchemas(adla, db.Name);
  foreach (var schm in schemas)
  {
      Console.WriteLine($"\t{schm.Name}");
  }
}
```

### List table columns
The following code shows how to access the database with a Data Lake Analytics Catalog management client to list the columns in a specified table.

```
var tbl = adlaCatalogClient.Catalog.GetTable(adla, "master", "dbo", "MyTableName");
IEnumerable<USqlTableColumn> columns = tbl.ColumnList;

foreach (USqlTableColumn utc in columns)
{
  string scriptPath = "/Samples/Scripts/SearchResults_Wikipedia_Script.txt";
  Stream scriptStrm = adlsFileSystemClient.FileSystem.Open(_adlsAccountName, scriptPath);
  string scriptTxt = string.Empty;
  using (StreamReader sr = new StreamReader(scriptStrm))
  {
      scriptTxt = sr.ReadToEnd();
  }

  var jobName = "SR_Wikipedia";
  var jobId = Guid.NewGuid();
  var properties = new USqlJobProperties(scriptTxt);
  var parameters = new JobInformation(jobName, JobType.USql, properties, priority: 1, degreeOfParallelism: 1, jobId: jobId);
  var jobInfo = adlaJobClient.Job.Create(_adlaAnalyticsAccountTest, jobId, parameters);
  Console.WriteLine($"Job {jobName} submitted.");

}
```

### List failed jobs
The following code lists information about jobs that failed.

```
var odq = new ODataQuery<JobInformation> { Filter = "result eq 'Failed'" };
var jobs = adlaJobClient.Job.List(adlaClient, odq );
foreach (var j in jobs)
{
   Console.WriteLine($"{j.Name}\t{j.JobId}\t{j.Type}\t{j.StartTime}\t{j.EndTime}");
}
```

## Common graph scenarios

### Look up user in the AAD directory

```
var userinfo = graphClient.Users.Get( "bill@contoso.com" );
```

### Get the ObjectId of a user in the AAD directory

```
var userinfo = graphClient.Users.Get( "bill@contoso.com" );
Console.WriteLine( userinfo.ObjectId )
```

## Manage compute policies
The DataLakeAnalyticsAccountManagementClient object provides methods for managing the compute policies for a Data Lake Analytics account.

### List compute policies
The following code retrieves a list of compute policies for a Data Lake Analytics account.

```
var policies = adlaAccountClient.ComputePolicies.ListByAccount(rg, adla);
foreach (var p in policies)
{
   Console.WriteLine($"Name: {p.Name}\tType: {p.ObjectType}\tMax AUs / job: {p.MaxDegreeOfParallelismPerJob}\tMin priority / job: {p.MinPriorityPerJob}");
}
```

### Create a new compute policy
The following code creates a new compute policy for a Data Lake Analytics account, setting the maximum AUs available to the specified user to 50, and the minimum job priority to 250.

```
var userAadObjectId = "3b097601-4912-4d41-b9d2-78672fc2acde";
var newPolicyParams = new ComputePolicyCreateOrUpdateParameters(userAadObjectId, "User", 50, 250);
adlaAccountClient.ComputePolicies.CreateOrUpdate(rg, adla, "GaryMcDaniel", newPolicyParams);
```

## Next steps
* [Overview of Microsoft Azure Data Lake Analytics](data-lake-analytics-overview.md)
* [Manage Azure Data Lake Analytics using Azure portal](data-lake-analytics-manage-use-portal.md)
* [Monitor and troubleshoot Azure Data Lake Analytics jobs using Azure portal](data-lake-analytics-monitor-and-troubleshoot-jobs-tutorial.md)
