# Azure percept with Azure Synapse Analytics

## How to integrate Edge AI with Azure Synapse Analytics

## Use Case

- AI can be used in Edge to score or detect objects
- Storing what was detected and their outcomes in cloud
- Helps to retrain and help data scientist to fine tune the model
- Help report on the outcomes of the model

## Architecture

![alt text](https://github.com/balakreshnan/AzurePercept/blob/main/images/Architecturesynapse1.jpg "Architecture")

## Architecture Explained

- Use Azure Percept as AI @ Edge devices
- Device will have module deployed to detect object
- Use can vary based on individual use case
- I am using Azure Stream analytics which can be used in Edge and cloud
- Stream analytics would do the input parsing
- Store the data in either CSV or Parquet
- Store the data in Synapse workspace - default storage
- Make sure single source of truth for data lake
- Code in stream analytics to get model scored data

```
Select width, height, position_x, position_y, label, Confidence, timestamp, 
EventEnqueuedUtcTime, IoTHub.ConnectionDeviceId, IoTHub.EnqueuedTime 
into output 
from input
```

- Model outputs the label and it's bounding box
- Confidence is also sent to take decision

![alt text](https://github.com/balakreshnan/AzurePercept/blob/main/images/percept2.jpg "Architecture")

![alt text](https://github.com/balakreshnan/AzurePercept/blob/main/images/percept3.jpg "Architecture")

- Validate if the files are generated

![alt text](https://github.com/balakreshnan/AzurePercept/blob/main/images/percept4.jpg "Architecture")

- Here is a sample data output as JSON coming from Percept output

```
{
    "width": "516",
    "height": "260",
    "position_x": "282",
    "position_y": "312",
    "label": "laptop",
    "confidence": "0.549805",
    "timestamp": "1615126558907404099",
    "EventProcessedUtcTime": "2021-03-07T14:17:28.5049562Z",
    "PartitionId": 2,
    "EventEnqueuedUtcTime": "2021-03-07T14:15:58.2040000Z",
    "IoTHub": {
      "MessageId": "MSG_ID",
      "CorrelationId": "CORE_ID",
      "ConnectionDeviceId": "aicamera1",
      "ConnectionDeviceGenerationId": "637504148641820944",
      "EnqueuedTime": "2021-03-07T14:15:58.2150000Z",
      "StreamId": null
    }
}
```

- Now log into Azure Synapse Studio
- Create a Server less sql query to access the data
- here is a sample query

```
SELECT
    TOP 100 *
FROM
    OPENROWSET(
        BULK 'https://storageaccount.dfs.core.windows.net/containername/santacruz/incoming/*/*/*/*.parquet',
        FORMAT='PARQUET'
    ) AS [result]
```

- Output from above

![alt text](https://github.com/balakreshnan/AzurePercept/blob/main/images/percept5.jpg "Architecture")

- Now lets load the same data into big data platform like spark
- Spark allows us to further processing
- The data is there only in one place
- We are not moving the data
- Create a notebook
- Here is the code to load data frame

```
%%pyspark
df = spark.read.load('abfss://containername@storageaccount.dfs.core.windows.net/santacruz/incoming/*/*/*/*.parquet', format='parquet')
display(df.limit(10))
```

- here is the output

![alt text](https://github.com/balakreshnan/AzurePercept/blob/main/images/percept6.jpg "Architecture")

- From here we can dervice any type of insights
- Now lets build a simple report
- Connect to power bi
- Go to synapse studio workspace and grab the serverless endpoint to connect

![alt text](https://github.com/balakreshnan/AzurePercept/blob/main/images/percept7.jpg "Architecture")

- Provide the view
- before providing a view create a new one in sql script

```
create VIEW percept1 AS 
SELECT
    *
FROM
    OPENROWSET(
        BULK 'https://storageaccountname.dfs.core.windows.net/containername/santacruz/incoming/*/*/*/*.parquet',
        FORMAT='PARQUET'
    ) AS [result]
```

- Then provide the query

```
Select * from percept1;
```

![alt text](https://github.com/balakreshnan/AzurePercept/blob/main/images/percept8.jpg "Architecture")

- should see the data now

![alt text](https://github.com/balakreshnan/AzurePercept/blob/main/images/percept9.jpg "Architecture")

- Load the data 
- Create a new report and drag few columns to make one

![alt text](https://github.com/balakreshnan/AzurePercept/blob/main/images/percept10.jpg "Architecture")