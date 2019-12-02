# How to Configure Machine Learning Inference Using the AWS Management Console<a name="ml-console"></a>

To follow the steps in this tutorial, you must be using AWS IoT Greengrass Core v1\.6 or later\.

You can perform machine learning \(ML\) inference locally on a Greengrass core device using data from connected devices\. For information, including requirements and constraints, see [Perform Machine Learning Inference](ml-inference.md)\.

This tutorial describes how to use the AWS Management Console to configure a Greengrass group to run a Lambda inference app that recognizes images from a camera locally, without sending data to the cloud\. The inference app accesses the camera module on a Raspberry Pi and runs inference using the open source [SqueezeNet](https://github.com/DeepScale/SqueezeNet) model\.

The tutorial contains the following high\-level steps:

1. [Configure the Raspberry Pi](#config-raspberry-pi)

1. [Install the MXNet Framework](#install-mxnet)

1. [Create a Model Package](#package-ml-model)

1. [Create and Publish a Lambda Function](#ml-console-create-lambda)

1. [Add the Lambda Function to the Group](#ml-console-config-lambda)

1. [Add Resources to the Group](#ml-console-add-resources)

1. [Add a Subscription to the Group](#ml-console-add-subscription)

1. [Deploy the Group](#ml-console-deploy-group)

1. [Test the App](#ml-console-test-app)

## Prerequisites<a name="ml-inference-prerequisites"></a>

To complete this tutorial, you need:
+ Raspberry Pi 4 Model B, or Raspberry Pi 3 Model B/B\+, set up and configured for use with AWS IoT Greengrass\. To learn how to set up your Raspberry Pi with AWS IoT Greengrass, see [Module 1](https://docs.aws.amazon.com/greengrass/latest/developerguide/module1.html) and [Module 2](https://docs.aws.amazon.com/greengrass/latest/developerguide/module2.html) of [Getting Started with AWS IoT Greengrass](gg-gs.md)\.
**Note**  
The Raspberry Pi might require a 2\.5A [power supply](https://www.raspberrypi.org/documentation/hardware/raspberrypi/power/) to run the deep learning frameworks that are typically used for image classification\. A power supply with a lower rating might cause the device to reboot\.
+ [Raspberry Pi Camera Module V2 \- 8 Megapixel, 1080p](https://www.amazon.com/Raspberry-Pi-Camera-Module-Megapixel/dp/B01ER2SKFS)\. To learn how to set up the camera, see [Connecting the camera](https://www.raspberrypi.org/documentation/usage/camera/) in the Raspberry Pi documentation\. 
+ A Greengrass group and a Greengrass core\. To learn how to create a Greengrass group or core, see [Getting Started with AWS IoT Greengrass](gg-gs.md)\.

**Note**  
This tutorial uses a Raspberry Pi, but AWS IoT Greengrass supports other platforms, such as Intel Atom and [NVIDIA Jetson TX2](#jetson-lambda-config)\. The example for Jetson TX2 can use static images instead of images streamed from a camera\.

## Step 1: Configure the Raspberry Pi<a name="config-raspberry-pi"></a>

In this step, you install updates to the Rasbian operating system, install the camera module software and Python dependencies, and enable the camera interface\. Run the following commands in your Raspberry Pi terminal\.

1. Install updates to Raspbian\.

   ```
   sudo apt-get update
   sudo apt-get dist-upgrade
   ```

1. Install the picamera interface for the camera module and other Python libraries that are required for this tutorial\.

   ```
   sudo apt-get install -y python-dev python-setuptools python-pip python-picamera
   ```

1. Reboot the Raspberry Pi\.

   ```
   sudo reboot
   ```

1. Open the Raspberry Pi configuration tool\.

   ```
   sudo raspi-config
   ```

1. Use the arrow keys to open **Interfacing Options** and enable the camera interface\. If prompted, allow the device to reboot\.

1. Use the following command to test the camera setup\.

   ```
   raspistill -v -o test.jpg
   ```

   This opens a preview window on the Raspberry Pi, saves a picture named `test.jpg` to your `/home/pi` directory, and displays information about the camera in the Raspberry Pi terminal\.

## Step 2: Install the MXNet Framework<a name="install-mxnet"></a>

In this step, you download precompiled Apache MXNet libraries and install them on your Raspberry Pi\.

**Note**  
This tutorial uses libraries for the MXNet ML framework, but libraries for TensorFlow are also available\. For more information, including limitations, see [Runtimes and Precompiled Framework Libraries for ML Inference](ml-inference.md#precompiled-ml-libraries)\.

1. On the [AWS IoT Greengrass Machine Learning Runtimes and Precompiled Libraries](what-is-gg.md#gg-ml-runtimes-pc-libs) downloads page, locate MXNet version 1\.2\.1 for Raspberry Pi\. Choose **Download**\.
**Note**  
By downloading this software you agree to the [Apache License 2\.0](https://www.apache.org/licenses/LICENSE-2.0)\.

1. Transfer the downloaded `ggc-mxnet-v1.2.1-python-raspi.tar.gz` file from your computer to your Raspberry Pi\.
**Note**  
For ways that you can do this on different platforms, see [this step](gg-device-start.md) in the Getting Started section\. For example, you might use the following scp command:  

   ```
   scp ggc-mxnet-v1.2.1-python-raspi.tar.gz pi@IP-address:/home/pi
   ```

1. In your Raspberry Pi terminal, unpack the transferred file\.

   ```
   tar -xzf ggc-mxnet-v1.2.1-python-raspi.tar.gz
   ```

1. Install the MXNet framework\.

   ```
   cd ggc-mxnet-v1.2.1-python-raspi/
   ./mxnet_installer.sh
   ```
**Note**  
You can continue to [Step 3: Create an MXNet Model Package](#package-ml-model) while the framework is being installed, but you must wait until the installation is complete before you proceed to [Step 4: Create and Publish a Lambda Function](#ml-console-create-lambda)\.  
You can optionally run unit tests to verify the installation\. To do so, add the `-u` option to the previous command\. If successful, each test logs a line in the terminal that ends with `ok`\. If all tests are successful, the final log statement contains `OK`\. Running unit tests increases the installation time\.

   The script also creates a Lambda function deployment package named `greengrassObjectClassification.zip` that contains the function code and dependencies\. You upload this deployment package later\.

1. When the installation is complete, transfer `greengrassObjectClassification.zip` to your computer\. Depending on your environment, you can use the scp command or a utility such as [WinSCP](https://winscp.net/eng/download.php)\.

## Step 3: Create an MXNet Model Package<a name="package-ml-model"></a>

In this step, you download files for a sample pretrained MXNet model, and then save them as a `.zip` file\. AWS IoT Greengrass can use models from Amazon S3, provided that they use the `tar.gz` or `.zip` format\.

1. Download the following files to your computer:
   + [squeezenet\_v1\.1\-0000\.params](https://s3.amazonaws.com/model-server/model_archive_1.0/examples/squeezenet_v1.1/squeezenet_v1.1-0000.params)\. A parameter file that describes weights of the connectivity\.
   + [squeezenet\_v1\.1\-symbol\.json](https://s3.amazonaws.com/model-server/model_archive_1.0/examples/squeezenet_v1.1/squeezenet_v1.1-symbol.json)\. A symbol file that describes the neural network structure\.
   + [synset\.txt](https://s3.amazonaws.com/model-server/model_archive_1.0/examples/squeezenet_v1.1/synset.txt)\. A synset file that maps recognized class IDs to human\-readable class names\.
**Note**  
All MXNet model packages use these three file types, but the contents of TensorFlow model packages vary\.

1. Zip the three files, and name the compressed file `squeezenet.zip`\. You upload this model package to Amazon S3 in [Step 6: Add Resources to the Greengrass Group](#ml-console-add-resources)\.

## Step 4: Create and Publish a Lambda Function<a name="ml-console-create-lambda"></a>

In this step, you create a Lambda function and configure it to use the deployment package\. Then, you publish a function version and create an alias\.

The Lambda function deployment package is named `greengrassObjectClassification.zip`\. This is the `zip` file that was generated during the MXNet framework installation in [Step 2: Install the MXNet Framework](#install-mxnet)\. It contains an inference app that performs common tasks, such as loading models, importing Apache MXNet, and taking actions based on predictions\. The app contains the following key components:
+ App logic:
  + **load\_model\.py**\. Loads MXNet models\.
  + **greengrassObjectClassification\.py**\. Runs predictions on images that are streamed from the camera\.
+ Dependencies:
  + **greengrasssdk**\. The AWS IoT Greengrass Core SDK for Python, used by the function to publish MQTT messages\.
**Note**  
The `mxnet` library was installed on the core device during the MXNet framework installation\.

First, create the Lambda function\.

1. In the AWS IoT console, in the navigation pane, choose **Greengrass**, and then choose **Groups**\.  
![\[The navigation pane in the AWS IoT console with Groups highlighted.\]](http://docs.aws.amazon.com/greengrass/latest/developerguide/images/console-groups.png)

1. Choose the Greengrass group where you want to add the Lambda function\.

1. On the group configuration page, choose **Lambdas**, and then choose **Add Lambda**\.  
![\[The group page with Lambdas and Add Lambda highlighted.\]](http://docs.aws.amazon.com/greengrass/latest/developerguide/images/console-group-lambdas.png)

1. On the **Add a Lambda to your Greengrass Group** page, choose **Create new Lambda**\. This opens the AWS Lambda console\.  
![\[The Add a Lambda to your Greengrass Group page with Create new Lambda highlighted.\]](http://docs.aws.amazon.com/greengrass/latest/developerguide/images/console-group-lambdas-new-lambda.png)

1. Choose **Author from scratch** and use the following values to create your function:
   + For **Function name**, enter **greengrassObjectClassification**\.
   + For **Runtime**, choose **Python 3\.7**\.

   For **Permissions**, keep the default setting\. This creates an execution role that grants basic Lambda permissions\. This role isn't used by AWS IoT Greengrass\.

1. Choose **Create function**\.  
![\[The Create function page with Create function highlighted.\]](http://docs.aws.amazon.com/greengrass/latest/developerguide/images/ml-inference/create-function.png)

    

   Now, upload your Lambda function deployment package and register the handler\.

1. On the **Configuration** tab for the `greengrassObjectClassification` function, for **Function code**, use the following values:
   + For **Code entry type**, choose **Upload a \.zip file**\. 
   + For **Runtime**, choose **Python 3\.7**\.
   + For **Handler**, enter **greengrassObjectClassification\.function\_handler**\.

1. Choose **Upload**\.  
![\[The Function code section with Upload highlighted.\]](http://docs.aws.amazon.com/greengrass/latest/developerguide/images/ml-inference/upload-deployment-package.png)

1. Choose your `greengrassObjectClassification.zip` deployment package\.

1. Choose **Save**\.

    

   Next, publish the first version of your Lambda function\. Then, create an [alias for the version](https://docs.aws.amazon.com/lambda/latest/dg/versioning-aliases.html)\.
**Note**  
Greengrass groups can reference a Lambda function by alias \(recommended\) or by version\. Using an alias makes it easier to manage code updates because you don't have to change your subscription table or group definition when the function code is updated\. Instead, you just point the alias to the new function version\.

1. From the **Actions** menu, choose **Publish new version**\.  
![\[The Publish new version option in the Actions menu.\]](http://docs.aws.amazon.com/greengrass/latest/developerguide/images/ml-inference/lambda-publish-version.png)

1. For **Version description**, enter **First version**, and then choose **Publish**\.

1. On the **greengrassObjectClassification: 1** configuration page, from the **Actions** menu, choose **Create alias**\.  
![\[The Create alias option in the Actions menu.\]](http://docs.aws.amazon.com/greengrass/latest/developerguide/images/ml-inference/lambda-create-alias.png)

1. On the **Create a new alias** page, use the following values:
   + For **Name**, enter **mlTest**\.
   + For **Version**, enter **1**\.
**Note**  
AWS IoT Greengrass doesn't support Lambda aliases for **$LATEST** versions\.

1. Choose **Create**\.  
![\[The Create a new alias page with Create highlighted.\]](http://docs.aws.amazon.com/greengrass/latest/developerguide/images/ml-inference/lambda-create-alias-create.png)

   Now, add the Lambda function to your Greengrass group\.

## Step 5: Add the Lambda Function to the Greengrass Group<a name="ml-console-config-lambda"></a>

In this step, you add the Lambda function to the group and then configure its lifecycle and environment variables\.

First, add the Lambda function to your Greengrass group\.

1. In the AWS IoT console, open the group configuration page\.

1. Choose **Lambdas**, and then choose **Add Lambda**\.  
![\[The group page with Lambdas and Add Lambda highlighted.\]](http://docs.aws.amazon.com/greengrass/latest/developerguide/images/console-group-lambdas.png)

1. On the **Add a Lambda to your Greengrass Group** page, choose **Use existing Lambda**\.  
![\[The Add a Lambda to your Greengrass Group page with Use existing Lambda highlighted.\]](http://docs.aws.amazon.com/greengrass/latest/developerguide/images/console-group-lambdas-existing-lambda.png)

1. Choose **greengrassObjectClassification**, and then choose **Next**\.

1. On the **Select a Lambda version** page, choose **Alias:mlTest**, and then choose **Finish**\.

    

   Next, configure the lifecycle and environment variables of the Lambda function\.

1. On the **Lambdas** page, choose the **greengrassObjectClassification** Lambda function\.  
![\[The Lambdas page with the greengrassObjectClassification Lambda function highlighted.\]](http://docs.aws.amazon.com/greengrass/latest/developerguide/images/ml-inference/lambda.png)

1. On the **greengrassObjectClassification** configuration page, choose **Edit**\.

1. On the **Group\-specific Lambda configuration** page, use the following values:
   + For **Memory limit**, enter **96 MB**\.
   + For **Timeout**, enter **10 seconds**\.
   + For **Lambda lifecycle**, choose **Make this function long\-lived and keep it running indefinitely**\. 
   + For **Read access to /sys directory**, choose **Enable**\.

   For more information, see [Lifecycle Configuration for Greengrass Lambda Functions](lambda-functions.md#lambda-lifecycle)\.  
![\[The greengrassObjectClassification page with updated properties.\]](http://docs.aws.amazon.com/greengrass/latest/developerguide/images/ml-inference/lambda-config.png)

1. Under **Environment variables**, create a key\-value pair\. A key\-value pair is required by functions that interact with MXNet models on a Raspberry Pi\.

   For the key, use MXNET\_ENGINE\_TYPE\. For the value, use NaiveEngine\. 
**Note**  
In your own user\-defined Lambda functions, you can optionally set the environment variable in your function code\.

1. Keep the default values for all other properties and choose **Update**\.

## Step 6: Add Resources to the Greengrass Group<a name="ml-console-add-resources"></a>

In this step, you create resources for the camera module and the ML inference model\. You also affiliate the resources with the Lambda function, which makes it possible for the function to access the resources on the core device\.

First, create two local device resources for the camera: one for shared memory and one for the device interface\. For more information about local resource access, see [Access Local Resources with Lambda Functions and Connectors](access-local-resources.md)\.

1. On the group configuration page, choose **Resources**\.  
![\[The group configuration page with Resources highlighted.\]](http://docs.aws.amazon.com/greengrass/latest/developerguide/images/console-group-resources.png)

1. On the **Local** tab, choose **Add a local resource**\.

1. On the **Create a local resource** page, use the following values:
   + For **Resource name**, enter **videoCoreSharedMemory**\.
   + For **Resource type**, choose **Device**\.
   + For **Device path**, enter **/dev/vcsm**\.

     The device path is the local absolute path of the device resource\. This path can only refer to a character device or block device under `/dev`\.
   + For **Group owner file access permission**, choose **Automatically add OS group permissions of the Linux group that owns the resource**\.

     The **Group owner file access permission** option lets you grant additional file access permissions to the Lambda process\. For more information, see [Group Owner File Access Permission](access-local-resources.md#lra-group-owner)\.  
![\[The Create a local resource page with edited resource properties.\]](http://docs.aws.amazon.com/greengrass/latest/developerguide/images/ml-inference/local-resource-vcsm.png)

1. Under **Lambda function affiliations**, choose **Select**\.

1. Choose **greengrassObjectClassification**, choose **Read and write access**, and then choose **Done**\.  
![\[Lambda function affiliation properties with Done highlighted.\]](http://docs.aws.amazon.com/greengrass/latest/developerguide/images/ml-inference/local-resource-vcsm-lambda.png)

   Next, you add a local device resource for the camera interface\.

1. Choose **Add another resource**\.

1. On the **Create a local resource** page, use the following values:
   + For **Resource name**, enter **videoCoreInterface**\.
   + For **Resource type**, choose **Device**\.
   + For **Device path**, enter **/dev/vchiq**\.
   + For **Group owner file access permission**, choose **Automatically add OS group permissions of the Linux group that owns the resource**\.   
![\[The Create a local resource page with edited resource properties.\]](http://docs.aws.amazon.com/greengrass/latest/developerguide/images/ml-inference/local-resource-vchiq.png)

1. Under **Lambda function affiliations**, choose **Select**\.

1. Choose **greengrassObjectClassification**, choose **Read and write access**, and then choose **Done**\.

1. At the bottom of the page, choose **Save**\.

 

Now, add the inference model as a machine learning resource\. This step includes uploading the `squeezenet.zip` model package to Amazon S3\.

1. On the **Resources** page for your group, choose **Machine Learning**, and then choose **Add a machine learning resource**\.

1. On the **Create a machine learning resource** page, for **Resource name**, enter **squeezenet\_model**\.  
![\[The Add Machine Learning Model page with updated properties.\]](http://docs.aws.amazon.com/greengrass/latest/developerguide/images/ml-inference/ml-resource-squeezenet.png)

1. For **Model source**, choose **Upload a model in S3**\.

1.  Under **Model from S3**, choose **Select**\. 

1.  Choose **Upload a model**\. This opens up a new tab to the Amazon S3 console\. 

1.  In the Amazon S3 console tab, upload the `squeezenet.zip` file to an Amazon S3 bucket\. For information, see [ How Do I Upload Files and Folders to an S3 Bucket? ](https://docs.aws.amazon.com/AmazonS3/latest/user-guide/upload-objects.html) in the *Amazon Simple Storage Service Console User Guide*\. 
**Note**  
For the bucket to be accessible, your bucket name must contain the string **greengrass**\. Choose a unique name \(such as **greengrass\-bucket\-*user\-id*\-*epoch\-time***\)\. Don't use a period \(`.`\) in the bucket name\. 

1.  In the AWS IoT Greengrass console tab, locate and choose your Amazon S3 bucket\. Locate your uploaded `squeezenet.zip` file, and choose **Select**\. You might need to choose **Refresh** to update the list of available buckets and files\. 

1. For **Local path**, enter **/greengrass\-machine\-learning/mxnet/squeezenet**\.

   This is the destination for the local model in the Lambda runtime namespace\. When you deploy the group, AWS IoT Greengrass retrieves the source model package and then extracts the contents to the specified directory\. The sample Lambda function for this tutorial is already configured to use this path \(in the `model_path` variable\)\.

1. Under **Identify resource owner and set access permissions**, choose **No OS group**\.

1. Under **Lambda function affiliations**, choose **Select**\.

1. Choose **greengrassObjectClassification**, choose **Read\-only access**, and then choose **Done**\.

1. Choose **Save**\.

### Using Amazon SageMaker Trained Models<a name="sm-models"></a>

This tutorial uses a model that's stored in Amazon S3, but you can easily use Amazon SageMaker models too\. The AWS IoT Greengrass console has built\-in Amazon SageMaker integration, so you don't need to manually upload these models to Amazon S3\. For requirements and limitations for using Amazon SageMaker models, see [Supported Model Sources](ml-inference.md#supported-model-sources)\.

To use an Amazon SageMaker model:
+ For **Model source**, choose **Use an existing SageMaker model**, and then choose the name of the model's training job\.
+ For **Local path**, enter the path to the directory where your Lambda function looks for the model\.

## Step 7: Add a Subscription to the Greengrass Group<a name="ml-console-add-subscription"></a>

In this step, you add a subscription to the group\. This subscription enables the Lambda function to send prediction results to AWS IoT by publishing to an MQTT topic\.

1. On the group configuration page, choose **Subscriptions**, and then choose **Add Subscription**\.  
![\[The group page with Subscriptions and Add Subscription highlighted.\]](http://docs.aws.amazon.com/greengrass/latest/developerguide/images/console-group-subscriptions.png)

1. On the **Select your source and target** page, configure the source and target, as follows:

   1. In **Select a source**, choose **Lambdas**, and then choose **greengrassObjectClassification**\.

   1. In **Select a target**, choose **Services**, and then choose **IoT Cloud**\.

   1. Choose **Next**\.  
![\[The Select your source and target page with Next highlighted.\]](http://docs.aws.amazon.com/greengrass/latest/developerguide/images/ml-inference/add-subscription.png)

1. On the **Filter your data with a topic** page, in **Topic filter**, enter **hello/world**, and then choose **Next**\.  
![\[The Filter your data with a topic page with Next highlighted.\]](http://docs.aws.amazon.com/greengrass/latest/developerguide/images/ml-inference/add-subscription-topic.png)

1. Choose **Finish**\.

## Step 8: Deploy the Greengrass Group<a name="ml-console-deploy-group"></a>

In this step, you deploy the current version of the group definition to the Greengrass core device\. The definition contains the Lambda function, resources, and subscription configurations that you added\.

1. Make sure that the AWS IoT Greengrass core is running\. Run the following commands in your Raspberry Pi terminal, as needed\.

   1. To check whether the daemon is running:

      ```
      ps aux | grep -E 'greengrass.*daemon'
      ```

      If the output contains a `root` entry for `/greengrass/ggc/packages/1.10.0/bin/daemon`, then the daemon is running\.
**Note**  
The version in the path depends on the AWS IoT Greengrass Core software version that's installed on your core device\.

   1. To start the daemon:

      ```
      cd /greengrass/ggc/core/
      sudo ./greengrassd start
      ```

1. On the group configuration page, choose **Deployments**, and from the **Actions** menu, choose **Deploy**\.  
![\[The group page with Deployments and Deploy highlighted.\]](http://docs.aws.amazon.com/greengrass/latest/developerguide/images/console-group-deployments-deploy.png)

1. On the **Configure how devices discover your core** page, choose **Automatic detection**\.

   This enables devices to automatically acquire connectivity information for the core, such as IP address, DNS, and port number\. Automatic detection is recommended, but AWS IoT Greengrass also supports manually specified endpoints\. You're only prompted for the discovery method the first time that the group is deployed\.  
![\[The Configure how devices discover your core page with Automatic detection highlighted.\]](http://docs.aws.amazon.com/greengrass/latest/developerguide/images/console-discovery.png)
**Note**  
If prompted, grant permission to create the [Greengrass service role](service-role.md) and associate it with your AWS account in the current AWS Region\. This role allows AWS IoT Greengrass to access your resources in AWS services\.

   The **Deployments** page shows the deployment timestamp, version ID, and status\. When completed, the status displayed for the deployment should be **Successfully completed**\.

   For troubleshooting help, see [Troubleshooting AWS IoT Greengrass](gg-troubleshooting.md)\.

## Step 9: Test the Inference App<a name="ml-console-test-app"></a>

Now you can verify whether the deployment is configured correctly\. To test, you subscribe to the `hello/world` topic and view the prediction results that are published by the Lambda function\.

**Note**  
If a monitor is attached to the Raspberry Pi, the live camera feed is displayed in a preview window\.

1. In the AWS IoT console, choose **Test**\.  
![\[The navigation pane in the AWS IoT console with Test highlighted.\]](http://docs.aws.amazon.com/greengrass/latest/developerguide/images/console-test.png)

1. For **Subscriptions**, use the following values:
   + For the subscription topic, use hello/world\.
   + For **MQTT payload display**,choose **Display payloads as strings**\.

1. Choose **Subscribe to topic**\.

   If the test is successful, the messages from the Lambda function appear at the bottom of the page\. Each message contains the top five prediction results of the image, using the format: probability, predicted class ID, and corresponding class name\.  
![\[The Subscriptions page showing test results with message data.\]](http://docs.aws.amazon.com/greengrass/latest/developerguide/images/ml-inference/prediction-results.png)

### Troubleshooting AWS IoT Greengrass ML Inference<a name="ml-inference-troubleshooting"></a>

If the test is not successful, you can try the following troubleshooting steps\. Run the commands in your Raspberry Pi terminal\.

#### Check Error Logs<a name="troubleshooting-check-logs"></a>

1. <a name="root-access-logs"></a>Switch to the root user and navigate to the `log` directory\. Access to AWS IoT Greengrass logs requires root permissions\.

   ```
   sudo su
   cd /greengrass/ggc/var/log
   ```

1. In the `system` directory, check `runtime.log` or `python_runtime.log`\.

   In the `user/region/account-id` directory, check `greengrassObjectClassification.log`\.

   For more information, see [Troubleshooting with Logs](gg-troubleshooting.md#troubleshooting-logs)\.

##### Unpacking Error in Runtime\.log<a name="troubleshooting-targz-unpacking"></a>

If `runtime.log` contains an error similar to the following, make sure that your `tar.gz` source model package has a parent directory\.

```
Greengrass deployment error: unable to download the artifact model-arn: Error while processing. 
Error while unpacking the file from /tmp/greengrass/artifacts/model-arn/path to /greengrass/ggc/deployment/path/model-arn,
error: open /greengrass/ggc/deployment/path/model-arn/squeezenet/squeezenet_v1.1-0000.params: no such file or directory
```

If your package doesn't have a parent directory that contains the model files, use the following command to repackage the model:

```
tar -zcvf model.tar.gz ./model
```

For example:

```
─$ tar -zcvf test.tar.gz ./test
./test
./test/some.file
./test/some.file2
./test/some.file3
```

**Note**  
Don't include trailing `/*` characters in this command\.

 

#### Verify That the Lambda Function Is Successfully Deployed<a name="troubleshooting-check-lambda"></a>

1. List the contents of the deployed Lambda in the `/lambda` directory\. Replace the placeholder values before you run the command\.

   ```
   cd /greengrass/ggc/deployment/lambda/arn:aws:lambda:region:account:function:function-name:function-version
   ls -la
   ```

1. Verify that the directory contains the same content as the `greengrassObjectClassification.zip` deployment package that you uploaded in [Step 4: Create and Publish a Lambda Function](#ml-console-create-lambda)\.

   Make sure that the `.py` files and dependencies are in the root of the directory\.

 

#### Verify That the Inference Model Is Successfully Deployed<a name="troubleshooting-check-model"></a>

1. Find the process identification number \(PID\) of the Lambda runtime process:

   ```
   ps aux | grep 'lambda-function-name*'
   ```

   In the output, the PID appears in the second column of the line for the Lambda runtime process\.

1. Enter the Lambda runtime namespace\. Be sure to replace the placeholder *pid* value before you run the command\.
**Note**  
This directory and its contents are in the Lambda runtime namespace, so they aren't visible in a regular Linux namespace\.

   ```
   sudo nsenter -t pid -m /bin/bash
   ```

1. List the contents of the local directory that you specified for the ML resource\.

   ```
   cd /greengrass-machine-learning/mxnet/squeezenet/
   ls -ls
   ```

   You should see the following files:

   ```
   32 -rw-r--r-- 1 ggc_user ggc_group   31675 Nov 18 15:19 synset.txt
   32 -rw-r--r-- 1 ggc_user ggc_group   28707 Nov 18 15:19 squeezenet_v1.1-symbol.json
   4832 -rw-r--r-- 1 ggc_user ggc_group 4945062 Nov 18 15:19 squeezenet_v1.1-0000.params
   ```

## Next Steps<a name="next-steps"></a>

Next, explore other inference apps\. AWS IoT Greengrass provides other Lambda functions that you can use to try out local inference\. You can find the examples package in the precompiled libraries folder that you downloaded in [Step 2: Install the MXNet Framework](#install-mxnet)\.

## Configuring an NVIDIA Jetson TX2<a name="jetson-lambda-config"></a>

To run this tutorial on an NVIDIA Jetson TX2, you provide source images and configure the Lambda function\. If you're using the GPU, you must also add local device resources\.

To learn how to configure your Jetson so you can install the AWS IoT Greengrass Core software, see [Setting Up Other Devices](setup-filter.other.md)\.

1. Download static PNG or JPG images for the Lambda function to use for image classification\. The app works best with small image files\. Alternatively, you can instrument a camera on the Jetson board to capture the source images\.

   Save your image files in the directory that contains the `greengrassObjectClassification.py` file \(or in a subdirectory of this directory\)\. This is in the Lambda function deployment package that you upload in [Step 4: Create and Publish a Lambda Function](#ml-console-create-lambda)\.

1. Edit the configuration of the Lambda function to increase the **Memory limit** value\. Use 500 MB for CPU, or 2048 MB for GPU\. Follow the procedure in [Step 5: Add the Lambda Function to the Greengrass Group](#ml-console-config-lambda)\.

1. **GPU only**: Add the following local device resources\. Follow the procedure in [Step 6: Add Resources to the Greengrass Group](#ml-console-add-resources)\.

   For each resource:
   + For **Resource type**, choose **Device**\.
   + For **Group owner file access permission**, choose **Automatically add OS group permissions of the Linux group that owns the resource**\.
   + For **Lambda function affiliations**, grant **Read and write access** to your Lambda function\.

          
[\[See the AWS documentation website for more details\]](http://docs.aws.amazon.com/greengrass/latest/developerguide/ml-console.html)