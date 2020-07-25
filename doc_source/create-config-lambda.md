# Create and configure the Lambda function<a name="create-config-lambda"></a>

In this step, you create a Lambda function that tracks the number of cars that pass the traffic light\. Every time that the `GG_TrafficLight` shadow state changes to `G`, the Lambda function simulates the passing of a random number of cars \(from 1 to 20\)\. On every third `G` light change, the Lambda function sends basic statistics, such as min and max, to a DynamoDB table\.

1. On your computer, create a folder named `car_aggregator`\.

1. From the [TrafficLight ](https://github.com/aws/aws-greengrass-core-sdk-python/tree/master/examples/TrafficLight) examples folder on GitHub, download the `carAggregator.py` file to the `car_aggregator` folder\. This is your Lambda function code\.
**Note**  
This example Python file is stored in the AWS IoT Greengrass Core SDK repository for convenience, but it doesn't use the AWS IoT Greengrass Core SDK\.

1. If you aren't working in the US East \(N\. Virgina\) Region, open `carAggregator.py` and change `region_name` in the following line to the AWS Region that's currently selected in the AWS IoT console\. For the list of supported AWS Regions, see [AWS IoT Greengrass](https://docs.aws.amazon.com/general/latest/gr/greengrass.html) in the *Amazon Web Services General Reference*\.

   ```
   dynamodb = boto3.resource('dynamodb', region_name='us-east-1')
   ```

1. Run the following command in a [command\-line](https://en.wikipedia.org/wiki/Command-line_interface) window to install the [Boto 3 \- The AWS SDK for Python](https://github.com/boto/boto3/blob/develop/README.rst) package and its dependencies in the `car_aggregator` folder\. Greengrass Lambda functions use the AWS SDK to access other AWS services\. \(For Windows, use an [elevated command prompt](https://technet.microsoft.com/en-us/library/cc947813(v=ws.10).aspx)\.\)

   ```
   pip install boto3 -t path-to-car_aggregator-folder
   ```

   This results in a directory listing similar to the following:  
![\[Screenshot of directory listing showing carAggregator.py.\]](http://docs.aws.amazon.com/greengrass/latest/developerguide/images/gg-get-started-095.png)

1. Compress the contents of the `car_aggregator` folder into a `.zip` file named `car_aggregator.zip`\. \(Compress the folder's contents, not the folder\.\) This is your Lambda function deployment package\.

1. In the Lambda console, create a function named **GG\_Car\_Aggregator**, and set the remaining fields as follows:
   + For **Runtime**, choose **Python 3\.7**\.
   + For **Permissions**, keep the default setting\. This creates an execution role that grants basic Lambda permissions\. This role isn't used by AWS IoT Greengrass\.

   Choose **Create function**\.  
![\[Basic information section with Function name set to GG_Car_Aggregator and Runtime set to Python 3.7.\]](http://docs.aws.amazon.com/greengrass/latest/developerguide/images/gg-get-started-095.5.png)

1. Upload your Lambda function deployment package:

   1. On the **Configuration** tab, under **Function code**, set the following fields:
      + For **Code entry type**, choose **Upload a \.zip file**\.
      + For **Runtime**, choose **Python 3\.7**\.
      + For **Handler**, enter **carAggregator\.function\_handler**

   1. Choose **Upload**, and then choose `car_aggregator.zip`\.

   1. Choose **Save**\.  
![\[Screenshot of GG_Car_Aggregator with the Handler field set to carAggregator.function_handler.\]](http://docs.aws.amazon.com/greengrass/latest/developerguide/images/gg-get-started-096.png)

1. Publish the Lambda function, and then create an alias named **GG\_CarAggregator**\. For step\-by\-step instructions, see the steps to [publish the Lambda function](create-lambda.md#publish-function-version) and [create an alias](create-lambda.md#create-version-alias) in Module 3 \(Part 1\)\.

1. In the AWS IoT console, add the Lambda function that you just created to your AWS IoT Greengrass group:

   1. On the group configuration page, choose **Lambdas**, and then choose **Add Lambda**\.

   1. Choose **Use existing Lambda**\.  
![\[Use existing Lambda highlighted.\]](http://docs.aws.amazon.com/greengrass/latest/developerguide/images/gg-get-started-096.2.png)

   1. Choose **GG\_Car\_Aggregator**, and then choose **Next**\.  
![\[GG_Car_Aggregator and Next highlighted.\]](http://docs.aws.amazon.com/greengrass/latest/developerguide/images/gg-get-started-096.3.png)

   1. Choose **Alias: GG\_CarAggregator**, and then choose **Finish**\.  
![\[Alias: GG_CarAggregator and Finish highlighted.\]](http://docs.aws.amazon.com/greengrass/latest/developerguide/images/gg-get-started-096.4.png)
**Note**  
You can remove other Lambda functions from earlier modules\.

1. Edit the Lambda function configuration:

   1. Choose the ellipsis \(**…**\) associated with the Lambda function, and then choose **Edit Configuration**\.  
![\[The Edit Configuration option highlighted for the Lambda function.\]](http://docs.aws.amazon.com/greengrass/latest/developerguide/images/gg-get-started-097.5.png)

   1. Under **Memory limit**, enter **64 MB**\.

   1. Under **Lambda lifecycle**, choose **Make this function long\-lived and keep it running indefinitely**, and then choose **Update**\.  
![\[GG_Car_Aggregator configuration page with Make this function long-lived and keep it running indefinitely highlighted.\]](http://docs.aws.amazon.com/greengrass/latest/developerguide/images/gg-get-started-098.png)