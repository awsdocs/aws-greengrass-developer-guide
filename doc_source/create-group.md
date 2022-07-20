--------

AWS IoT Greengrass Version 1 no longer receives feature updates, and will receive only security patches and bug fixes until June 30, 2023\. For more information, see the [AWS IoT Greengrass V1 maintenance policy](https://docs.aws.amazon.com/greengrass/v1/developerguide/maintenance-policy.html)\. We strongly recommend that you [migrate to AWS IoT Greengrass Version 2](https://docs.aws.amazon.com/greengrass/v2/developerguide/move-from-v1.html), which adds [significant new features](https://docs.aws.amazon.com/greengrass/v2/developerguide/greengrass-v2-whats-new.html) and [support for additional platforms](https://docs.aws.amazon.com/greengrass/v2/developerguide/operating-system-feature-support-matrix.html)\.

--------

# Create an AWS IoT Greengrass group for the core<a name="create-group"></a>

AWS IoT Greengrass *groups* contain settings and other information about its components, such as client devices, Lambda functions, and connectors\. A group defines the configuration for a core, including how its components can interact with each other\.

In this section, you create a group for your core\.

**Tip**  
For an example that uses the AWS IoT Greengrass API to create and deploy a group, see the [gg\_group\_setup](https://github.com/awslabs/aws-greengrass-group-setup) repository on GitHub\.

**To create a group for the core**

1. Navigate to the [AWS IoT console](https://console.aws.amazon.com/iot)\.

1. Under **Manage**, expand **Greengrass devices**, and choose **Groups \(V1\)**\.
**Note**  
If you don't see the **Greengrass devices** menu, change to an AWS Region that supports AWS IoT Greengrass V1\. For the list of supported Regions, see [AWS IoT Greengrass V1 endpoints and quotas](https://docs.aws.amazon.com/general/latest/gr/greengrass.html) in the *AWS General Reference*\. You must [create the AWS IoT thing for your core](provision-core.md) in a Region where AWS IoT Greengrass V1 is available\.

1. On the **Greengrass groups** page, choose **Create group**\.

1. On the **Create Greengrass group** page, do the following:

   1. For **Greengrass group name**, enter a name that describes the group, such as **MyGreengrassGroup**\.

   1. For **Greengrass core**, choose the AWS IoT thing that you created earlier, such as **MyGreengrassV1Core**\.

      The console automatically selects the thing's device certificate for you\.

   1. Choose **Create group**\.