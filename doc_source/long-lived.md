--------

You are viewing the documentation for AWS IoT Greengrass Version 1\. AWS IoT Greengrass Version 2 is the latest major version of AWS IoT Greengrass\. For more information about using AWS IoT Greengrass V2, see the [https://docs.aws.amazon.com/greengrass/v2/developerguide](https://docs.aws.amazon.com/greengrass/v2/developerguide)\.

--------

# Configure long\-lived Lambda functions for AWS IoT Greengrass<a name="long-lived"></a>

You are now ready to configure your Lambda function for AWS IoT Greengrass\.

1. <a name="console-gg-groups"></a>In the AWS IoT console, in the navigation pane, choose **Greengrass**, **Classic \(V1\)**, **Groups**\.

1. Under **Greengrass groups**, choose the group that you created in [Module 2](module2.md)\.

1. On the group configuration page, choose **Lambdas**, and then choose **Add Lambda**\.

1. On the **Add a Lambda to your Greengrass Group** page, choose **Use existing Lambda**\.

1. On the **Use existing Lambda** page, choose **Greengrass\_HelloWorld\_Counter**, and then choose **Next**\.  
![\[Screenshot with Greengrass_HelloWorld_Counter and the Next button highlighted.\]](http://docs.aws.amazon.com/greengrass/v1/developerguide/images/gg-get-started-049.png)

1. On the **Select a Lambda version** page, choose **Alias: GG\_HW\_Counter**, and then choose **Finish**\.

1. On the **Lambdas** page, from the **…** menu for the new function, choose **Edit Configuration**\.  
![\[Screenshot of the Group Configuration page with Lambdas in the navigation pane and Edit Configuration highlighted.\]](http://docs.aws.amazon.com/greengrass/v1/developerguide/images/gg-get-started-050.png)

1. On the **Group\-specific Lambda configuration** page, edit the following properties:
   + Set **Timeout** to 25 seconds\. This Lambda function sleeps for 20 seconds before each invocation\.
   + For **Lambda lifecycle**, choose **Make this function long\-lived and keep it running indefinitely**\.
   + Keep the default values for all other fields, such as **Run as** and **Containerization**\.

      
![\[Screenshot with the Timeout field set to 25 seconds and the Make this function long-lived and keep it running indefinitely radio button selected.\]](http://docs.aws.amazon.com/greengrass/v1/developerguide/images/gg-get-started-051.png)

1. Choose **Update**\.