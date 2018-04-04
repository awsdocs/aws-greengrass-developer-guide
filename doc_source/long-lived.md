# Configure Long\-Lived Lambda Functions for AWS Greengrass<a name="long-lived"></a>

You are now ready to configure your Lambda function for AWS Greengrass\.

1. In the AWS IoT console, under **Greengrass**, choose **Groups**, and then choose the group that you created in [Module 2](module2.md)\.

1. On the group configuration page, choose **Lambdas**, and then choose **Add Lambda**\.

1. On the **Add a Lambda to your Greengrass Group** page, choose **Use existing Lambda**\.

1. On the **Use existing Lambda page**, choose **Greengrass\_HelloWorld\_Counter**, and then choose **Next**\.  
![\[Screenshot with Greengrass_HelloWorld_Counter and the Next button highlighted.\]](http://docs.aws.amazon.com/greengrass/latest/developerguide/images/gg-get-started-049.png)

1. On the **Select a Lambda version** page, choose **Alias: GG\_HW\_Counter**, and then choose **Finish**\.

1. On the **Lambdas** page, from the **Greengrass\_HelloWorld\_Counter** menu, choose **Edit Configuration**\.  
![\[Screenshot of the Group Configuration page with Lambdas in the left nav and Edit Configuration highlighted.\]](http://docs.aws.amazon.com/greengrass/latest/developerguide/images/gg-get-started-050.png)

1. On the configuration page, edit the following properties:
   + **Timeout** \- set to 25 seconds\. This Lambda function sleeps for 20 seconds before each invocation\.
   + **Lambda lifecycle** \- choose **Make this function long\-lived and keep it running indefinitely**\.

      
![\[Screenshot with the Timeout field set to 25 seconds and the Make this function long-lived and keep it running indefinitely radio button selected.\]](http://docs.aws.amazon.com/greengrass/latest/developerguide/images/gg-get-started-051.png)

1. Choose **Update**\.