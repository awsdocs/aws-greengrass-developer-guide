# Test Communications<a name="test-comms"></a>

1. Make sure that your computer and the AWS Greengrass core device are connected to the internet and are using the *the same network*\. The following procedure can be used to confirm this:

   1. Determine the IP address of the AWS Greengrass core by running the following command:

      ```
      hostname -I
      ```

   1. From your computer, run the following command using the core's IP address \(you can use Ctrl \+ C to halt the `ping` command\):

      ```
      ping IP-address
      ```

      Output similar to the following indicates successful communication between the computer and the AWS Greengrass core device \(note 0% packet loss\):  
![\[Successful ping command output.\]](http://docs.aws.amazon.com/greengrass/latest/developerguide/images/gg-get-started-075.5.png)

1. From the AWS IoT console, choose **Settings**, and note the value of **Endpoint**:  
![\[AWS IoT endpoint value.\]](http://docs.aws.amazon.com/greengrass/latest/developerguide/images/gg-get-started-075.7.png)

    In the below, the endpoint value is indicated by *AWS\_IOT\_ENDPOINT*\. 

1. Choose **Greengrass**, **Groups**, and then choose your group\. Choose **Settings**, select **Manually manage connection information**, then choose **View Cores for specific endpoint information**\. Choose your core, then **Connectivity**\. Choose **Edit** and ensure your have only one **Endpoint** value which must be the IP address of your AWS Greengrass core device \(for port 8883\)\. Choose **Update**\. This ensures that the `basicDiscovery.py` script will connect to the correct AWS Greengrass core device IP address\.

1. Open two [command\-line](https://en.wikipedia.org/wiki/Command-line_interface) windows on your computer \(not the AWS Greengrass core device\)\. One command\-line window will be for the **HelloWorld\_Publisher** device and the other for the **HelloWorld\_Subscriber** device\. Every time the following script \(`basicDiscovery.py`\) is executed for the first time, the AWS Greengrass discovery service connects to the AWS Greengrass core\. After a device has discovered the AWS Greengrass core and successfully connected to it, future messaging and operations can be executed locally \(without the need for an internet connection\)\.
**Note**  
You can run the following command from the folder containing the `basicDiscovery.py` file for detailed script usage information:  

   ```
   python basicDiscovery.py --help
   ```

   From the **HelloWorld\_Publisher** device window, run the following commands:

   ```
   cd path-to-certs-folder
   python basicDiscovery.py --endpoint AWS_IOT_ENDPOINT --rootCA root-ca-cert.pem --cert publisher.cert.pem --key publisher.private.key --thingName HelloWorld_Publisher --topic 'hello/world/pubsub' --mode publish --message 'Hello, World! Sent from HelloWorld_Publisher'
   ```

   You should see output similar to the following:  
![\[Screenshot of the publisher output.\]](http://docs.aws.amazon.com/greengrass/latest/developerguide/images/gg-get-started-076.png)
**Note**  
If you get a message about unrecognized arguments, try changing the single quotes to double quotes for the `message` parameter\.

1. From the **HelloWorld\_Subscriber** device window, run the following commands:

   ```
   cd path-to-certs-folder
   python basicDiscovery.py --endpoint AWS_IOT_ENDPOINT --rootCA root-ca-cert.pem --cert subscriber.cert.pem --key subscriber.private.key --thingName HelloWorld_Subscriber --topic 'hello/world/pubsub' --mode subscribe
   ```

   You should see the following output:  
![\[Screenshot of the subscriber output.\]](http://docs.aws.amazon.com/greengrass/latest/developerguide/images/gg-get-started-077.png)

Notice that closing the **HelloWorld\_Publisher** window halts additional messages from accruing in the **HelloWorld\_Subscriber** window\.

For information about the latest usage of `basicDiscovery.py`, see the `README` file of the [AWS IoT Device SDK for Python](https://github.com/aws/aws-iot-device-sdk-python)\.