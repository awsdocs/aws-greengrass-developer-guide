# Calling the local health check API<a name="health-check"></a>

AWS IoT Greengrass contains a local HTTP API that provides a snapshot of the current state of local worker processes that were started by AWS IoT Greengrass\. This snapshot includes user\-defined Lambda functions and system Lambda functions\. System Lambda functions are part of the AWS IoT Greengrass Core software\. They run as local worker processes on the core device and manage operations such as message routing, local shadow sync, and automatic IP address detection\.

The health check API supports the following requests:
+ Send a `GET` request to [get health information for all workers](#health-check-get)\.
+ Send a `POST` request to [get health information for specified workers](#health-check-post)\.

Requests are sent locally on the device and don't require an internet connection\.

## Get health information for all workers<a name="health-check-get"></a>

Send a `GET` request to get health information about all running workers\.
+ Replace *port* with the port number of the IPC\.

```
GET http://localhost:port/2016-11-01/health/workers
```

`port`  
The port number of the IPC\.  
The value can vary between 1024 and 65535\. The default value is 8000\.  
To change this port number, you can update the `ggDaemonPort` property in the `config.json` file\. For more information, see [AWS IoT Greengrass core configuration file](gg-core.md#config-json)\.

**Example request**

The following example `curl` request gets health information for all workers\.

```
curl http://localhost:8000/2016-11-01/health/workers
```

### JSON Response<a name="health-check-get-response"></a>

This request returns an array of [worker health information](#health-information-objects) objects\.

**Example response**

The following example response lists health information objects for all worker processes that were started by AWS IoT Greengrass\.

```
[
    {
      "FuncArn": "arn:aws:lambda:::function:GGShadowService:1",
      "WorkerId" : "65515053-2f70-43dc-7cc0-1712bEXAMPLE",
      "ProcessId": "1234",
      "WorkerState": "Waiting"
    },
    {
      "FuncArn": "arn:aws:lambda:::function:GGSecretManager:1",
      "WorkerId": "a9916cc2-1b4d-4f0e-4b12-b1872EXAMPLE",
      "ProcessId": "9798",
      "WorkerState": "Waiting"
    },
    {
      "FuncArn": "arn:aws:lambda:us-west-2:123456789012:function:my-lambda-function:3",
      "WorkerId": "2e6f785e-66a5-42c9-67df-42073EXAMPLE",
      "ProcessId": "11837",
      "WorkerState": "Waiting"
    },
    ...
]
```

## Get health information about specified workers<a name="health-check-post"></a>

Send a `POST` request to get health information about specified workers\. Replace *port* with the port number of the IPC\. The default is 8000\.

```
POST http://localhost:port/2016-11-01/health/workers
```

**Example request**

The following example `curl` request gets health information for specified workers\.

```
curl --data "@body.json" http://localhost:8000/2016-11-01/health/workers
```

Here's an example `body.json` request body:

```
{
    "FuncArns": [
        "arn:aws:lambda:::function:GGShadowService:1",
        "arn:aws:lambda:us-west-2:123456789012:function:my-lambda-function:3"
    ]
}
```

The request body contains a `FuncArns` array\.

`FuncArns`  
A list of Amazon Resource Names \(ARNs\) for the Lambda functions that represent the target workers\.  
+ For user\-defined Lambda functions, specify the ARN of the currently deployed version\. If you added Lambda functions to the group using an alias ARN, you can use the GET request to get all workers, and then choose the ARNs you want to query for\.
+ For system Lambda functions, specify the ARN of the corresponding Lambda function\. For more information, see [System Lambda functions](#system-lambda-functions)\.
Type: array of strings  
Minimum length: 1  
Maximum length: The total number of workers started by AWS IoT Greengrass on the core device\.

### JSON Response<a name="health-check-post-response"></a>

This request returns a `Workers` array and an `InvalidArns` array\.

`Workers`  
A list of health information objects for the specified workers\.  
Type: array of [health information objects](#health-information-objects)

`InvalidArns`  
A list of function ARNs that are invalid, including function ARNs that don't have associated workers\.  
Type: array of strings

**Example response**

The following example response lists [health information objects](#health-information-objects) for the specified workers\.

```
{ 
    "Workers": [
        {
            "FuncArn": "arn:aws:lambda:::function:GGShadowService:1",
            "WorkerId" : "65515053-2f70-43dc-7cc0-1712bEXAMPLE",
            "ProcessId": "1234",
            "WorkerState": "Waiting"
        },
        {
            "FuncArn": "arn:aws:lambda:us-west-2:123456789012:function:my-lambda-function:3",
            "WorkerId": "2e6f785e-66a5-42c9-67df-42073ESAMPLE",
            "ProcessId": "11837",
            "WorkerState": "Waiting"
        }
    ],
    "InvalidArns" : [
        "some-malformed-arn", 
        "arn:aws:lambda:us-west-2:123456789012:function:some-unknown-function:1"
    ]
}
```

This request returns the following errors:

400 Invalid request  
The request body is malformed\. To resolve this issue, use the following format and resend the request:  

```
{"FuncArns":["function-1-arn","function-2-arn"]}
```

400 Request exceeds max number of workers  
The number of ARNs specified in the `FuncArns` array exceeds the number of workers\.

## Worker health information<a name="health-information-objects"></a>

A health information object contains the following properties:

`FuncArn`  
The ARN of the system Lambda function that represents the worker\.  
Type: `string`

`WorkerId`  
The ID of the worker\. This property can be useful for debugging\. The `runtime.log` file and the Lambda function logs contain the worker ID, so this property can be especially useful to debug an on\-demand Lambda function that spins up multiple instances\.  
Type: `string`

`ProcessId`  
The process ID \(PID\) of the worker process\.  
Type: `int`

`WorkerState`  
The state of the worker\.  
Type: `string`  
The following are possible worker states:    
`Working`  
Processing a message\.  
`Waiting`  
Waiting for a message\. Applies to long\-lived Lambda functions running as a daemon or standalone process\.  
`Starting`  
Spun up, getting started\.  
`FailedInitialization`  
Failed to initialize\.  
`Terminated`  
Stopped by the Greengrass daemon  
`NotStarted`  
Failed to start, making another start attempt\.  
`Initialized`  
Successfully initialized\.

### System Lambda functions<a name="system-lambda-functions"></a>

You can request health information for the following system Lambda functions:

`GGCloudSpooler`  
Manages the queue for MQTT messages that have AWS IoT Core as the source or target\.  
ARN: `arn:aws:lambda:::function:GGCloudSpooler:1`

`GGConnManager`  
Routes MQTT messages between the Greengrass core and connected devices\.  
ARN: `arn:aws:lambda:::function:GGConnManager`

`GGDeviceCertificateManager`  
Listens to the AWS IoT shadow for changes to the core's IP endpoints and generates the server\-side certificate used by GGConnManager for mutual authentication\.  
ARN: `arn:aws:lambda:::function:GGDeviceCertificateManager`

`GGIPDetector`  
Manages automatic IP address detection that enables devices in the Greengrass group to discover the Greengrass core device\. This service isn't applicable when you provide IP addresses manually\.  
ARN: `arn:aws:lambda:::function:GGIPDetector:1`

`GGSecretManager`  
Manages secure storage of local secrets and access by user\-defined Lambda and connectors\.  
ARN: `arn:aws:lambda:::function:GGSecretManager:1`

`GGShadowService`  
Manages local shadows for connected devices\.  
ARN: `arn:aws:lambda:::function:GGShadowService`

`GGShadowSyncManager`  
Synchronizes local shadows with the AWS Cloud for the core device and connected devices, if the device's `syncShadow` property is set to `true`\.  
ARN: `arn:aws:lambda:::function:GGShadowSyncManager`

`GGStreamManager`  
Processes data streams locally and performs automatic exports to the AWS Cloud\.  
ARN: `arn:aws:lambda:::function:GGStreamManager:1`

`GGTES`  
The local token exchange service that retrieves IAM credentials defined in the Greengrass group role that local code uses to access AWS services\.  
ARN: `arn:aws:lambda:::function:GGTES`