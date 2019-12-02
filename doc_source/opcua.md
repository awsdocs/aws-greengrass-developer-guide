# Implement Custom OPC\-UA Support to Communicate with Industrial Equipment<a name="opcua"></a>

OPC\-UA is an information exchange standard for industrial communication\. You can create custom implementations for AWS IoT Greengrass that allow you to use OPC\-UA to ingest and process messages from industrial equipment\. Based on rules you define, messages can be delivered to local or cloud targets\.

This tutorial shows an example custom OPC\-UA implementation on your AWS IoT Greengrass core\. The example is based on an open source implementation that supports certificate\-based authentication and is fully customizable\. You can follow this pattern to add support for OPC\-UA and other custom, legacy, or proprietary messaging protocols\.

**Note**  
This example is a lightweight implementation that handles limited amounts of data\. We recommend that you use AWS IoT SiteWise \(currently in limited preview\) to collect and organize industrial data at scale\. For more information, see [What Is AWS IoT SiteWise?](https://docs.aws.amazon.com/iot-sitewise/latest/userguide/) in the *AWS IoT SiteWise User Guide*\.

The example implementation uses a Node\.js Lambda function that acts as an OPC\-UA proxy on a Greengrass core\. This is possible because Lambda functions running on a core can access network resources\. In the following diagram, the `OPCUA_Adapter` Lambda function transfers information sent from an OPC\-UA server over TCP to other functions or services in the Greengrass group\.

![\[Example OPC-UA architecture.\]](http://docs.aws.amazon.com/greengrass/latest/developerguide/images/opcua1.png)

In this tutorial, you create a Lambda function that connects to an OPC\-UA server, monitors an OPC\-UA node in the server, and gets a message when the value of a monitored node changes\. You configure a long\-lived connection between the Lambda function and your OPC\-UA server and use OPC\-UA subscriptions that allow the function to monitor changes to predefined nodes\. Changes to these nodes trigger a `Publish` event from the OPC\-UA server\. When the Lambda function receives messages from the server, it republishes the messages to predefined topics\.

The topic structure uses the following syntax:

![\[Topic structure construction.\]](http://docs.aws.amazon.com/greengrass/latest/developerguide/images/opcua2.png)

The tutorial contains the following high\-level steps:
+ [Set Up a Test OPC\-UA Server](#opcua-test-server)
+ [Create an OPC\-UA Adapter Lambda Function That Interacts with the OPC\-UA Server](#opcua-interact)
+ [Test the OPC\-UA Adapter Lambda Function](#opcua-verify-lambda)

## Prerequisites<a name="opcua-prerequisites"></a>
+ A Greengrass group and a Greengrass core\. To learn how to create a Greengrass group or core, see [Getting Started with AWS IoT Greengrass](gg-gs.md)\.
+  Node\.js 8\.10 installed on your core device\. 

## Set Up a Test OPC\-UA Server<a name="opcua-test-server"></a>
+ Use the following commands to set up a test OPC\-UA server\. Skip this step if you have an OPC\-UA server you want to use\.
**Note**  
The `npm` package manager must be installed to run this command\. For information, see [Get npm](https://www.npmjs.com/get-npm) on the npm website\.

  ```
  git clone git://github.com/node-opcua/node-opcua.git
  cd node-opcua
  git checkout v0.0.65
  npm install
  node bin/simple_server
  ```

  The server produces the following output:

  ```
  [ec2-user@your_instance_id node-opcua]$ node bin/simple_server
    server PID          : 28585
  
  registering server to :opc.tcp://your_instance_id4840/UADiscovery
  err Cannot find module 'usage'
  skipping installation of cpu_usage and memory_usage nodes
    server on port      : 26543
    endpointUrl         : opc.tcp://your_instance_idregion.compute.internal:26543
    serverInfo          :
        applicationUri                  : urn:54f7890cca4c49a1:NodeOPCUA-Server
        productUri                      : NodeOPCUA-Server
        applicationName                 : locale=en text=NodeOPCUA
        applicationType                 : SERVER
        gatewayServerUri                : null
        discoveryProfileUri             : null
        discoveryUrls                   : 
        productName                     : NODEOPCUA-SERVER
    buildInfo           :
        productUri                      : NodeOPCUA-Server
        manufacturerName                : Node-OPCUA : MIT Licence ( see http://node-opcua.github.io/)
        productName                     : NODEOPCUA-SERVER
        softwareVersion                 : 0.0.65
        buildNumber                     : 1234
        buildDate                       : Thu Aug 03 2017 00:13:50 GMT+0000 (UTC)
  
    server now waiting for connections. CTRL+C to stop
  ```

## Create an OPC\-UA Adapter Lambda Function That Interacts with the OPC\-UA Server<a name="opcua-interact"></a>

1. Prepare your Lambda function\.

   Get the code for an OPC\-UA adapter Lambda function from GitHub: 

   ```
   git clone https://github.com/aws-samples/aws-greengrass-samples.git
   cd aws-greengrass-samples/greengrass-opcua-adapter-nodejs
   npm install
   ```
**Note**  
The `node-opcua` library \(v0\.0\.65\) used by the Lambda function attempts to regenerate some model files at runtime\. Because Lambda functions start with a containerized, read\-only file system on a Greengrass core, the attempt to generate other code fails\. The next step fixes this issue\.

1. Change line 109 of the file at `node_modules/node-opcua/lib/misc/factories.js` to this: 

   ```
   var generated_source_is_outdated = (!generated_source_exists); 
   ```

   Run this command to make that change: 

   ```
   sed -i '109s/.*/    var generated_source_is_outdated = (!generated_source_exists);/' node_modules/node-opcua/lib/misc/factories.js
   ```

1. Configure the server and monitored nodes\.

   Change the `configSet` variable inside the `index.js` file of the OPC\-UA adapter Lambda function to use the target server IP address and port, and the node IDs you want to monitor\. By default, it uses the following configuration: 

   ```
   const configSet = {   
       server: {
           name: 'server',
           url: 'opc.tcp://localhost:26543',
       },  
       subscriptions: [
           {   
           name: 'MyPumpSpeed',
           nodeId: 'ns=1;s=PumpSpeed',
           },  
       ],  
   };
   ```

   In this case, you connect to an OPC\-UA server running on the same host as your Greengrass core \(on port 26543\) and monitor the node with the OPC\-UA ID `'ns=1;s=PumpSpeed'`\. 

1. Configure the authentication mode\.

   The [OPC\-UA library](https://github.com/node-opcua/node-opcua) used in this example supports three modes of authentication to your OPC\-UA server\. The most secure method is certificate\-based authentication, but the library also supports user name and password or no authentication\.

   To set certificate\-based authentication:
   + Package your certificate and private key with your Lambda function \(for example, under a directory named `certs/`\)\.
   + Change the `clientOptions` variable to contain `certificateFile`, `privateKeyFile`, `securityModes`, and `securityPolicies` options: 

     ```
     const clientOptions = { 
         keepSessionAlive: true,
         certificateFile: /lambda/certs/certificate_name.pem.crt,
         privateKeyFile: /lambda/certs/private_key_name.pem.key,
         securityModes: MessageSecurityMode.SIGN,
         securityPolicies: SecurityPolicy.BASIC256,
         connectionStrategy: {
             maxRetry: 1000000,
             initialDelay: 2000,
             maxDelay: 10 * 1000,
         },  
     };
     ```

1. Download the AWS IoT Greengrass Core SDK for Node\.js from the [AWS IoT Greengrass Core SDK](what-is-gg.md#gg-core-sdk-download) downloads page\.

1. Create and upload your Lambda function\.

   Run the following commands to create the Lambda function and upload the code and dependencies\. For more information, see [Configure the Lambda Function for AWS IoT Greengrass](config-lambda.md)\.

   ```
   #  Install Greengrass SDK in the node_modules directory. If you used the curl command to download the SDK, replace the tar.gz file name 
   #  with the name you used.
   tar -zxvf aws-greengrass-core-sdk-js-*.tar.gz -C /tmp/
   unzip /tmp/aws_greengrass_core_sdk_js/sdk/aws-greengrass-core-sdk-js.zip -d node_modules
   
   # Archive the whole directory as a zip file
   zip -r opcuaLambda.zip * -x \*.git\*
   
   # Create an AWS Lambda function with the zip file
   aws lambda create-function --function-name function-name --runtime 'nodejs8.10' --role your_role --handler 'index.handler' --zip-file opcuaLambda.zip
   ```

1. Configure and deploy the Lambda function\.

   Add the Lambda function to your Greengrass group and configure its group\-level settings\. For instructions, see [Configure the Lambda Function for AWS IoT Greengrass](config-lambda.md)\.
   + For **Lambda lifecycle**, choose **Make this function long\-lived and keep it running indefinitely**\.
   + For **Memory**, choose at least 64 MB\.

   Deploy the group\. For instructions, see [Deploy Cloud Configurations to an AWS IoT Greengrass Core Device](configs-core.md)\.

## Test the OPC\-UA Adapter Lambda Function<a name="opcua-verify-lambda"></a>

The Lambda function should start receiving messages from your OPC\-UA server after you deploy the group\.
+ If you're using your own OPC\-UA server, trigger a change in the OPC\-UA node ID that you specified and check if your Lambda function receives a message\.
+ If you're using the example server, the PumpSpeed node is configured to simulate a series of consecutive updates, so your Lambda function should receive multiple messages per second\.

You can see the messages received by your Lambda function in one of two ways: 
+ Check the Lambda function's logs:

  ```
  sudo cat /greengrass-root/ggc/var/log/user/region/account-id/your_function_name.log 
  ```

  The logs should look similar to this example: 

  ```
  [2017-11-14T16:33:09.05Z][INFO]-started subscription : 305964
  
  [2017-11-14T16:33:09.05Z][INFO]-monitoring node id =  ns=1;s=PumpSpeed
  
  [2017-11-14T16:33:09.099Z][INFO]-monitoredItem initialized
  
  [2017-11-15T23:49:34.752Z][INFO]-Publishing message on topic "/opcua/server/node/MyPumpSpeed" with Payload "{"id":"ns=1;s=PumpSpeed","value":{"dataType":"Double","arrayType":"Scalar","value":237.5250759433095}}"
  ```
+ In the AWS IoT console, create a subscription that allows your Lambda function to publish messages to AWS IoT:

  Set your Lambda function as the source and **IoT Cloud** as the target\. For instructions, see [this step](config-lambda.md#group-subscriptions)\.

  Follow the steps in [Verify the Lambda Function Is Running on the Device](lambda-check.md) to view the messages in the AWS IoT console\.

**Note**  
To avoid incurring charges for message sent to AWS IoT after you finish testing, stop the Greengrass daemon, and then terminate the example server\.  
To stop the AWS IoT Greengrass daemon:  

```
cd /greengrass-root/ggc/core/
sudo ./greengrassd stop
```

## Next Steps<a name="opcua-next"></a>

Use this pattern to create your own implementation to support OPC\-UA and other custom, legacy, or proprietary messaging protocols\. Your implementation can use local Lambda functions to access network resources and add support for any protocol on top of TCP/IP\. You can also use Greengrass local resources to add support for protocols that need access to hardware adapters and drivers\. Or, to learn about AWS IoT SiteWise \(currently in limited preview\), an industry\-level solution designed to ingest data at scale, see [What Is AWS IoT SiteWise?](https://docs.aws.amazon.com/iot-sitewise/latest/userguide/) in the *AWS IoT SiteWise User Guide*\.