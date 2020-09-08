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
![\[Successful ping command output.\]](http://docs.aws.amazon.com/greengrass/latest/developerguide/images/gg-get-started-075.5.png)
**Note**  
If you're unable to ping an EC2 instance that's running AWS IoT Greengrass, make sure that the inbound security group rules for the instance allow ICMP traffic for [Echo request](https://docs.aws.amazon.com/AWSEC2/latest/UserGuide/security-group-rules-reference.html#sg-rules-ping) messages\. For more information, see [ Adding rules to a security group](https://docs.aws.amazon.com/AWSEC2/latest/UserGuide/using-network-security.html#adding-security-group-rule) in the *Amazon EC2 User Guide for Linux Instances*\.  
On Windows host computers, in the Windows Firewall with Advanced Security app, you might also need to enable an inbound rule that allows inbound echo requests \(for example, **File and Printer Sharing \(Echo Request \- ICMPv4\-In\)**\), or create one\.

1. Get your AWS IoT endpoint\.

   1. <a name="iot-settings"></a>In the [AWS IoT console](https://console.aws.amazon.com/iot/), in the navigation pane, choose **Settings**\.

   1. <a name="iot-settings-endpoint"></a>Under **Settings**, make a note of the value of **Endpoint**\. You use this value to replace the *AWS\_IOT\_ENDPOINT* placeholder in the commands in the following steps\.  
![\[AWS IoT endpoint value.\]](http://docs.aws.amazon.com/greengrass/latest/developerguide/images/gg-get-started-075.7.png)
**Note**  
Make sure that your [endpoints correspond to your certificate type](gg-core.md#certificate-endpoints)\.

1. On your computer \(not the AWS IoT Greengrass core device\), open two [command\-line](https://en.wikipedia.org/wiki/Command-line_interface) \(terminal or command prompt\) windows\. One window represents the HelloWorld\_Publisher device and the other represents the HelloWorld\_Subscriber device\.

   Upon execution, `basicPubSub.py` attempts to collect information on the location of the AWS IoT Greengrass core at its endpoints\. This information is stored after the device has discovered and successfully connected to the core\. This allows future messaging and operations to be executed locally \(without the need for an internet connection\)\.
**Note**  
You can run the following command from the folder that contains the `basicPubSub.py` file for detailed script usage information:  

   ```
   python basicPubSub.py --help
   ```

1. From the HelloWorld\_Publisher device window, run the following commands\.
   + Replace *path\-to\-certs\-folder* with the path to the folder that contains the certificates, keys, and `basicPubSub.py`\.
   + Replace *AWS\_IOT\_ENDPOINT* with your endpoint\.
   + Replace the two *publisher* instances with the hash in the file name for your HelloWorld\_Publisher device\.

   ```
   cd path-to-certs-folder
   python basicPubSub.py --endpoint AWS_IOT_ENDPOINT --rootCA root-ca-cert.pem --cert publisher.cert.pem --key publisher.private.key --clientId HelloWorld_Publisher --topic 'hello/world/pubsub' --mode publish --message 'Hello, World! Sent from HelloWorld_Publisher'
   ```

   You should see output similar to the following, which includes entries such as `Published topic 'hello/world/pubsub': {"message": "Hello, World! Sent from HelloWorld_Publisher", "sequence": 1}`\.
**Note**  
If the script returns an `error: unrecognized arguments` message, change the single quotation marks to double quotation marks for the `--topic` and `--message` parameters and run the command again\.  
To troubleshoot a connection issue, you can try using [manual IP detection](#corp-network-manual-detection)\.  
![\[Screenshot of the publisher output.\]](http://docs.aws.amazon.com/greengrass/latest/developerguide/images/gg-get-started-076.png)

1. From the HelloWorld\_Subscriber device window, run the following commands\.
   + Replace *path\-to\-certs\-folder* with the path to the folder that contains the certificates, keys, and `basicPubSub.py`\.
   + Replace *AWS\_IOT\_ENDPOINT* with your endpoint\.
   + Replace the two *subscriber* instances with the hash in the file name for your HelloWorld\_Subscriber device\.

   ```
   cd path-to-certs-folder
   python basicPubSub.py --endpoint AWS_IOT_ENDPOINT --rootCA root-ca-cert.pem --cert subscriber.cert.pem --key subscriber.private.key --clientId HelloWorld_Subscriber --topic 'hello/world/pubsub' --mode subscribe
   ```

   You should see the following output, which includes entries such as `Received message on topic hello/world/pubsub: {"message": "Hello, World! Sent from HelloWorld_Publisher", "sequence": 1}`\.  
![\[Screenshot of the subscriber output.\]](http://docs.aws.amazon.com/greengrass/latest/developerguide/images/gg-get-started-077.png)

Close the HelloWorld\_Publisher window to stop messages from accruing in the HelloWorld\_Subscriber window\.

Testing on a corporate network might interfere with connecting to the core\. As a workaround, you can manually enter the endpoint\. This ensures that the `basicPubSub.py` script connects to the correct IP address of the AWS IoT Greengrass core device\.

**To manually enter the endpoint**

1. Choose **Greengrass**, choose **Groups**, and then choose your group\.

1. Choose **Settings**\. 

1. For **Local connection detection**, choose **Manually manage connection information**, and then choose **View Cores for specific endpoint information**\.

1. Choose your core, and then choose **Connectivity**\.

1. Choose **Edit** and make sure that you have only one endpoint value\. This value must be the IP address endpoint for port 8883 of your AWS IoT Greengrass core device \(for example, `192.168.1.4`\)\.

1. Choose **Update**\.