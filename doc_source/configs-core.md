# Deploy Cloud Configurations to an AWS IoT Greengrass Core Device<a name="configs-core"></a>

1. Make sure that your AWS IoT Greengrass core device is connected to the internet\. \(For example, see if you can successfully navigate to a webpage\.\)

1. Make sure that the AWS IoT Greengrass daemon is running on your core device\. Run the following commands in your core device terminal\.

   1. To check whether the daemon is running:

      ```
      ps aux | grep -E 'greengrass.*daemon'
      ```

      If the output contains a `root` entry for `/greengrass/ggc/packages/1.9.2/bin/daemon`, then the daemon is running\.

   1. To start the daemon:

      ```
      cd /greengrass/ggc/core/
      sudo ./greengrassd start
      ```

   Now you're ready to deploy the Lambda function and subscription configurations to your AWS IoT Greengrass core device\.

1. In the AWS IoT Core console, on the group configuration page, from **Actions**, choose **Deploy**\.  
![\[Screenshot of the Group page with Deployments, Actions menu, and Deploy highlighted.\]](http://docs.aws.amazon.com/greengrass/latest/developerguide/images/gg-get-started-040.png)

1. On the **Configure how devices discover your core** page, choose **Automatic detection**\. This enables devices to automatically acquire connectivity information for the core, such as IP address, DNS, and port number\. Automatic detection is recommended, but AWS IoT Greengrass also supports manually specified endpoints\. You're only prompted for the discovery method the first time that the group is deployed\.  
![\[Screenshot of Configure how Devices discover your Core with Automatic detection highlighted.\]](http://docs.aws.amazon.com/greengrass/latest/developerguide/images/console-discovery.png)

1. If prompted, on the **Grant permission to access other services** page, choose **Grant permission**\. This configures the Greengrass service role, which allows AWS IoT Greengrass to access other AWS services on your behalf\. This role is required for deployments to succeed\. For more information, see [Greengrass Service Role](service-role.md)\.

The first deployment might take a few minutes\. When the deployment is complete, you should see **Successfully completed** in the **Status** column on the **Deployments** page:

**Note**  
The deployment status is also displayed below the group's name on the page header\.

![\[Screenshot showing a status of Successfully completed.\]](http://docs.aws.amazon.com/greengrass/latest/developerguide/images/gg-get-started-042.png)

For troubleshooting help, see [Troubleshooting AWS IoT Greengrass](gg-troubleshooting.md)\.