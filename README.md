# MongoMigrationDataAssessment

Data Assessment Console Application is a CLI tool that helps evaluate the suitability and readiness of your data and queries for migrating from MongoDB to Azure Cosmos DB for MongoDB. You can use this tool to examine your data and find out the steps that you may need to take to smoothly run your workloads on Azure Cosmos DB. To examine your data, this tool performs a set of assessments to detect incompatibilities, which depend on the size/format of the data stored in the collection. This enables users to check the data in their collections for incompatibilities that might arise during data migration. The tool also has an optional feature to collect a complete list of features and queries used by your application and shows it to you through the data assessment report.

The user can view the output Json file of data assessment or can upload the output file to our Azure Cosmos DB Migration for MongoDB Azure Data Studio (ADS) extension during assessment. On ADS, the data related incompatibilities found during data assessment will be shown to the customer along with incompatibilities identified through the schema assessment run through ADS extension. The features and queries collected as part of data assessment will also be evaluated and corresponding incompatibilities will be shown. 

With this user-friendly data assessment tool, you can now be fully assured and ready for data migration and a successful modernization of your workloads to Azure Cosmos DB for MongoDB!

## Supported source and target versions 

This tool enables data assessment for MongoDB versions 4.4 and higher. The assessment can be directed at both the RU (Request Units) and vCore offerings of Azure Cosmos DB for MongoDB.  

## How to run data assessment

### Pre-requisites

- .net runtime 6.0 
- The client machine on which you plan to run data assessment should have access to the source MongoDB endpoint either over private or public network.
- The user connected to MongoDB should have access to `readAnyDatabase` and `clusterMonitor` roles assigned on the source instance. 

### Steps to run data assessment

1. Download the latest zip from the **Releases** section located in the right pane of this repo.
1. Unzip  the downloaded zip.
1. Navigate to the unzipped directory from Comand Line (cmd). 
1. Execute the command below to run data assessment based on details available in source endpoint by default. This method does not need any preparation and can be run anytime.
    ```DOS
    MongoDataAssessment.exe -c "source-mongodb-connection-string" -t RU
    ```
    Alternately, you may specify the path to the MongoDB logs for your system. These logs will be analyzed as a part of the data assessment, to provide a comprehensive list of features and queries used by your application. 
    ``` DOS
    MongoDataAssessment.exe -c "source-mongodb-connection-string" -t RU -l "logs-directory-path"
    ```
    The output Json report is saved by default at the path below but can be modified using the -o parameter while running the console application. 
    *C:\Users\<usernamne>\AppData\Local\Temp\MongoDataAssessment\Reports*

4. To view the data and query incompatibilities upload the data assessment report to Azure Cosmos DB Migration for MongoDB ADS extension during schema assessment. 

**Note:** The data assessment tool does not modify any of the existing configurations or resources in the source environment, it only reads the available data and logs. The entire list of commands the tool runs are: ```countDocuments, find, listDatabases, listCollections, listIndexes, getMore and aggregate (using $bsonSize, $gt, $project and $match)```. 

- Refer to [Assessment command parameters](#assessment-command-parameters) section below to learn more about the parameters supported while running data assessment exe. 

- Refer to [Feature Analysis](#feature-analysis) section below to learn how to modify log configuration to make logs useful for feature and query analysis in data assessment. 

 ### Assessment command parameters 

**-c | --connectionString** *(Required)*

Source MongoDB endpoint connection string 
  - The hostname or IP should be one either privately or publicly accessible from the client machine. 
  - The credentials specified should have minimum readAnyDatabase and clusterMonitor [roles](https://www.mongodb.com/docs/manual/reference/built-in-roles/#mongodb-authrole-clusterMonitor) assigned.
  - In case of a sharded cluster, use the connection string of the mongos instance.
  - Special characters in username and password will need to be percent-encoded before adding them to the connection string. You may use a publicly available [URL encoder](https://www.urlencoder.org/) to do the encoding. 

**-t | --targetVersion** *(Required)*

Target version is the version of Azure Cosmos DB for MongoDB that you want to migrate to. The possible values are RU and vCore corresponding to RU and vCore offerings of Azure Cosmos DB for MongoDB. 

**-l | --logFilePath** *(Optional)*

Path to the directory containing MongoDB logs. These logs will be used for feature and query analysis. 

**-o | --outputFolderPath** *(Optional)*

Path to the directory where the output of data assessment will be stored. The default output folder is *C:\Users\<usernamne>\AppData\Local\Temp\MongoDataAssessment\Reports*

**-ca | --collectionsToAssessFilePath** *(Optional)*

To assess only specific collections from the cluster. Users can provide those collection namespaces in a file (one namespace per line). This argument is the path to the file containing namespaces of collections to assess in data assessment. 

**-ce | --collectionsToExcludeFilePath** *(Optional)*

To exclude specific collections from the cluster for data assessment. Users can provide those collection namespaces (databasename.collectionname) in a file (one namespace per line). This argument is the path to the file containing namespaces of collections to exclude in data assessment. 

**-g | --getCompleteDetails** *(Optional)*

The tool by default returns a maximum of 20 documents per collection for each check done as part of data assessment. If users want to get all the documents rather than 20, they can pass the value of this argument as *true*.  

### Understanding the data assessment report 

The data assessment report is a Json file with the following keys: 

**host**: The host details of the source MongoDB endpoint provided by you while executing the data assessment exe. 

```
"host": "test-cluster.mongodb.net", 
```
 

**targetVersion**: The version of Azure Cosmos DB for MongoDB chosen by you while executing the data assessment exe. 
```
"targetVersion": "RU", 
```
 

**features**: This field contains an array having feature usage details. The exact details present are database name, collection name, feature name and feature usage count.  

In the report, if both the database name and collection name are blank for a feature, it means the feature was run at instance level, example: ```setParameter```. When only collection name is blank, it means that the feature was run at database level, example: ```listCollections```. 
```
"features": [ 
        { 
            "featureUsageDetails": [ 
                { 
                    "databaseName": "", 
                    "collectionName": "", 
                    "featureName": "setParameter", 
                    "featureUsageCount": 1 
                }, 
                { 
                    "databaseName": "testDB", 
                    "collectionName": "", 
                    "featureName": "listCollections", 
                    "featureUsageCount": 3 
                }, 
                { 
                    "databaseName": "testDB", 
                    "collectionName": "testCollection", 
                    "featureName": "insert", 
                    "featureUsageCount": 4 
                } 
            ] 
        } 
    ]
 ```
**Issues**: This field contains the data related incompatibilities identified during data assessment. The following are the incompatibilities which will be present in issues: 

  - idTypeNonCompatibleShardKey 
  
  - duplicateId 
  
  - unsupportedFieldsForTTLIndex 
  
  - unsupportedFieldsForUniqueIndex 
  
  - unsupportedDocumentSize 

Each of the above incompatibility types may contain an array of identified incompatibilities. The incompatibilities are  

```
"issues": { 
        "idTypeNonCompatibleShardKey": [ 
            { 
                "databaseName": "testDatabase", 
                "collectionName": "testCollection", 
                "isComplete": true, 
                "nonCompatibleShardKeyTypes": [ 
                    { 
                        "type": "ObjectId", 
                        "ids": [ 
                            "65cca107e205c1d8665ddd6e", 
                            "65cca107e205c1d8665ddd6f" 
                        ] 
                    } 
                ] 
            } 
        ], 
        "duplicateId": [ 
            { 
                "databaseName": "testDatabase1", 
                "collectionName": "testCollection1", 
                "isComplete": true, 
                "duplicateIds": [ 
                    { 
                        "id": "65cda286493ca0dbca026dfd", 
                        "type": "ObjectId" 
                    } 
                ] 
            } 
        ], 
        "unsupportedDocumentSize": [ 
            { 
                "databaseName": "testDatabase2", 
                "collectionName": "testCollection2", 
                "isComplete": true, 
                "documentList": [ 
                    { 
                        "documentId": "65cda286493ca0dbca026dfd", 
                        "documentSize": "2100284" 
                    } 
                ] 
            } 

        ], 
        "unsupportedFieldsForUniqueIndex": [ 
            { 
                "databaseName": "testDatabase3", 
                "collectionName": "testCollection3", 
                "isComplete": true, 
                "documentsList": { 
                    "65df6ef58f8cce6a2cd8764b": { 
                        "documentId": "65df6ef58f8cce6a2cd8764b", 
                        "indexList": { 
                            "a_1": { 
                                "indexName": "testIndexName", 
                                "fields": [ 
                                    { 
                                        "fieldName": "testIndexFieldName", 
                                        "fieldType": "Array" 
                                    } 
                                ] 
                            } 
                        } 
                    } 
                } 
            } 
        ], 
        "unsupportedFieldsForTTLIndex": [ 
            { 
                "databaseName": "testDatabase4",
                "collectionName": "testCollection4", 
                "isComplete": true, 
                "documentsList": { 
                    "65cdaa87493ca0dbca026e03": { 
                        "documentId": "65cdaa87493ca0dbca026e03", 
                        "indexList": { 
                            "a_1": { 
                                "indexName": "testIndexName1", 
                                "fields": [ 
                                    { 
                                        "fieldName": "testIndexFieldName1", 
                                        "fieldType": "Array" 
                                    } 
                                ] 
                            } 
                        } 
                    } 
                } 
            } 
        ] 
    } 
```
### Feature Analysis 

#### What do we mean by Feature Analysis? 

MongoDB has a comprehensive list of query language constructs which include commands, stages, operators and operations. In this section, we are going to use the term “Features” to refer to these query language constructs.  

Azure Cosmos DB for MongoDB provides supports to a lot of features available in native MongoDB and is constantly adding support for more features. Some of the features which are currently not available on Azure Cosmos DB for MongoDB might be important for customer’s application. The aim of feature analysis is to identify these features and queries and help customer in making an informed decision while migrating from MongoDB to Azure Cosmos DB for MongoDB. 
 
#### How to do Feature Analysis in Data Assessment? 

During feature analysis, we have ensured that we don’t add any significant overhead which impacts the performance of your cluster. Also, it works for both mongod and mongos instances. 

We parse the log files to get the features used in customer’s application. The default settings of logging don’t capture all the features. So, you need to alter log settings and MongoDB provides an option to alter the logging settings extensively to get more information from the logs. 

There are multiple [components](https://www.mongodb.com/docs/manual/reference/log-messages/#components) of MongoDB logs which are discussed in their official documentation. For the current use case to fetch features, you need to alter settings for following components: 

  - [command](https://www.mongodb.com/docs/manual/reference/log-messages/#mongodb-data-COMMAND): Messages related to database commands, such as [create](https://www.mongodb.com/docs/v5.3/reference/command/create/#mongodb-dbcommand-dbcmd.create). 
  - [index](https://www.mongodb.com/docs/manual/reference/log-messages/#mongodb-data-INDEX): Messages related to indexing operations, such as creating indexes. 
  - [query](https://www.mongodb.com/docs/manual/reference/log-messages/#mongodb-data-QUERY): Messages related to queries, because queries can have [aggregations](https://www.mongodb.com/docs/manual/aggregation/). 
  - [write](https://www.mongodb.com/docs/manual/reference/log-messages/#mongodb-data-WRITE): Messages related to write operations, because write/update can also have [aggregations](https://www.mongodb.com/docs/manual/aggregation/). 

MongoDB provides a construct in log settings called [verbosity levels](https://www.mongodb.com/docs/manual/reference/log-messages/#verbosity-levels). By changing the log verbosity settings, a user can instruct MongoDB to increase or decrease the amount of log messages. This setting can be set either for all the components or for some specific components too. [Verbosity levels](https://www.mongodb.com/docs/manual/reference/parameters/#mongodb-parameter-param.logComponentVerbosity) varies from 0 to 5 with 0 being the least verbose and 5 being most verbose. Changing the log verbosity for the above-mentioned components will serve the purpose of getting logs for features and queries used by your application.   

Before altering the log settings, you need to check the existing log settings. You can use the [db.getLogComponents()](https://www.mongodb.com/docs/manual/reference/method/db.getLogComponents/#mongodb-method-db.getLogComponents) mongosh (mongo shell) method to get the current log verbosity settings. 

In order to alter the verbosity settings, you need to run an [admin](https://www.mongodb.com/docs/manual/reference/command/nav-administration/) command from mongosh (mongo shell) with following with [setParameter](https://www.mongodb.com/docs/manual/reference/command/setParameter/#mongodb-dbcommand-dbcmd.setParameter) and [logComponentVerbosity](https://www.mongodb.com/docs/manual/reference/parameters/#mongodb-parameter-param.logComponentVerbosity) parameters: 
```
setParameter: 1 
logComponentVerbosity: {verbosity settings document} 
```

The exact details on how to configure log verbosity can be read from official documentation. The command which you need to execute is as follows: 
```
db.adminCommand(  
    { 
        setParameter: 1, 
        logComponentVerbosity: 
        {  
            verbosity: 0, 
            command: { verbosity: 1 }, 
            index: { verbosity: 1 },  
            query: { verbosity: 1 }, 
            write: {verbosity: 1} 
        } 
    }) 
```
Running the above commands increases the verbosity for the command, index, query and write components to 1, along with keeping verbosity for other components to a minimum i.e. 0. Now, the logs will have more informational and debug statements related to these components and those statements will have details of features and queries used by your application. 

## FAQs for Feature Analysis? 

### How to get current log verbosity settings? 

Run ```db.getLogComponents()``` in mongosh (mongo shell) 

### How do you configure log verbosity settings? 

Refer to log verbosity settings mentioned above. For further details refer to official MongoDB [documentation](https://www.mongodb.com/docs/manual/reference/log-messages/#configure-log-verbosity-levels). 

### What happens if an instance is restarted? 

In case an instance is restarted, MongoDB reverts back to default log settings. Hence, you need to rerun the command to change log verbosity settings. 

### In a sharded cluster, does updating log verbosity for one node get applied to all nodes? 

In case of sharded cluster, every node (mongos and all mongod nodes) has their own logs. To change log verbosity, connect to a specific node from mongo shell and then update the verbosity settings. The new log verbosity settings will be applied only to node which you are connected through mongo shell and not to all the nodes. If verbosity settings are changed for a mongos node, it will be applied to mongos node only. Similarly, altering verbosity settings for a mongod node will apply it to that mongod node only.  

### In a sharded cluster, which instance shall we alter log settings (mongod or mongos)? 
While using a sharded cluster, changing verbosity settings for a node will alter settings for that node only and not all the nodes. For a sharded cluster, it is recommended to alter settings for the router mongos node.  

### How long does a customer needs to enable these settings to capture features data? 
There is no fixed time period for which these settings should be altered as it is dependent on your application. The recommendation is to enable it long enough so that most of the features are used in application and hence captured in logs. If most of the features/queries used in your application can be captured in 1 day or 2 days, then that’s how long we need the logs to be captured. If you are unsure about the duration, we advocate running tests or possible scenarios after altering verbosity settings to quickly capture features. 

### Is there any performance overhead? 
There is no mention of any performance impact due to logging overhead in official MongoDB documentation. We also consider it to be negligible given the only change is in verbosity of logs which adds more log statements. During the tests run at our end, we didn’t encounter any overhead too. 

### Since the logs size will be more than earlier due to additional log statements, what is the expected disk space taken up by logs? 
The log size will be more than earlier because we have changed the verbosity settings. There is no specific metric to predict the size of log file. It will be dependent on the verbosity settings, period for which those settings were enabled, your workload and some internal MongoDB settings. A general observation in the tests carried out by us is that the logs size will be constant multiple of earlier size and not anything exponential. For a production environment, the increase in log size can have storage implications, so you are advised to keep a check on log file size and reset verbosity settings to default if required. 

### How to reset verbosity settings to default? 
If you ran the above-mentioned command to alter verbosity settings, then you need to run following command to revert it to default. 

  ```
  db.adminCommand(  
      { 
          setParameter: 1, 
          logComponentVerbosity: 
          {  
              verbosity: 0, 
              command: { verbosity: -1 }, 
              index: { verbosity: -1 },  
              query: { verbosity: -1 }, 
              write: {verbosity: -1} 
          } 
      })
  ```

If you ran some other command or changed verbosity for other components too, then you need to be cautious to revert verbosity settings to default for all the components which were modified earlier. We recommend customers to follow the steps mentioned below:

1. Run ```db.getLogComponents()``` to get the current verbosity settings. 
 
2. Run following command to set root level verbosity to zero.
 
      ```
    		db.adminCommand({ 
    			setParameter: 1, 
    			logComponentVerbosity: {verbosity: 0} 
    		})
      ```
3. Now, get current verbosity settings again and reset the verbosity settings for components having non-zero verbosity individually using following command. 
    ```
    db.setLogLevel(-1, <component_name>)
    ```
