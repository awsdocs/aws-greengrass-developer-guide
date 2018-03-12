# Create and Configure the Lambda Function<a name="create-config-lambda"></a>

In this step, you create a Lambda function that tracks the number of cars that pass the traffic light\. Every time that the `GG_TrafficLight` shadow state changes to `G`, the Lambda simulates the passing of a randomized number of cars \(from 1 to 20\)\. On every third `G` light change, the Lambda sends basic statistics, such as min and max, to a DynamoDB table\.

1. On your computer, create a folder named `car_aggregator`\.

1. From the [GitHub repository](https://github.com/aws-samples/aws-greengrass-samples/tree/master/traffic-light-example-python) download the `carAggregator.py` Lambda function to the `car_aggregator` folder\.

1. Install the **boto3** package \(AWS SDK for Python\) and its dependencies in the `car_aggregator` folder by running the following command in a [command\-line](https://en.wikipedia.org/wiki/Command-line_interface) window \(for Windows, use an [elevated command prompt](https://technet.microsoft.com/en-us/library/cc947813(v=ws.10).aspx)\):

   ```
   pip install boto3 -t path-to-car_aggregator-folder
   ```

   This results in a directory listing similar to the following:  
![\[Screenshot of directory listing showing carAggregator.py.\]](http://docs.aws.amazon.com/greengrass/latest/developerguide/images/gg-get-started-095.png)

   Greengrass Lambda functions use the AWS SDK to access other Amazon Web Services\. For more information, see [ Boto 3 \- The AWS SDK for Python](https://github.com/boto/boto3/blob/develop/README.rst)\. 

1. Compress the contents of the `car_aggregator` folder into a `.zip` file named `car_aggregator.zip`\. This is your Lambda function deployment package\.

1. In the Lambda console, create a function named **GG\_Car\_Aggregator**, and set the remaining fields as follows:

   + **Runtime** \- choose **Python 2\.7**\.

   + **Role** \- choose **Choose an existing role**\.

   + **Existing role** \- choose **Lambda\_DynamoDB\_Role**\.

   Then, choose **Create function**\.  
![\[Author from scratch panel with Name set to GG_Car_Aggregator, Runtime to Python 2.7, Role to Choose an existing role and Existing role to Lambda_DynamoDB_Role.\]](http://docs.aws.amazon.com/greengrass/latest/developerguide/images/gg-get-started-095.5.png)

1. Upload your Lambda function deployment package, as follows:

   1. On the **Configuration** tab, under **Function code**, set the following fields:

      + **Code entry type** \- choose **Upload a \.ZIP file**\.

      + **Runtime** \- choose **Python 2\.7**\.

      + **Handler** \- type **carAggregator\.function\_handler**\.

   1. Choose **Upload**, and then choose `car_aggregator.zip`\.

   1. Choose **Save**\.  
![\[Screenshot of GG_Car_Aggregator with the Handler field set to carAggregator.function_handler.\]](http://docs.aws.amazon.com/greengrass/latest/developerguide/images/gg-get-started-096.png)

1. Publish the Lambda function, and then create an alias named **GG\_CarAggregator**\. For step\-by\-step instructions, see the [Publish the Lambda function](create-lambda.md#publish-function-version) and [Create an alias](create-lambda.md#create-version-alias) steps in Module 3 \(Part 1\)\.

1. In the AWS IoT console, add the Lambda function that you just created to your AWS Greengrass group, as follows:

   1. On the group configuration page, choose **Lambdas**, and then choose **Add Lambda**:

   1. Choose **Use existing Lambda**:  
![\[Use existing Lambda highlighted.\]](http://docs.aws.amazon.com/greengrass/latest/developerguide/images/gg-get-started-096.2.png)

   1. Choose **GG\_Car\_Aggregator**, and then choose **Next**:  
![\[GG_Car_Aggregator and Next highlighted.\]](http://docs.aws.amazon.com/greengrass/latest/developerguide/images/gg-get-started-096.3.png)

   1. Choose **Alias: GG\_CarAggregator**, and then choose **Finish**:  
![\[Alias: GG_CarAggregator and Finish highlighted.\]](http://docs.aws.amazon.com/greengrass/latest/developerguide/images/gg-get-started-096.4.png)
**Note**  
You can remove other Lambda functions from earlier modules\.

1. Edit the Lambda function configuration, as follows:

   1. Choose the ellipsis \(**…**\) associated with the Lambda function, then choose **Edit Configuration**:  
![\[The Edit Configuration option highlighted for the Lambda function.\]](http://docs.aws.amazon.com/greengrass/latest/developerguide/images/gg-get-started-097.5.png)

   1. Under **Lambda lifecycle**, select **Make this function long\-lived and keep it running indefinitely**, and then choose **Update**:  
![\[GG_Car_Aggregator configuration page with Make this function long-lived and keep it running indefinitely highlighted.\]](http://docs.aws.amazon.com/greengrass/latest/developerguide/images/gg-get-started-098.png)