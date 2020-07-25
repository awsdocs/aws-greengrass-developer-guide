# How to create a secret resource \(console\)<a name="secrets-console"></a>

This feature is available for AWS IoT Greengrass Core v1\.7 and later\.

This tutorial shows how to use the AWS Management Console to add a *secret resource* to a Greengrass group\. A secret resource is a reference to a secret from AWS Secrets Manager\. For more information, see [Deploy secrets to the AWS IoT Greengrass core](secrets.md)\.

On the AWS IoT Greengrass core device, connectors and Lambda functions can use the secret resource to authenticate with services and applications, without hard\-coding passwords, tokens, or other credentials\.

In this tutorial, you start by creating a secret in the AWS Secrets Manager console\. Then, in the AWS IoT Greengrass console, you add a secret resource to a Greengrass group from the group's **Resources** page\. This secret resource references the Secrets Manager secret\. Later, you attach the secret resource to a Lambda function, which allows the function to get the value of the local secret\.

**Note**  
Alternatively, the console allows you to create a secret and secret resource when you configure a connector or Lambda function\. You can do this from the connector's **Configure parameters** page or the Lambda function's **Resources** page\.  
Only connectors that contain parameters for secrets can access secrets\. For a tutorial that shows how the Twilio Notifications connector uses a locally stored authentication token, see [Getting started with Greengrass connectors \(console\)](connectors-console.md)\.

The tutorial contains the following high\-level steps:

1. [Create a Secrets Manager secret](#secrets-console-create-secret)

1. [Add a secret resource to a group](#secrets-console-create-resource)

1. [Create a Lambda function deployment package](#secrets-console-create-deployment-package)

1. [Create a Lambda function](#secrets-console-create-function)

1. [Add the function to the group](#secrets-console-create-gg-function)

1. [Attach the secret resource to the function](#secrets-console-affiliate-gg-function)

1. [Add subscriptions to the group](#secrets-console-create-subscription)

1. [Deploy the group](#secrets-console-create-deployment)

1. [Test the Lambda function](#secrets-console-test-solution)

The tutorial should take about 20 minutes to complete\.

## Prerequisites<a name="secrets-console-prerequisites"></a>

To complete this tutorial, you need:
+ A Greengrass group and a Greengrass core \(v1\.7 or later\)\. To learn how to create a Greengrass group and core, see [Getting started with AWS IoT Greengrass](gg-gs.md)\. The Getting Started tutorial also includes steps for installing the AWS IoT Greengrass Core software\.
+ AWS IoT Greengrass must be configured to support local secrets\. For more information, see [Secrets Requirements](secrets.md#secrets-reqs)\.
**Note**  
This requirement includes allowing access to your Secrets Manager secrets\. If you're using the default Greengrass service role, Greengrass has permission to get the values of secrets with names that start with *greengrass\-*\.
+ To get the values of local secrets, your user\-defined Lambda functions must use AWS IoT Greengrass Core SDK v1\.3\.0 or later\.

## Step 1: Create a Secrets Manager secret<a name="secrets-console-create-secret"></a>

In this step, you use the AWS Secrets Manager console to create a secret\.

1. <a name="create-secret-step-signin"></a>Sign in to the [AWS Secrets Manager console](https://console.aws.amazon.com/secretsmanager/)\.
**Note**  
For more information about this process, see [ Step 1: Create and store your secret in AWS Secrets Manager](https://docs.aws.amazon.com/secretsmanager/latest/userguide/tutorials_basic.html) in the *AWS Secrets Manager User Guide*\.

1. <a name="create-secret-step-create"></a>Choose **Store a new secret**\.

1. <a name="create-secret-step-othertype"></a>Under **Select secret type**, choose **Other type of secrets**\.

1. Under **Specify the key\-value pairs to be stored for this secret**:
   + For **Key**, enter **test**\.
   + For **Value**, enter **abcdefghi**\.  
![\[Specifying the secret's key and value in the Secrets Manager console.\]](http://docs.aws.amazon.com/greengrass/latest/developerguide/images/secrets-manager-key-value.png)

1. <a name="create-secret-step-encryption"></a>Keep **DefaultEncryptionKey** selected for the encryption key, and then choose **Next**\.
**Note**  
You aren't charged by AWS KMS if you use the default AWS managed key that Secrets Manager creates in your account\.

1. For **Secret name**, enter **greengrass\-TestSecret**, and then choose **Next**\.
**Note**  
By default, the Greengrass service role allows AWS IoT Greengrass to get the value of secrets with names that start with *greengrass\-*\. For more information, see [secrets requirements](secrets.md#secrets-reqs)\.

1. <a name="create-secret-step-rotation"></a>This tutorial doesn't require rotation, so choose **Disable automatic rotation**, and then choose **Next**\.

1. <a name="create-secret-step-review"></a>On the **Review** page, review your settings, and then choose **Store**\.

   Next, you create a secret resource in your Greengrass group that references the secret\.

## Step 2: Add a secret resource to a Greengrass group<a name="secrets-console-create-resource"></a>

In this step, you configure a group resource that references the Secrets Manager secret\.

1. <a name="console-gg-groups"></a>In the AWS IoT console, choose **Greengrass**, and then choose **Groups**\.

1. <a name="create-secret-resource-step-choosegroup"></a>Choose the group that you want to add the secret resource to\.

1. <a name="create-secret-resource-step-secretstab"></a>On the group configuration page, choose **Resources**, and then choose **Secret**\. This tab displays the secret resources that belong to the group\. You can add, edit, and remove secret resources from this tab\.  
![\[The Secret tab on the group's Resources page.\]](http://docs.aws.amazon.com/greengrass/latest/developerguide/images/console-group-resources-secret-zero.png)
**Note**  
Alternatively, the console allows you to create a secret and secret resource when you configure a connector or Lambda function\. You can do this from the connector's **Configure parameters** page or the Lambda function's **Resources** page\.

1. <a name="create-secret-resource-step-addsecretresource"></a>Choose **Add a secret resource**\.

1. On the **Add a secret resource to your group** page, choose **Select**, and then choose **greengrass\-TestSecret**\.  
![\[The Add a secret resource to your group page with Select highlighted.\]](http://docs.aws.amazon.com/greengrass/latest/developerguide/images/console-group-resources-secret-add-select.png)

1. <a name="create-secret-resource-step-selectlabels"></a>On the **Select labels \(Optional\)** page, choose **Next**\. The AWSCURRENT staging label represents the latest version of the secret\. This label is always included in a secret resource\.
**Note**  
This tutorial requires the AWSCURRENT label only\. You can optionally include labels that are required by your Lambda function or connector\.

1. On the **Name your secret resource** page, enter **MyTestSecret**, and then choose **Save**\.

## Step 3: Create a Lambda function deployment package<a name="secrets-console-create-deployment-package"></a>

To create a Lambda function, you must first create a Lambda function *deployment package* that contains the function code and dependencies\. Greengrass Lambda functions require the [AWS IoT Greengrass Core SDK](lambda-functions.md#lambda-sdks-core) for tasks such as communicating with MQTT messages in the core environment and accessing local secrets\. This tutorial creates a Python function, so you use the Python version of the SDK in the deployment package\.

**Note**  
To get the values of local secrets, your user\-defined Lambda functions must use AWS IoT Greengrass Core SDK v1\.3\.0 or later\.

1. <a name="download-ggc-sdk"></a> From the [AWS IoT Greengrass Core SDK](what-is-gg.md#gg-core-sdk-download) downloads page, download the AWS IoT Greengrass Core SDK for Python to your computer\.

1. <a name="unzip-ggc-sdk"></a>Unzip the downloaded package to get the SDK\. The SDK is the `greengrasssdk` folder\.

1. Save the following Python code function in a local file named `secret_test.py`\.

   ```
   import greengrasssdk
    
   # Create SDK clients.
   secrets_client = greengrasssdk.client('secretsmanager')
   message_client = greengrasssdk.client('iot-data')
   message = ''
   
   # This handler is called when the function is invoked.
   # It uses the 'secretsmanager' client to get the value of the test secret using the secret name.
   # The test secret is a text type, so the SDK returns a string. 
   # For binary secret values, the SDK returns a base64-encoded string.
   def function_handler(event, context):
       response = secrets_client.get_secret_value(SecretId='greengrass-TestSecret')
       secret_value = response.get('SecretString')
       if secret_value is None:
           message = 'Failed to retrieve secret.'
       else:
           message = 'Success! Retrieved secret.'
       
       message_client.publish(topic='secrets/output', payload=message)
       print('published: ' + message)
   ```

   The `get_secret_value` function supports the name or ARN of the Secrets Manager secret for the `SecretId` value\. This example uses the secret name\. For this example secret, AWS IoT Greengrass returns the key\-value pair: `{"test":"abcdefghi"}`\.
**Important**  
Make sure that your user\-defined Lambda functions handle secrets securely and don't log any any sensitive data that's stored in the secret\. For more information, see [ Mitigate the Risks of Logging and Debugging Your Lambda Function](https://docs.aws.amazon.com/secretsmanager/latest/userguide/best-practices.html#best-practice_lamda-debug-statements) in the *AWS Secrets Manager User Guide*\. Although this documentation specifically refers to rotation functions, the recommendation also applies to Greengrass Lambda functions\.

1. Zip the following items into a file named `secret_test_python.zip`\. When you create the ZIP file, include only the code and dependencies, not the containing folder\.
   + **secret\_test\.py**\. App logic\.
   + **greengrasssdk**\. Required library for all Python Greengrass Lambda functions\.

   This is your Lambda function deployment package\.

## Step 4: Create a Lambda function<a name="secrets-console-create-function"></a>

In this step, you use the AWS Lambda console to create a Lambda function and configure it to use your deployment package\. Then, you publish a function version and create an alias\.

1. First, create the Lambda function\.

   1. <a name="lambda-console-open"></a>In the AWS Management Console, choose **Services**, and open the AWS Lambda console\.

   1. <a name="lambda-console-create-function"></a>Choose **Create function** and then choose **Author from scratch**\.

   1. In the **Basic information** section, use the following values:
      + For **Function name**, enter **SecretTest**\.
      + For **Runtime**, choose **Python 3\.7**\.
      + For **Permissions**, keep the default setting\. This creates an execution role that grants basic Lambda permissions\. This role isn't used by AWS IoT Greengrass\.

   1. <a name="lambda-console-save-function"></a>At the bottom of the page, choose **Create function**\.

1. Next, register the handler and upload your Lambda function deployment package\.

   1. On the **Configuration** tab for the SecretTest function, in **Function code**, use the following values:
      + For **Code entry type**, choose **Upload a \.zip file**\.
      + For **Runtime**, choose **Python 3\.7**\.
      + For **Handler**, enter **secret\_test\.function\_handler**

   1. <a name="lambda-console-upload"></a>Choose **Upload**\.

   1. Choose your `secret_test_python.zip` deployment package\.

   1. <a name="lambda-console-save-config"></a>Choose **Save**\.
**Note**  
The **Test** button on the AWS Lambda console doesn't work with this function\. The AWS IoT Greengrass Core SDK doesn't contain modules that are required to run your Greengrass Lambda functions independently in the AWS Lambda console\. These modules \(for example, `greengrass_common`\) are supplied to the functions after they are deployed to your Greengrass core\.
**Tip**  
You can see your code in the **Function code** section by choosing **Edit code inline** from the **Code entry type** menu\.

1. Now, publish the first version of your Lambda function and create an [alias for the version](https://docs.aws.amazon.com/lambda/latest/dg/versioning-aliases.html)\.
**Note**  
Greengrass groups can reference a Lambda function by alias \(recommended\) or by version\. Using an alias makes it easier to manage code updates because you don't have to change your subscription table or group definition when the function code is updated\. Instead, you just point the alias to the new function version\.

   1. <a name="shared-publish-function-version"></a>From the **Actions** menu, choose **Publish new version**\.

   1. <a name="shared-publish-function-version-description"></a>For **Version description**, enter **First version**, and then choose **Publish**\.

   1. On the **SecretTest: 1** configuration page, from the **Actions** menu, choose **Create alias**\.

   1. On the **Create a new alias** page, use the following values:
      + For **Name**, enter **GG\_SecretTest**\.
      + For **Version**, choose **1**\.
**Note**  
AWS IoT Greengrass doesn't support Lambda aliases for **$LATEST** versions\.

   1. Choose **Create**\.

Now you're ready to add the Lambda function to your Greengrass group and attach the secret resource\.

## Step 5: Add the Lambda function to the Greengrass group<a name="secrets-console-create-gg-function"></a>

In this step, you add the Lambda function to the Greengrass group in the AWS IoT console\.

1. <a name="choose-add-lambda"></a>On the group configuration page, choose **Lambdas**, and then choose **Add Lambda**\.  
![\[The group page with Lambdas and Add Lambda highlighted.\]](http://docs.aws.amazon.com/greengrass/latest/developerguide/images/console-group-lambdas.png)

1. <a name="add-lambda-to-group-console"></a>On the **Add a Lambda to your Greengrass Group** page, choose **Use existing Lambda**\.  
![\[The Add a Lambda to your Greengrass Group page with Use existing Lambda highlighted.\]](http://docs.aws.amazon.com/greengrass/latest/developerguide/images/console-group-lambdas-existing-lambda.png)

1. On the **Use existing Lambda** page, choose **SecretTest**, and then choose **Next**\.

1. On the **Select a Lambda version** page, choose **Alias:GG\_SecretTest**, and then choose **Finish**\.

Next, affiliate the secret resource with the function\.

## Step 6: Attach the secret resource to the Lambda function<a name="secrets-console-affiliate-gg-function"></a>

In this step, you attach the secret resource to the Lambda function in your Greengrass group\. This affiliates the resource with the function, which allows the function to get the value of the local secret\.

1. On the group's **Lambdas** page, choose the **SecretTest** function\.

1. On the function's details page, choose **Resources**, choose **Secret**, and then choose **Attach a secret resource**\.  
![\[The Resources page with Attach a secret resource highlighted.\]](http://docs.aws.amazon.com/greengrass/latest/developerguide/images/console-group-function-resources-secret-attach.png)

1. On the **Attach a secret resource to your Lambda function** page, choose **Choose secret resource**\.

1. On the **Select a secret resource from your group** page, choose **MyTestSecret**, and then choose **Save**\.

## Step 7: Add subscriptions to the Greengrass group<a name="secrets-console-create-subscription"></a>

In this step, you add subscriptions that allow AWS IoT and the Lambda function to exchange messages\. One subscription allows AWS IoT to invoke the function, and one allows the function to send output data to AWS IoT\.

1. <a name="shared-subscriptions-addsubscription"></a>On the group configuration page, choose **Subscriptions**, and then choose **Add Subscription**\.  
![\[The group page with Subscriptions and Add Subscription highlighted.\]](http://docs.aws.amazon.com/greengrass/latest/developerguide/images/console-group-subscriptions.png)

1. Create a subscription that allows AWS IoT to publish messages to the function\.

   On the **Select your source and target** page, configure the source and target:

   1. For **Select a source**, choose **Services**, and then choose **IoT Cloud**\.

   1. For **Select a target**, choose **Lambdas**, and then choose **SecretTest**\.

   1. Choose **Next**\.

1. On the **Filter your data with a topic** page, for **Topic filter**, enter **secrets/input**, and then choose **Next**\.

1. Choose **Finish**\.

1. Repeat steps 1 \- 4 to create a subscription that allows the function to publish status to AWS IoT\.

   1. For **Select a source**, choose **Lambdas**, and then choose **SecretTest**\.

   1. For **Select a target**, choose **Services**, and then choose **IoT Cloud**\.

   1. For **Topic filter**, enter **secrets/output**\.

## Step 8: Deploy the Greengrass group<a name="secrets-console-create-deployment"></a>

Deploy the group to the core device\. During deployment, AWS IoT Greengrass fetches the value of the secret from Secrets Manager and creates a local, encrypted copy on the core\.

1. <a name="shared-deploy-group-checkggc"></a>Make sure that the AWS IoT Greengrass core is running\. Run the following commands in your Raspberry Pi terminal, as needed\.

   1. To check whether the daemon is running:

      ```
      ps aux | grep -E 'greengrass.*daemon'
      ```

      If the output contains a `root` entry for `/greengrass/ggc/packages/ggc-version/bin/daemon`, then the daemon is running\.
**Note**  
The version in the path depends on the AWS IoT Greengrass Core software version that's installed on your core device\.

   1. To start the daemon:

      ```
      cd /greengrass/ggc/core/
      sudo ./greengrassd start
      ```

1. <a name="shared-deploy-group-deploy"></a>On the group configuration page, choose **Deployments**, and from the **Actions** menu, choose **Deploy**\.  
![\[The group page with Deployments and Deploy highlighted.\]](http://docs.aws.amazon.com/greengrass/latest/developerguide/images/console-group-deployments-deploy.png)

1. <a name="shared-deploy-group-ipconfig"></a>If prompted, on the **Configure how devices discover your core** page, choose **Automatic detection**\.

   This enables devices to automatically acquire connectivity information for the core, such as IP address, DNS, and port number\. Automatic detection is recommended, but AWS IoT Greengrass also supports manually specified endpoints\. You're only prompted for the discovery method the first time that the group is deployed\.  
![\[The Configure how devices discover your core page with Automatic detection highlighted.\]](http://docs.aws.amazon.com/greengrass/latest/developerguide/images/console-discovery.png)
**Note**  
If prompted, grant permission to create the [Greengrass service role](service-role.md) and associate it with your AWS account in the current AWS Region\. This role allows AWS IoT Greengrass to access your resources in AWS services\.

   The **Deployments** page shows the deployment timestamp, version ID, and status\. When completed, the status displayed for the deployment should be **Successfully completed**\.

   For troubleshooting help, see [Troubleshooting AWS IoT Greengrass](gg-troubleshooting.md)\.

## Test the Lambda function<a name="secrets-console-test-solution"></a>

1. <a name="choose-test-page"></a>On the AWS IoT console home page, choose **Test**\.  
![\[The left pane in the AWS IoT console with Test highlighted.\]](http://docs.aws.amazon.com/greengrass/latest/developerguide/images/console-test.png)

1. For **Subscriptions**, use the following values, and then choose **Subscribe to topic**\.    
[\[See the AWS documentation website for more details\]](http://docs.aws.amazon.com/greengrass/latest/developerguide/secrets-console.html)

1. For **Publish**, use the following values, and then choose **Publish to topic** to invoke the function\.    
[\[See the AWS documentation website for more details\]](http://docs.aws.amazon.com/greengrass/latest/developerguide/secrets-console.html)

   If successful, the function publishes a "Success" message\.

## See also<a name="secrets-console-see-also"></a>
+ [Deploy secrets to the AWS IoT Greengrass core](secrets.md)