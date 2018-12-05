# Test Communications<a name="test-comms"></a>

1. Use the following procedure to make sure that your computer and the AWS IoT Greengrass core device are connected to the internet and are using the same network\. 

   1. Run the following command to determine the IP address of the AWS IoT Greengrass core:

      ```
      hostname -I
      ```

   1. From your computer, run the following command using the IP address of the core\. You can use Ctrl \+ C to stop the ping command\.

      ```
      ping IP-address
      ```

      Output similar to the following indicates successful communication between the computer and the AWS IoT Greengrass core device \(0% packet loss\):  
![\[Successful ping command output.\]](http://docs.aws.amazon.com/greengrass/latest/developerguide/images/gg-get-started-075.5.png)

1. In the AWS IoT console, choose **Settings**\.  
![\[The Settings page selected in the AWS IoT console.\]](http://docs.aws.amazon.com/greengrass/latest/developerguide/images/console-settings.png)

1. Under **Settings**, make a note of the value of **Endpoint**\. In the following examples, this endpoint value is indicated by *AWS\_IOT\_ENDPOINT*\.  
![\[AWS IoT endpoint value.\]](http://docs.aws.amazon.com/greengrass/latest/developerguide/images/gg-get-started-075.7.png)

1. Choose **Greengrass**, choose **Groups**, and then choose your group\. Choose **Settings**, choose **Manually manage connection information**, and then choose **View Cores for specific endpoint information**\. Choose your core, and then choose **Connectivity**\. Choose **Edit** and make sure that you have only one endpoint value\. This value must be the IP address of your AWS IoT Greengrass core device \(for port 8883\)\. Choose **Update**\. This ensures that the `basicDiscovery.py` script connects to the correct AWS IoT Greengrass core device IP address\.

1. Open two [command\-line](https://en.wikipedia.org/wiki/Command-line_interface) windows on your computer \(not the AWS IoT Greengrass core device\)\. One command\-line window is for the HelloWorld\_Publisher device and the other is for the HelloWorld\_Subscriber device\. Upon execution, `basicDiscovery.py` attempts to collect information on the location of the AWS IoT Greengrass core at the given endpoint\. This information is stored after the device has discovered and successfully connected to the AWS IoT Greengrass core\. This allows future messaging and operations to be executed locally \(without the need for an internet connection\)\.
**Note**  
You can run the following command from the folder that contains the `basicDiscovery.py` file for detailed script usage information:  

   ```
   python basicDiscovery.py --help
   ```

   From the HelloWorld\_Publisher device window, run the following commands:

   ```
   cd path-to-certs-folder
   python basicDiscovery.py --endpoint AWS_IOT_ENDPOINT --rootCA root-ca-cert.pem --cert publisher.cert.pem --key publisher.private.key --thingName HelloWorld_Publisher --topic 'hello/world/pubsub' --mode publish --message 'Hello, World! Sent from HelloWorld_Publisher'
   ```

   You should see output similar to the following:  
![\[Screenshot of the publisher output.\]](http://docs.aws.amazon.com/greengrass/latest/developerguide/images/gg-get-started-076.png)
**Note**  
If you get a message about unrecognized arguments, for the `message` parameter, change the single quotation marks to double quotation marks\.

1. From the HelloWorld\_Subscriber device window, run the following commands:

   ```
   cd path-to-certs-folder
   python basicDiscovery.py --endpoint AWS_IOT_ENDPOINT --rootCA root-ca-cert.pem --cert subscriber.cert.pem --key subscriber.private.key --thingName HelloWorld_Subscriber --topic 'hello/world/pubsub' --mode subscribe
   ```

   You should see the following output:  
![\[Screenshot of the subscriber output.\]](http://docs.aws.amazon.com/greengrass/latest/developerguide/images/gg-get-started-077.png)

Closing the HelloWorld\_Publisher window stops messages from accruing in the HelloWorld\_Subscriber window\.

For information about the latest usage of `basicDiscovery.py`, see the `README` file of the [AWS IoT Device SDK for Python](https://github.com/aws/aws-iot-device-sdk-python)\.