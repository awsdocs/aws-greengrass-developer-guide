--------

AWS IoT Greengrass Version 1 no longer receives feature updates, and will receive only security patches and bug fixes until June 30, 2023\. For more information, see the [AWS IoT Greengrass V1 maintenance policy](https://docs.aws.amazon.com/greengrass/v1/developerguide/maintenance-policy.html)\. We strongly recommend that you [migrate to AWS IoT Greengrass Version 2](https://docs.aws.amazon.com/greengrass/v2/developerguide/move-from-v1.html), which adds [significant new features](https://docs.aws.amazon.com/greengrass/v2/developerguide/greengrass-v2-whats-new.html) and [support for additional platforms](https://docs.aws.amazon.com/greengrass/v2/developerguide/operating-system-feature-support-matrix.html)\.

--------

# Test communications<a name="test-comms"></a>

1. <a name="ping-device"></a>Make sure that your computer and the AWS IoT Greengrass core device are connected to the internet using the same network\.

   1. On the AWS IoT Greengrass core device, run the following command to find its IP address\.

      ```
      hostname -I
      ```

   1. On your computer, run the following command using the IP address of the core\. You can use Ctrl \+ C to stop the ping command\.

      ```
      ping IP-address
      ```

      Output similar to the following indicates successful communication between the computer and the AWS IoT Greengrass core device \(0% packet loss\):  
![\[Successful ping command output.\]](http://docs.aws.amazon.com/greengrass/v1/developerguide/images/gg-get-started-075.5.png)
**Note**  
If you're unable to ping an EC2 instance that's running AWS IoT Greengrass, make sure that the inbound security group rules for the instance allow ICMP traffic for [Echo request](https://docs.aws.amazon.com/AWSEC2/latest/UserGuide/security-group-rules-reference.html#sg-rules-ping) messages\. For more information, see [ Adding rules to a security group](https://docs.aws.amazon.com/AWSEC2/latest/UserGuide/using-network-security.html#adding-security-group-rule) in the *Amazon EC2 User Guide for Linux Instances*\.  
On Windows host computers, in the Windows Firewall with Advanced Security app, you might also need to enable an inbound rule that allows inbound echo requests \(for example, **File and Printer Sharing \(Echo Request \- ICMPv4\-In\)**\), or create one\.

1. Get your AWS IoT endpoint\.

   1. <a name="iot-settings"></a>From the [AWS IoT console](https://console.aws.amazon.com/iot/) navigation pane, choose **Settings**\.

   1. <a name="iot-settings-endpoint"></a>Under **Device data endpoint**, make a note of the value of **Endpoint**\. You use this value to replace the *AWS\_IOT\_ENDPOINT* placeholder in the commands in the following steps\.
**Note**  
Make sure that your [endpoints correspond to your certificate type](gg-core.md#certificate-endpoints)\.

1. On your computer \(not the AWS IoT Greengrass core device\), open two [command\-line](https://en.wikipedia.org/wiki/Command-line_interface) \(terminal or command prompt\) windows\. One window represents the HelloWorld\_Publisher client device and the other represents the HelloWorld\_Subscriber client device\.

   Upon execution, `basicDiscovery.py` attempts to collect information on the location of the AWS IoT Greengrass core at its endpoints\. This information is stored after the client device has discovered and successfully connected to the core\. This allows future messaging and operations to be executed locally \(without the need for an internet connection\)\.
**Note**  
Client IDs used for MQTT connections must match the thing name of the client device\. The `basicDiscovery.py` script sets the client ID for MQTT connections to the thing name that you specify when you run the script\.   
Run the following command from the folder that contains the `basicDiscovery.py` file for detailed script usage information:  

   ```
   python basicDiscovery.py --help
   ```

1. From the HelloWorld\_Publisher client device window, run the following commands\.
   + Replace *path\-to\-certs\-folder* with the path to the folder that contains the certificates, keys, and `basicDiscovery.py`\.
   + Replace *AWS\_IOT\_ENDPOINT* with your endpoint\.
   + Replace the two *publisherCertId* instances with the certificate ID in the file name for your HelloWorld\_Publisher client device\.

   ```
   cd path-to-certs-folder
   python basicDiscovery.py --endpoint AWS_IOT_ENDPOINT --rootCA AmazonRootCA1.pem --cert publisherCertId-certificate.pem.crt --key publisherCertId-private.pem.key --thingName HelloWorld_Publisher --topic 'hello/world/pubsub' --mode publish --message 'Hello, World! Sent from HelloWorld_Publisher'
   ```

   You should see output similar to the following, which includes entries such as `Published topic 'hello/world/pubsub': {"message": "Hello, World! Sent from HelloWorld_Publisher", "sequence": 1}`\.
**Note**  
If the script returns an `error: unrecognized arguments` message, change the single quotation marks to double quotation marks for the `--topic` and `--message` parameters and run the command again\.  
To troubleshoot a connection issue, you can try using [manual IP detection](#corp-network-manual-detection)\.  
![\[Screenshot of the publisher output.\]](http://docs.aws.amazon.com/greengrass/v1/developerguide/images/gg-get-started-076.png)

1. From the HelloWorld\_Subscriber client device window, run the following commands\.
   + Replace *path\-to\-certs\-folder* with the path to the folder that contains the certificates, keys, and `basicDiscovery.py`\.
   + Replace *AWS\_IOT\_ENDPOINT* with your endpoint\.
   + Replace the two *subscriberCertId* instances with the certificate ID in the file name for your HelloWorld\_Subscriber client device\.

   ```
   cd path-to-certs-folder
   python basicDiscovery.py --endpoint AWS_IOT_ENDPOINT --rootCA AmazonRootCA1.pem --cert subscriberCertId-certificate.pem.crt --key subscriberCertId-private.pem.key --thingName HelloWorld_Subscriber --topic 'hello/world/pubsub' --mode subscribe
   ```

   You should see the following output, which includes entries such as `Received message on topic hello/world/pubsub: {"message": "Hello, World! Sent from HelloWorld_Publisher", "sequence": 1}`\.  
![\[Screenshot of the subscriber output.\]](http://docs.aws.amazon.com/greengrass/v1/developerguide/images/gg-get-started-077.png)

Close the HelloWorld\_Publisher window to stop messages from accruing in the HelloWorld\_Subscriber window\.

Testing on a corporate network might interfere with connecting to the core\. As a workaround, you can manually enter the endpoint\. This ensures that the `basicDiscovery.py` script connects to the correct IP address of the AWS IoT Greengrass core device\.

**To manually enter the endpoint**

1. <a name="console-gg-groups"></a>In the AWS IoT console navigation pane, under **Manage**, expand **Greengrass devices**, and then choose **Groups \(V1\)**\.

1. Under **Greengrass groups**, choose your group\.

1. Configure the core to manually manage MQTT broker endpoints\. Do the following:

   1. On the group configuration page, choose the **Lambda functions** tab\.

   1. Under **System Lambda functions**, choose **IP detector**, and then choose **Edit**\.

   1. In the **Edit IP detector settings**, choose **Manually manage MQTT broker endpoints**, and then choose **Save**\.

1. Enter the MQTT broker endpoint for the core\. Do the following:

   1. Under **Overview**, choose the **Greengrass core**\.

   1. Under **MQTT broker endpoints**, choose **Manage endpoints**\.

   1. Choose **Add endpoint** and make sure that you have only one endpoint value\. This value must be the IP address endpoint for port 8883 of your AWS IoT Greengrass core device \(for example, `192.168.1.4`\)\.

   1. Choose **Update**\.