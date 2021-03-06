<properties 
   pageTitle="Get started with Data Lake Store using REST API| Microsoft Azure" 
   description="Use WebHDFS REST APIs to perform operations on Data Lake Store" 
   services="data-lake-store" 
   documentationCenter="" 
   authors="nitinme" 
   manager="paulettm" 
   editor="cgronlun"/>
 
<tags
   ms.service="data-lake-store"
   ms.devlang="na"
   ms.topic="article"
   ms.tgt_pltfrm="na"
   ms.workload="big-data" 
   ms.date="04/07/2016"
   ms.author="nitinme"/>

# Get started with Azure Data Lake Store using REST APIs

> [AZURE.SELECTOR]
- [Portal](data-lake-store-get-started-portal.md)
- [PowerShell](data-lake-store-get-started-powershell.md)
- [.NET SDK](data-lake-store-get-started-net-sdk.md)
- [Java SDK](data-lake-store-get-started-java-sdk.md)
- [REST API](data-lake-store-get-started-rest-api.md)
- [Azure CLI](data-lake-store-get-started-cli.md)
- [Node.js](data-lake-store-manage-use-nodejs.md)

In this article, you will learn how to use WebHDFS REST APIs and Data Lake Store REST APIs to perform account management as well as filesystem operations on Azure Data Lake Store. Azure Data Lake Store exposes its own REST APIs for account management operations. However, because Data Lake Store is compatible with HDFS and Hadoop ecosystem, it supports using WebHDFS REST APIs for filesystem operations.

>[AZURE.NOTE] For detailed information on the REST API support for Data Lake Store, see [Azure Data Lake Store REST API Reference](https://msdn.microsoft.com/library/mt693424.aspx).

## Prerequisites

- **An Azure subscription**. See [Get Azure free trial](https://azure.microsoft.com/pricing/free-trial/).
- **Enable your Azure subscription** for Data Lake Store public preview. See [instructions](data-lake-store-get-started-portal.md#signup).
- **Create an Azure Active Directory Application**. See [Create Active Directory application and service principal using portal](../resource-group-create-service-principal-portal.md). Once you have created the application, retrieve the following values related to the application.
	- Get client ID and tenant ID for the application.
	- Create an authentication key
	- Set delegated permissions

	Instructions on how to retrieve these values are available in the link provided above.
- **Assign the Azure Active Directory application to a role**. The role can be at the level of the scope at which you want to give permission to the Azure Active Directory application. For example, you can assign the application at the subscription level or at the level of a resource group. For instructions, see [Assign application to a role](../resource-group-create-service-principal-portal.md#assign-application-to-role).
- [cURL](http://curl.haxx.se/). This article uses cURL to demonstrate how to make REST API calls against a Data Lake Store account.

## How do I authenticate using Azure Active Directory?

You can use two approaches to authenticate using Azure Active Directory.

* **Interactive**, where the application prompts the user to log in. For more information, see [Authorization code grant flow](https://msdn.microsoft.com/library/azure/dn645542.aspx).

* **Non-interactive**, where the application provides its own credentials. For more information, see [Service to service calls using credentials](https://msdn.microsoft.com/library/azure/dn645543.aspx).

This article uses **non-interactive** approach. For this, you must issue a POST request like the one shown below. 

	curl -X POST https://login.microsoftonline.com/<TENANT-ID>/oauth2/token  \
      -F grant_type=client_credentials \
      -F resource=https://management.core.windows.net/ \
      -F client_id=<CLIENT-ID> \
      -F client_secret=<AUTH-KEY>

The output of this request will include an authorization token (denoted by `access-token` in the output below) that you will subsequently pass with your REST API calls. Save this authentication token in a text file; you will need this later in this article.

	{"token_type":"Bearer","expires_in":"3599","expires_on":"1458245447","not_before":"1458241547","resource":"https://management.core.windows.net/","access_token":"<REDACTED>"}

## Create a Data Lake Store account

This operation is based on the REST API call defined [here](https://msdn.microsoft.com/library/mt694078.aspx).

Use the following cURL command:

	curl -i -X PUT -H "Authorization: Bearer <REDACTED>" -H "Content-Type: application/json" https://management.azure.com/subscriptions/{subscription-id}/resourceGroups/{resource-group-name}/providers/Microsoft.DataLakeStore/accounts/nitinadlstore?api-version=2015-10-01-preview -d@C:\temp\input.json

In the above command, replace \<`REDACTED`\> with the authorization token you retrieved earlier. The request payload for this command is contained in the **input.json** file that is provided for the `-d` parameter above. The contents of the input.json file resemble the following:

	{
	"location": "eastus2",
	"tags": {
		"department": "finance"
		},
	"properties": {}
	}	

## Create folders in a Data Lake Store account

This operation is based on the WebHDFS REST API call defined [here](http://hadoop.apache.org/docs/stable/hadoop-project-dist/hadoop-hdfs/WebHDFS.html#Make_a_Directory).

Use the following cURL command:

	curl -i -X PUT -H "Authorization: Bearer <REDACTED>" -d "" https://nitinadlstore.azuredatalakestore.net/webhdfs/v1/mytempdir/?op=MKDIRS

In the above command, replace \<`REDACTED`\> with the authorization token you retrieved earlier. This command creates a directory called **mytempdir** under the root folder of your Data Lake Store account.

You should see a response like this if the operation completes successfully:

	{"boolean":true}

## List folders in a Data Lake Store account

This operation is based on the WebHDFS REST API call defined [here](http://hadoop.apache.org/docs/stable/hadoop-project-dist/hadoop-hdfs/WebHDFS.html#List_a_Directory).

Use the following cURL command:

	curl -i -X GET -H "Authorization: Bearer <REDACTED>" https://nitinadlstore.azuredatalakestore.net/webhdfs/v1/?op=LISTSTATUS

In the above command, replace \<`REDACTED`\> with the authorization token you retrieved earlier.

You should see a response like this if the operation completes successfully:

	{
	"FileStatuses": {
		"FileStatus": [{
			"length": 0,
			"pathSuffix": "mytempdir",
			"type": "DIRECTORY",
			"blockSize": 268435456,
			"accessTime": 1458324719512,
			"modificationTime": 1458324719512,
			"replication": 0,
			"permission": "777",
			"owner": "NotSupportYet",
			"group": "NotSupportYet"
		}]
	}
	}

## Upload data into a Data Lake Store account

This operation is based on the WebHDFS REST API call defined [here](http://hadoop.apache.org/docs/stable/hadoop-project-dist/hadoop-hdfs/WebHDFS.html#Create_and_Write_to_a_File).

Uploading data using the WebHDFS REST API is a two-step process, as explained below.

1. Submit a HTTP PUT request without sending the file data to be uploaded.

		curl -i -X PUT -H "Authorization: Bearer <REDACTED>" -d "" https://nitinadlstore.azuredatalakestore.net/webhdfs/v1/mytempdir/?op=CREATE

	The output for this command will be contain a temporary redirect URL, like the one shown below.

		HTTP/1.1 100 Continue

		HTTP/1.1 307 Temporary Redirect
		...
		...
		Location: https://nitinadlstore.azuredatalakestore.net/webhdfs/v1/mytempdir/somerandomfile.txt?op=CREATE&write=true
		...
		...

2. You must now submit another HTTP PUT request against the URL listed for the **Location** property in the response.

		curl -i -X PUT -T myinputfile.txt -H "Authorization: Bearer <REDACTED>" https://nitinadlstore.azuredatalakestore.net/webhdfs/v1/mytempdir/myinputfile.txt?op=CREATE&write=true

	The output will be similar to the following:

		HTTP/1.1 100 Continue

		HTTP/1.1 201 Created
		...
		...

## Read data from a Data Lake Store account

This operation is based on the WebHDFS REST API call defined [here](http://hadoop.apache.org/docs/stable/hadoop-project-dist/hadoop-hdfs/WebHDFS.html#Open_and_Read_a_File).

Reading data from a Data Lake Store account is a two-step process.

* You first submit a GET request against the endpoint `https://nitinadlstore.azuredatalakestore.net/webhdfs/v1/mytempdir/myinputfile.txt?op=OPEN`. This will return a location to submit the next GET request to.
* You then submit the GET request against the endpoint `https://nitinadlstore.azuredatalakestore.net/webhdfs/v1/mytempdir/myinputfile.txt?op=OPEN&read=true`. This will display the contents of the file.

However, because there is no difference in the input parameters between the first and the second step, you can use the `-L` parameter to submit the first request. `-L` option essentially combines two requests into one and will make cURL redo the request on the new location. Finally, the output from all the request calls is displayed, like shown below.

	curl -i -L GET -H "Authorization: Bearer <REDACTED>" https://nitinadlstore.azuredatalakestore.net/webhdfs/v1/mytempdir/myinputfile.txt?op=OPEN

You should see an output similar to the following:

	HTTP/1.1 307 Temporary Redirect
	...
	Location: https://nitinadlstore.azuredatalakestore.net/webhdfs/v1/mytempdir/somerandomfile.txt?op=OPEN&read=true
	...
	
	HTTP/1.1 200 OK
	...
	
	Hello, Data Lake Store user!

## Rename a file in a Data Lake Store account

This operation is based on the WebHDFS REST API call defined [here](http://hadoop.apache.org/docs/stable/hadoop-project-dist/hadoop-hdfs/WebHDFS.html#Rename_a_FileDirectory).

Use the following cURL command to rename a file.

	curl -i -X PUT -H "Authorization: Bearer <REDACTED>" -d "" https://nitinadlstore.azuredatalakestore.net/webhdfs/v1/mytempdir/myinputfile.txt?op=RENAME&destination=/mytempdir/myinputfile1.txt

You should see an output similar to the following:

	HTTP/1.1 200 OK
	...
	
	{"boolean":true}

## Delete a file from a Data Lake Store account

This operation is based on the WebHDFS REST API call defined [here](http://hadoop.apache.org/docs/stable/hadoop-project-dist/hadoop-hdfs/WebHDFS.html#Delete_a_FileDirectory).

Use the following cURL command to delete a file.

	curl -i -X DELETE -H "Authorization: Bearer <REDACTED>" https://nitinadlstore.azuredatalakestore.net/webhdfs/v1/mytempdir/myinputfile1.txt?op=DELETE

You should see an output like the following:

	HTTP/1.1 200 OK
	...
	
	{"boolean":true}

## Delete a Data Lake Store account

This operation is based on the REST API call defined [here](https://msdn.microsoft.com/library/mt694075.aspx).

Use the following cURL command to delete a Data Lake Store account.

	curl -i -X DELETE -H "Authorization: Bearer <REDACTED>" https://management.azure.com/subscriptions/{subscription-id}/resourceGroups/{resource-group-name}/providers/Microsoft.DataLakeStore/accounts/nitinadlstore?api-version=2015-10-01-preview

You should see an output like the following:

	HTTP/1.1 200 OK
	...
	...

## See also

- [Open Source Big Data applications compatible with Azure Data Lake Store](data-lake-store-compatible-oss-other-applications.md)
 
