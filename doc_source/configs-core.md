--------

You are viewing the documentation for AWS IoT Greengrass Version 1\. AWS IoT Greengrass Version 2 is the latest major version of AWS IoT Greengrass\. For more information about using AWS IoT Greengrass V2, see the [https://docs.aws.amazon.com/greengrass/v2/developerguide](https://docs.aws.amazon.com/greengrass/v2/developerguide)\.

--------

# Deploy cloud configurations to a Greengrass core device<a name="configs-core"></a>

1. Make sure that your Greengrass core device is connected to the internet\. For example, try successfully navigating to a webpage\.

1. Make sure that the Greengrass daemon is running on your core device\. In your core device terminal, run the following commands to check whether the daemon is running and start it, if needed\.

   1. To check whether the daemon is running:

      ```
      ps aux | grep -E 'greengrass.*daemon'
      ```

      If the output contains a `root` entry for `/greengrass/ggc/packages/1.11.4/bin/daemon`, then the daemon is running\.

   1. To start the daemon:

      ```
      cd /greengrass/ggc/core/
      sudo ./greengrassd start
      ```

   Now you're ready to deploy the Lambda function and subscription configurations to your Greengrass core device\.

1. <a name="console-gg-groups"></a>In the AWS IoT console, in the navigation pane, choose **Greengrass**, **Classic \(V1\)**, **Groups**\.

1. Under **Greengrass groups**, choose the group that you created in [Module 2](module2.md)\.

1. On the group configuration page, from **Actions**, choose **Deploy**\.  
![\[Screenshot of the Group page with Deployments, Actions menu, and Deploy highlighted.\]](http://docs.aws.amazon.com/greengrass/v1/developerguide/images/gg-get-started-040.png)

1. On the **Configure how devices discover your core** page, choose **Automatic detection**\. This enables devices to automatically acquire connectivity information for the core, such as IP address, DNS, and port number\. Automatic detection is recommended, but AWS IoT Greengrass also supports manually specified endpoints\. You're only prompted for the discovery method the first time that the group is deployed\.  
![\[Screenshot of Configure how Devices discover your Core with Automatic detection highlighted.\]](http://docs.aws.amazon.com/greengrass/v1/developerguide/images/console-discovery.png)

The first deployment might take a few minutes\. When the deployment is complete, you should see **Successfully completed** in the **Status** column on the **Deployments** page:

**Note**  
The deployment status is also displayed below the group's name on the page header\.

![\[Screenshot showing a status of Successfully completed.\]](http://docs.aws.amazon.com/greengrass/v1/developerguide/images/gg-get-started-042.png)

For troubleshooting help, see [Troubleshooting AWS IoT Greengrass](gg-troubleshooting.md)\.