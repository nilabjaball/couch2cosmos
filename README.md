# Couch 2 Cosmos

## Couchbase to  ComosDB Migration 

Cosmos DB is an Azure  Nosql service that is globally distributed, horizontally scalable, multi-model supporting key-value, managed Document, Graph and Columnar. 

### Method 1 

Azure Data Factory 
The Azure Data Factory (ADF) service is a fully managed service for composing data storage, processing, and movement services into streamlined, scalable, and reliable data production pipelines.  
With Data Factory, you can use the Copy Activity in a data pipeline to move data from  Couchbase to cosmosDB SQL API data store.
                                                                          


For data migration, you need the following steps 
•	Create an Azure Data Factory
•	Create a Self-Hosted Integration Runtime (SHIR)
•	Create linked services for Couchbase and CosmosDB 
•	Create dataset for Couchbase and CosmosDB
•	Finally, a pipeline with a copy activity to copy the Couchbase data to Cosmos DB 

Azure Data Factory 
An Azure subscription might have one or more Azure Data Factory instances. Using Azure Data Factory, you can create and schedule data-driven workflows (called pipelines) that can ingest data from disparate data stores. To create Azure Data Factory, follow the instructions in QuickStart module
https://docs.microsoft.com/en-us/azure/data-factory/quickstart-create-data-factory-portal


Self-Hosted Integration Runtime (SHIR)
The integration runtime (IR) is the compute infrastructure that Azure Data Factory uses to provide data-integration capabilities across different network environments.
A self-hosted integration runtime can run copy activities between a cloud data store and a data store in a private network, and it can dispatch transform activities against compute resources in an on-premises network or an Azure virtual network. The installation of a self-hosted integration runtime needs on an on-premises machine or a virtual machine (VM) inside a private network.

The steps to install and configure SHIR is detailed in this article
https://docs.microsoft.com/en-us/azure/data-factory/create-self-hosted-integration-runtime
 
For data migration, you need to host an additional VM  either in the same private network or in the on-premise data center with access to the Couchbase database.  In addition, you need to open 8091 and 9093 port between the SHIR VM and the Couchbase server. If you are running on a cluster, it is suggested to open at least on the cluster virtual IP or one of the cluster node.

Linked Services 

Linked services are much like connection strings, which define the connection information that's needed for Data Factory to connect to external resources.
For data migration, a Couchbase linked services and Cosmos DB SQL API needs to be created. 
•	After the creation is complete, you see the Data Factory page. Select the Author & Monitor tile to start the Azure Data Factory user interface (UI) application on a separate tab.
•	Go to connections and click on the code icon     
 
•	Use the below json to create the couchbase linked service

```

{
    "name": "<Linked Service Name>",
    "type": "Microsoft.DataFactory/factories/linkedservices",
    "properties": {
        "type": "Couchbase",
        "typeProperties": {
            "connectionString": {
                "type": "SecureString",
                "value": "Server=<server>; Port=<port>;AuthMech=1;CredString=[{\"user\": \"<user name>\", \"pass\":\"<password>\"}]"
            }
        },
"connectVia": {
            "referenceName": "<name of Self Hosted Integration Runtime>",
            "type": "IntegrationRuntimeReference"
        }
    }
}

Use the port used for REST API. The default is 8093.
•	Next, create the linked service for cosmos DB SQL API 

{
    "name": "<name>",
    "properties": {
        "type": "CosmosDb",
        "typeProperties": {
            "connectionString": {
                "type": "SecureString",
                "value": "AccountEndpoint=<EndpointUrl>;AccountKey=<AccessKey>;Database=<Database>"
            }
        },
        "connectVia": {
            "referenceName": "<name of Azure Integration Runtime>",
            "type": "IntegrationRuntimeReference"
        }
    }
}

 Get the Endpointurl, accesskey and Database name from key section of Cosmos DB resource page.

```
Dataset 
Datasets represent data structures within the data stores, which simply point to or reference the data you want to use in your activities as inputs or outputs.  For data migration, a couchbase and cosmos DB dataset needs to be created. 
•	couchabase , under the table, select the bucket that needs to be migrated to CosmosDB
 
•	Another dataset needs to be created that points to CosmosDB linked service. Select a collection that will be used as the destination 
 
Pipeline 

A pipeline is a logical grouping of activities that performs a unit of work. Activities represent a processing step in a pipeline. The only activity that needs to be done is a copy operation. 
•	Go to pipeline and click on create pipeline.
•	Under move and transform, select a copy data

  
Select Source and Couchbase data set and Sink as CosmosDB dataset.
You can either add a trigger and do run now or select a schedule as per your choice of time window.
Observations 
•	 During data copy,  data copy might fail with “Request rate is large” error. Increase the RU of cosmosDB for bulk copy of the JSON. Once the data copy is done, you can scale down the Cosmos DB RU size
•	The shape of the JSON is flattened when copied using Azure data factory. 
```
  {
"name"
              { "firstname":"xyz",
              "Lastname”: "abc"
}
}
When you create the couchbase table as dataset, the json output becomes 
{
"Name_firstname": "xyz",
"Name_lastname" : "abc"
}

```
•	ID column is transferred as PK in CosmosDB SQL API

### Method 2

CBexport  is an utility for exporting data from a Couchbase cluster. This utility is available in Couchbase enterprise edition.

Cbexport json command exports JSON from a Couchbase cluster. This command either exports JSON documents to a file with a document on each line or exports a file that contain a JSON list where each element is a document.
For Migration, the list option is preferred. The list format contains a JSON list where each element in the list is a JSON document. 

cbexport json -c couchbase://<cluster IP> -u <user name> -p password \ -b <bucket name > -o <filepath\filename>.json -f list
Once the data is exported, you can import the json using two methods 
a.	Copy data using Azure Data Factory 
b.	Import using Data Migration Tool


 
