# Use Greengrass OPC\-UA to Communicate with Industrial Equipment<a name="opcua"></a>

Greengrass supports OPC\-UA, an information exchange standard for industrial communication\. OPC\-UA allows you to ingest and process messages from industrial equipment and deliver them to devices in your Greengrass group or to the cloud based on rules you define\.

The Greengrass implementation of OPC\-UA supports certificate\-based authentication\. It is based on an open source implementation, and is fully customizable\. You can also bring your own implementation of OPC\-UA, and implement your own support for other custom, legacy, and proprietary messaging protocols\.

In this section we will cover the following steps: 
+ Connect to an existing OPC\-UA server\.
+ Monitor an existing OPC\-UA node within that server\.
+ Get called back when the monitored node's value changes\.

## Architectural Overview<a name="opcua-archi"></a>

Greengrass implements OPC\-UA as a Lambda function in NodeJS\. Since Lambda functions running on Greengrass cores have access to network resources, you can create Lambda functions that proxy information from your existing OPC\-UA servers over TCP to other functions or services in your Greengrass group\.

![\[Greengrass OPCUA Architecture.\]](http://docs.aws.amazon.com/greengrass/latest/developerguide/images/opcua1.png)

You can configure Greengrass to have a long\-lived connection to your OPC\-UA server\(s\), and, using OPC\-UA Subscriptions, you can have your OPCUA\_Adapter Lambda function monitor changes to pre\-defined nodes\. Any change to those nodes triggers a Publish event from the OPC\-UA server, which will be received by your Lambda function, and republished into predefined topic names\.

The topic structure is constructed as follows:

![\[Topic structure construction.\]](http://docs.aws.amazon.com/greengrass/latest/developerguide/images/opcua2.png)

## Set Up a Test OPC\-UA Server<a name="opcua-test-server"></a>

Use the following commands to set up a test OPC\-UA server\. Or, if you already have an OPC\-UA server you'd like to use instead, you may skip this step\.

```
git clone git://github.com/node-opcua/node-opcua.git
cd node-opcua
git checkout v0.0.64
npm install
node bin/simple_server
```

The server produces the following output:

```
[ec2-user@<your_instance_id> node-opcua]$ node bin/simple_server
  server PID          : 28585

registering server to :opc.tcp://<your_instance_id>4840/UADiscovery
err Cannot find module 'usage'
skipping installation of cpu_usage and memory_usage nodes
  server on port      : 26543
  endpointUrl         : opc.tcp://<your_instance_id>us-west-2.compute.internal:26543
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

## Make sure your Greengrass Group is ready<a name="opcua-group"></a>
+ Create a Greengrass group \(find more details in [Configure AWS Greengrass on AWS IoT](gg-config.md)\.\) 
+ Set up a Greengrass Core on one of the supported platforms \(Raspberry\-pi for [example](module1.md#setup-filter.rpi)\) 
+ [Set up](what-is-gg.md#gg-platforms) your Greengrass Core to be able to run nodejs6\.x Lambda functions

## Use Greengrass OPC\-UA to Interact with your OPC\-UA Server<a name="opcua-interact"></a>

1. Prepare your Lambda function

   Get the code for an OPC\-UA adapter Lambda function from GitHub: 

   ```
   git clone https://github.com/aws-samples/aws-greengrass-samples.git
   cd aws-greengrass-samples/greengrass-opcua-adapter-nodejs
   npm install
   ```

   **Note:**This Lambda function uses the node\-opcua library \(v0\.0\.64\), which attempts to re\-generate some model files at runtime\. That doesn't work when running as a Lambda function on Greengrass, because Lambda functions start with a Read\-Only file system, so any code trying to generate other code would not work\. The next step fixes this\.

1. Change the file at `node_modules/node-opcua/lib/misc/factories.js`: line 109 to this: 

   ```
   var generated_source_is_outdated = (!generated_source_exists); 
   ```

   Run this command to make that change: 

   ```
   sed -i '109s/.*/    var generated_source_is_outdated = (!generated_source_exists);/' node_modules/node-opcua/lib/misc/factories.js
   ```

1. Configure the server and monitored nodes

   Change the `configSet` variable inside the `index.js` file of the OPC\-UA Lambda function to contain the server IP and Port that you want to connect to, as well as the node Ids you would like to monitor\. By default it comes with the following example configuration: 

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

   In this case, we are connecting to an OPC\-UA server running on the same host as our Greengrass Core, on port 26543, and monitoring one node that has an OPC\-UA Id `'ns=1;s=PumpSpeed'`\. 

1. Configure the authentication mode

   The [OPC\-UA library](https://github.com/node-opcua/node-opcua) used in this example supports three modes of Authentication to your OPC\-UA server\. The most secure method is Certificate Based Authentication, but the library also allows you to specify username/password or no authentication\.

   Here is how to set Certificate Based Authentication:
   + Package your certificate and private key with your Lambda function, for example under a directory named `certs/`\.
   + Change the `clientOptions` variable to contain certificateFile, privateKeyFile and securityModes, securityPolicies options: 

     ```
     const clientOptions = { 
         keepSessionAlive: true,
         certificateFile: /lambda/certs/<certificate_name>.pem.crt,
         privateKeyFile: /lambda/certs/<private_key_name>.pem.key,
         securityModes: MessageSecurityMode.SIGN,
         securityPolicies: SecurityPolicy.BASIC256,
         connectionStrategy: {
             maxRetry: 1000000,
             initialDelay: 2000,
             maxDelay: 10 * 1000,
         },  
     };
     ```

1. Upload your Lambda

   Create a Greengrass Lambda function\. You can find more details on how to do that in [Configure the Lambda Function for AWS Greengrass](config-lambda.md)\. In a nutshell, create a Lambda function code archive by doing the following:

   ```
   # Download the nodejs greengrass sdk from 
   #   https://console.aws.amazon.com/iotv2/home?region=us-east-1#/software/greengrass/sdk.
   
   #  Install Greengrass SDK in the node_modules directory
   tar -zxvf aws-greengrass-core-sdk-js-*.tar.gz -C /tmp/
   unzip /tmp/aws_greengrass_core_sdk_js/sdk/aws-greengrass-core-sdk.zip -d node_modules
   
   # Archive the whole directory as a zip file
   zip -r opcuaLambda.zip * -x \*.git\*
   
   # Create an AWS Lambda with the created zip
   aws lambda create-function --function-name <Function_Name> --runtime 'nodejs6.10' --role <Your_Role> --handler 'index.handler' --zip-file opcuaLambda.zip
   ```

   Add this Lambda to your Greengrass Group\. Details are, again, in: [Configure the Lambda Function for AWS Greengrass](config-lambda.md)\. 

1. Configure and Deploy the Lambda function to your Greengrass Group

   After creating your AWS Lambda function, you add it to your Greengrass Group\. Follow the instructions in same section as above\. 
   + Make sure to specify the Lambda function as Long\-Running\.
   + Give it at least 64MB of memory size\.

   You can now create a deployment with your latest configuration\. You can find details in [Deploy Cloud Configurations to an AWS Greengrass Core Device](configs-core.md)\.

## Verify that your Lambda function is receiving OPC\-UA Publishes and posting them onto Greengrass<a name="opcua-verify-lambda"></a>

As described in the [Architecture section](#opcua-archi), your Lambda function should start receiving messages from your OPC\-UA server\. If you are using your own custom OPC\-UA server, make sure you trigger a change in the OPC\-UA node Id you specified, so that you see the change received by your Lambda function\. If you are using the example server above, the PumpSpeed node is configured to simulate a series of consecutive updates, so you should expect your Lambda function to receive multiple messages a second\.

You can see messages received by your Lambda function in one of two ways: 
+ Watch the Lambda function’s logs 

   You can view the logs from your Lambda function by running the following command: 

  ```
   sudo cat ggc/var/log/user/us-west-2/your_account_id/your_function_name.log 
  ```

  The logs should look similar to: 

  ```
  [2017-11-14T16:33:09.05Z][INFO]-started subscription : 305964
  
  [2017-11-14T16:33:09.05Z][INFO]-monitoring node id =  ns=1;s=PumpSpeed
  
  [2017-11-14T16:33:09.099Z][INFO]-monitoredItem initialized
  
  [2017-11-15T23:49:34.752Z][INFO]-Publishing message on topic "/opcua/server/node/MyPumpSpeed" with Payload "{"id":"ns=1;s=PumpSpeed","value":{"dataType":"Double","arrayType":"Scalar","value":237.5250759433095}}"
  ```
+ Configure Greengrass to forward messages from your Lambda function to the IoT Cloud\.

  Follow the steps outlined in [Verify the Lambda Function Is Running on the Device](lambda-check.md) to receive messages on the AWS IoT console\.

**Note:**
+ Make sure there is a Subscription from your Lambda function going to the IoT Cloud\. Details are in [Configure the Lambda Function for AWS Greengrass](config-lambda.md)\.
+ Since messages are forwarded to the cloud, make sure you terminate either the example server you configured above, or stop the Greengrass core, so that you don't end up publishing a lot of messages to IoT cloud and getting charged for them\!

## Next Steps<a name="opcua-next"></a>

With Greengrass, you can use this same architecture to create your own implementation of OPC\-UA, and also implement your own support for custom, legacy, and proprietary messaging protocols\. Since Lambda functions running on Greengrass cores have access to network resources, you can use them to implement support for any protocol that rides on top of TCP\-IP\. In addition, you can also take advantage of Greengrass Local Resource Access to implement support for protocols that need access to hardware adapters/drivers\.