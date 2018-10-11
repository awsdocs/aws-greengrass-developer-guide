# How to Configure Machine Learning Inference Using the AWS Management Console<a name="ml-console"></a>

To follow this tutorial, you must be using AWS Greengrass Core v1\.6\.0\.

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

## Prerequisites<a name="ml-inference-prerequisites"></a>

To complete this tutorial, you need:
+ [Raspberry Pi 3 Model B](https://www.raspberrypi.org/products/raspberry-pi-3-model-b/#buy-now-modal)\. 
+ [Raspberry Pi Camera Module V2 \- 8 Megapixel, 1080p](https://www.amazon.com/Raspberry-Pi-Camera-Module-Megapixel/dp/B01ER2SKFS)\. To learn how to set up the camera, see [ Connecting the camera](https://www.raspberrypi.org/documentation/usage/camera/) in the Raspberry Pi documentation\. 
+ A Greengrass group and a Greengrass core\. To learn how to create a Greengrass group or core, see [Getting Started with AWS Greengrass](gg-gs.md)\. The Getting Started section also includes steps for installing the AWS Greengrass Core software on a Raspberry Pi\.

**Note**  
This tutorial uses a Raspberry Pi, but AWS Greengrass supports other platforms, such as Intel Atom and [NVIDIA Jetson TX2](#jetson-lambda-config)\.

## Step 1: Configure the Raspberry Pi<a name="config-raspberry-pi"></a>

In this step, you update the Rasbian operating system, install the camera module software and Python dependencies, and enable the camera interface\. Run the following commands in your Raspberry Pi terminal\.

1. Update Raspbian\.

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

   This opens a preview window on the Raspberry Pi, saves a picture named `test.jpg` to your /home/pi directory, and displays information about the camera in the Raspberry Pi terminal\.

## Step 2: Install the MXNet Framework<a name="install-mxnet"></a>

In this step, you download precompiled Apache MXNet libraries and install them on your Raspberry Pi\.

**Note**  
This tutorial uses libraries for the MXNet ML framework, but libraries for TensorFlow are also available\. For more information, including limitations, see [Precompiled Libraries for ML Frameworks](ml-inference.md#precompiled-ml-libraries)\.

1. On your computer, open the [AWS IoT console](https://console.aws.amazon.com/iotv2)\.

1. In the left pane, choose **Software**\.  
![\[The AWS IoT console with Software highlighted.\]](http://docs.aws.amazon.com/greengrass/latest/developerguide/images/console-software.png)

1. In the **Machine learning libraries** section, for ** MXNet/TensorFlow precompiled libraries**, choose **Configure download**\.

1. On the **Machine learning libraries** page, under **Software configurations**, for MXNet Raspberry Pi version 1\.2\.1, choose **Download**\.
**Note**  
By downloading this software you agree to the [Apache License 2\.0](https://www.apache.org/licenses/LICENSE-2.0)\.

1. Transfer the downloaded `ggc-mxnet-v1.2.1-python-raspi.tar.gz` file from your computer to your Raspberry Pi\.
**Note**  
For ways that you can do this on different platforms, see [this step](gg-device-start.md) in the Getting Started section\. For example, you might use the following `scp` command:  

   ```
   scp ggc-mxnet-v1.2.1-python-raspi.tar.gz pi@IP-address:/home/pi
   ```

1. In your Raspberry Pi terminal, unpack the transferred file\.

   ```
   tar -xzf ggc-mxnet-v1.2.1-python-raspi.tar.gz
   ```

1. Install the MDXNet framework\.

   ```
   ./mxnet_installer.sh
   ```
**Note**  
You can continue to [Step 3: Create an MXNet Model Package](#package-ml-model) while the framework is installing, but you must wait for the installation to complete before proceeding to [Step 4: Create and Publish a Lambda Function](#ml-console-create-lambda)\.  
You can optionally run unit tests to verify the installation\. To do so, add the `-u` option to the previous command\. If successful, each test logs a line in the terminal that ends with `ok`\. If all tests are successful, the final log statement contains `OK`\. Note that running the unit tests increases the installation time\.

   The script also creates a Lambda function deployment package named `greengrassObjectClassification.zip`\. This package contains the function code and dependencies, including the `mxnet` Python module that Greengrass Lambda functions need to work with MXNet models\. You upload this deployment package later\.

1. When the installation is complete, transfer `greengrassObjectClassification.zip` to your computer\. Depending on your environment, you can use the `scp` command or a utility such as [WinSCP](https://winscp.net/eng/download.php)\.

## Step 3: Create an MXNet Model Package<a name="package-ml-model"></a>

In this step, you download files for a sample pretrained MXNet model, and then save them as a `.zip` file\. AWS Greengrass can use models from Amazon S3, provided that they use the `tar.gz` or `.zip` format\.

1. Download the following files to your computer:
   + [ squeezenet\_v1\.1\-0000\.params](http://data.dmlc.ml/mxnet/models/imagenet/squeezenet/squeezenet_v1.1-0000.params)\. A parameter file that describes weights of the connectivity\.
   + [ squeezenet\_v1\.1\-symbol\.json](http://data.dmlc.ml/mxnet/models/imagenet/squeezenet/squeezenet_v1.1-symbol.json)\. A symbol file that describes the neural network structure\.
   + [synset\.txt](http://data.dmlc.ml/mxnet/models/imagenet/synset.txt)\. A synset file that maps recognized class IDs to human\-readable class names\.
**Note**  
All MXNet model packages use these three file types, but the contents of TensorFlow model packages vary\.

1. Zip the three files, and name the compressed file `squeezenet.zip`\. You upload this model package to Amazon S3 in [Step 6: Add Resources to the Greengrass Group](#ml-console-add-resources)\.

## Step 4: Create and Publish a Lambda Function<a name="ml-console-create-lambda"></a>

In this step, you create a Lambda function and configure it to use the deployment package that was created in [Step 2: Install the MXNet Framework](#install-mxnet)\. Then, you publish a function version and create an alias\.

The Lambda function deployment package is named `greengrassObjectClassification.zip`\. It contains an inference app that performs common tasks, such as loading models, importing Apache MXNet, and taking actions based on predictions\. The app contains the following key components:
+ App logic:
  + **load\_model\.py**\. Loads MXNet models\.
  + **greengrassObjectClassification\.py**\. Runs predictions on images that are streamed from the camera\.
+ Dependencies:
  + **greengrasssdk**\. Required library for all Python Lambda functions\.
  + **mxnet**\. Required library for Python Lambda functions that run local inference using MXNet\.
+ License:
  + **license**\. Contains the required Greengrass Core Software License Agreement\.

**Note**  
You can reuse these dependencies and license when you create new MXNet inference Lambda functions\.

First, create the Lambda function\.

1. In the AWS IoT console, in the left pane, choose **Greengrass**, and then choose **Groups**\.  
![\[The left pane in the AWS IoT console with Groups highlighted.\]](http://docs.aws.amazon.com/greengrass/latest/developerguide/images/console-groups.png)

1. Choose the Greengrass group where you want to add the Lambda function\.

1. On the group configuration page, choose **Lambdas**, and then choose **Add Lambda**\.  
![\[The group page with Lambdas and Add Lambda highlighted.\]](http://docs.aws.amazon.com/greengrass/latest/developerguide/images/console-group-lambdas.png)

1. On the **Add a Lambda to your Greengrass Group** page, choose **Create new Lambda**\. This takes you to the AWS Lambda console\.  
![\[The Add a Lambda to your Greengrass Group page with Create new Lambda highlighted.\]](http://docs.aws.amazon.com/greengrass/latest/developerguide/images/console-group-lambdas-new-lambda.png)

1. Choose **Author from scratch**\.

1. In the **Author from scratch** section, use the following values:    
[\[See the AWS documentation website for more details\]](http://docs.aws.amazon.com/greengrass/latest/developerguide/ml-console.html)

1. At the bottom of the page, choose **Create function**\.  
![\[The Create function page with Create function highlighted.\]](http://docs.aws.amazon.com/greengrass/latest/developerguide/images/ml-inference/create-function.png)

 

Now, upload your Lambda function deployment package and register the handler\.

1. On the **Configuration** tab for the **greengrassObjectClassification** function, use the following values for **Function code**:    
[\[See the AWS documentation website for more details\]](http://docs.aws.amazon.com/greengrass/latest/developerguide/ml-console.html)

1. Choose **Upload**\.  
![\[The Function code section with Upload highlighted.\]](http://docs.aws.amazon.com/greengrass/latest/developerguide/images/ml-inference/upload-deployment-package.png)

1. Choose your `greengrassObjectClassification.zip` deployment package\.

1. At the top of the page, choose **Save**\.

 

Next, publish the first version of your Lambda function\. Then, create an [alias for the version](https://docs.aws.amazon.com/lambda/latest/dg/versioning-aliases.html)\.

**Note**  
Greengrass groups can reference a Lambda function by alias \(recommended\) or by version\. Using an alias makes it easier to manage code updates because you don't have to change your subscription table or group definition when the function code is updated\. Instead, you just point the alias to the new function version\.

1. From the **Actions** menu, choose **Publish new version**\.  
![\[The Publish new version option in the Actions menu.\]](http://docs.aws.amazon.com/greengrass/latest/developerguide/images/ml-inference/lambda-publish-version.png)

1. For **Version description**, type **First version**, and then choose **Publish**\.

1. On the **greengrassObjectClassification: 1** configuration page, from the **Actions** menu, choose **Create alias**\.  
![\[The Create alias option in the Actions menu.\]](http://docs.aws.amazon.com/greengrass/latest/developerguide/images/ml-inference/lambda-create-alias.png)

1. On the **Create a new alias** page, use the following values:    
[\[See the AWS documentation website for more details\]](http://docs.aws.amazon.com/greengrass/latest/developerguide/ml-console.html)
**Note**  
AWS Greengrass doesn't support Lambda aliases for **$LATEST** versions\.

1. Choose **Create**\.  
![\[The Create a new alias page with Create highlighted.\]](http://docs.aws.amazon.com/greengrass/latest/developerguide/images/ml-inference/lambda-create-alias-create.png)

   Now, add the Lambda function to your Greengrass group\.

## Step 5: Add the Lambda Function to the Greengrass Group<a name="ml-console-config-lambda"></a>

In this step, you add the Lambda function to the group and then configure its lifecycle\.

First, add the Lambda function to your Greengrass group\.

1. In the AWS IoT console, open the group configuration page\.

1. Choose **Lambdas**, and then choose **Add Lambda**\.  
![\[The group page with Lambdas and Add Lambda highlighted.\]](http://docs.aws.amazon.com/greengrass/latest/developerguide/images/console-group-lambdas.png)

1. On the **Add a Lambda to your Greengrass Group** page, choose **Use existing Lambda**\.  
![\[The Add a Lambda to your Greengrass Group page with Use existing Lambda highlighted.\]](http://docs.aws.amazon.com/greengrass/latest/developerguide/images/console-group-lambdas-existing-lambda.png)

1. On the **Use existing Lambda** page, choose **greengrassObjectClassification**, and then choose **Next**\.

1. On the **Select a Lambda version** page, choose **Alias:mlTest**, and then choose **Finish**\.

 

Next, configure the lifecycle of the Lambda function\.

1. On the **Lambdas** page, choose the **greengrassObjectClassification** Lambda function\.  
![\[The Lambdas page with the greengrassObjectClassification Lambda function highlighted.\]](http://docs.aws.amazon.com/greengrass/latest/developerguide/images/ml-inference/lambda.png)

1. On the **greengrassObjectClassification** configuration page, choose **Edit**\.

1. On the **Group\-specific Lambda configuration** page, use the following values:    
[\[See the AWS documentation website for more details\]](http://docs.aws.amazon.com/greengrass/latest/developerguide/ml-console.html)

   For more information, see [Lifecycle Configuration for Greengrass Lambda Functions](lambda-functions.md#lambda-lifecycle)\.  
![\[The greengrassObjectClassification page with updated properties.\]](http://docs.aws.amazon.com/greengrass/latest/developerguide/images/ml-inference/lambda-config.png)

1. At the bottom of the page, choose **Update**\.

## Step 6: Add Resources to the Greengrass Group<a name="ml-console-add-resources"></a>

In this step, you create resources for the camera module and the ML inference model\. You also affiliate the resources with the Lambda function, which enables the function to access the resources on the core device\.

First, create two local device resources for the camera: one for shared memory and one for the device interface\. For more information about local resource access, see [Access Local Resources with Lambda Functions](access-local-resources.md)\.

1. On the group configuration page, choose **Resources**\.  
![\[The group configuration page with Resources highlighted.\]](http://docs.aws.amazon.com/greengrass/latest/developerguide/images/console-group-resources.png)

1. On the **Local Resources** tab, choose **Add local resource**\.

1. On the **Create a local resource** page, use the following values:    
[\[See the AWS documentation website for more details\]](http://docs.aws.amazon.com/greengrass/latest/developerguide/ml-console.html)

   The **Device path** is the local absolute path of the device resource\. This path can only refer to a character device or block device under /dev\.

   The **Group owner file access permission** option lets you grant additional file access permissions to the Lambda process\. For more information, see [Group Owner File Access Permission](access-local-resources.md#lra-group-owner)\.  
![\[The Create a local resource page with edited resource properties.\]](http://docs.aws.amazon.com/greengrass/latest/developerguide/images/ml-inference/local-resource-vcsm.png)

1. Under **Lambda function affiliations**, choose **Select**\.

1. Choose **greengrassObjectClassification**, choose **Read and write access**, and then choose **Done**\.  
![\[Lambda function affiliation properties with Done highlighted.\]](http://docs.aws.amazon.com/greengrass/latest/developerguide/images/ml-inference/local-resource-vcsm-lambda.png)

   Next, you add a local device resource for the camera interface\.

1. At the bottom of the page, choose **Add another resource**\.

1. On the **Create a local resource** page, use the following values:    
[\[See the AWS documentation website for more details\]](http://docs.aws.amazon.com/greengrass/latest/developerguide/ml-console.html)  
![\[The Create a local resource page with edited resource properties.\]](http://docs.aws.amazon.com/greengrass/latest/developerguide/images/ml-inference/local-resource-vchiq.png)

1. Under **Lambda function affiliations**, choose **Select**\.

1. Choose **greengrassObjectClassification**, choose **Read and write access**, and then choose **Done**\.

1. At the bottom of the page, choose **Save**\.

 

Now, add the inference model as a machine learning resource\. This step includes uploading the `squeezenet.zip` model package to Amazon S3\.

1. On the **Machine Learning** tab, choose **Add machine learning resource**\.

1. On the **Create a machine learning resource** page, for **Resource name**, type **squeezenet\_model**\.

1. For **Model source**, choose **Locate or upload a model in S3**\.

1. Under **Model from S3**, choose **Select**, and then choose **Create S3 bucket**\.  
![\[The Create a machine learning resource page with Create S3 bucket highlighted.\]](http://docs.aws.amazon.com/greengrass/latest/developerguide/images/ml-inference/ml-resource-squeezenet.png)

1. For **Bucket name**, type a name that contains the string **greengrass** \(such as **greengrass\-*datetime***\), and then choose **Create**\.
**Note**  
Don't use a period \("`.`"\) in the bucket name\.

1. Choose **Upload a model**, and then choose the `squeezenet.zip` package that you created in [Step 3: Create an MXNet Model Package](#package-ml-model)\.

1. For **Local path**, type **/greengrass\-machine\-learning/mxnet/squeezenet**\.

   This is the destination for the local model in the Lambda runtime namespace\. When you deploy the group, AWS Greengrass retrieves the source model package and then extracts the contents to the specified directory\. The sample Lambda function for this tutorial is already configured to use this path \(in the `model_path` variable\)\.

1. Under **Lambda function affiliations**, choose **Select**\.

1. Choose **greengrassObjectClassification**, choose **Read\-only access**, and then choose **Done**\.

1. At the bottom of the page, choose **Save**\.

### Using Amazon SageMaker Trained Models<a name="sm-models"></a>

This tutorial uses a model that's stored in Amazon S3, but you can easily use Amazon SageMaker models too\. The Greengrass console has built\-in Amazon SageMaker integration, so you don't need to manually upload these models to Amazon S3\. For requirements and limitations for using Amazon SageMaker models, see [Supported Model Sources](ml-inference.md#supported-model-sources)\.

To use an Amazon SageMaker model:
+ For **Model source**, choose **Use an existing SageMaker model**, and then choose the name of the model's training job\.
+ For **Local path**, type the path to the directory where your Lambda function looks for the model\.

## Step 7: Add a Subscription to the Greengrass Group<a name="ml-console-add-subscription"></a>

In this step, you add a subscription to the group\. This subscription enables the Lambda function to send prediction results to AWS IoT by publishing to an MQTT topic\.

1. On the group configuration page, choose **Subscriptions**, and then choose **Add Subscription**\.  
![\[The group page with Subscriptions and Add Subscription highlighted.\]](http://docs.aws.amazon.com/greengrass/latest/developerguide/images/console-group-subscriptions.png)

1. On the **Select your source and target** page, configure the source and target, as follows:

   1. In **Select a source**, choose **Lambdas**, and then choose **greengrassObjectClassification**\.

   1. In **Select a target**, choose **Services**, and then choose **IoT Cloud**\.

   1. Choose **Next**\.  
![\[The Select your source and target page with Next highlighted.\]](http://docs.aws.amazon.com/greengrass/latest/developerguide/images/ml-inference/add-subscription.png)

1. On the **Filter your data with a topic** page, in the **Optional topic filter** field, type **hello/world**, and then choose **Next**\.  
![\[The Filter your data with a topic page with Next highlighted.\]](http://docs.aws.amazon.com/greengrass/latest/developerguide/images/ml-inference/add-subscription-topic.png)

1. Choose **Finish**\.

## Step 8: Deploy the Greengrass Group<a name="ml-console-deploy-group"></a>

In this step, you deploy the current version of the group definition to the Greengrass core device\. The definition contains the Lambda function, resources, and subscription configurations that you added\.

1. Make sure that the AWS Greengrass core is running\. Run the following commands in your Raspberry Pi terminal, as needed\.

   1. To check whether the daemon is running:

      ```
      ps aux | grep -E 'greengrass.*daemon'
      ```

      If the output contains a `root` entry for `/greengrass/ggc/packages/1.6.0/bin/daemon`, then the daemon is running\.
**Note**  
The version in the path depends on the AWS Greengrass Core software version that's installed on your core device\.

   1. To start the daemon:

      ```
      cd /greengrass/ggc/core/
      sudo ./greengrassd start
      ```

1. On the group configuration page, choose **Deployments**, and from the **Actions** menu, choose **Deploy**\.  
![\[The group page with Deployments and Deploy highlighted.\]](http://docs.aws.amazon.com/greengrass/latest/developerguide/images/console-group-deployments-deploy.png)

1. On the **Configure how devices discover your core** page, choose **Automatic detection**\.

   This enables devices to automatically acquire connectivity information for the core, such as IP address, DNS, and port number\. Automatic detection is recommended, but AWS Greengrass also supports manually specified endpoints\. You're only prompted for the discovery method the first time that the group is deployed\.  
![\[The Configure how devices discover your core page with Automatic detection highlighted.\]](http://docs.aws.amazon.com/greengrass/latest/developerguide/images/console-discovery.png)
**Note**  
If prompted, grant permission to create the AWS Greengrass service role on your behalf, which allows AWS Greengrass to access other AWS services\. You need to do this only one time per account\.

   The **Deployments** page shows the deployment time stamp, version ID, and status\. When completed, the deployment should show a **Successfully completed** status\.

   For help troubleshooting any issues that you encounter, see [Troubleshooting AWS Greengrass Applications](gg-troubleshooting.md)\.

## Test the Inference App<a name="test-app"></a>

Now you can verify whether the deployment is configured correctly\. To test, you subscribe to the **hello/world** topic and view the prediction results that are published by the Lambda function\.

**Note**  
If a monitor is attached to the Raspberry Pi, the live camera feed is displayed in a preview window\.

1. On the AWS IoT console home page, choose **Test**\.  
![\[The left pane in the AWS IoT console with Test highlighted.\]](http://docs.aws.amazon.com/greengrass/latest/developerguide/images/console-test.png)

1. For **Subscriptions**, use the following values:    
[\[See the AWS documentation website for more details\]](http://docs.aws.amazon.com/greengrass/latest/developerguide/ml-console.html)

1. Choose **Subscribe to topic**\.

   If the test is successful, the messages from the Lambda function appear at the bottom of the page\. Each message contains the top five prediction results of the image, using the format: probability, predicted class ID, and corresponding class name\.  
![\[The Subscriptions page showing test results with message data.\]](http://docs.aws.amazon.com/greengrass/latest/developerguide/images/ml-inference/prediction-results.png)

### Troubleshooting AWS Greengrass ML Inference<a name="ml-inference-troubleshooting"></a>

If the test is not successful, you can try the following troubleshooting steps\. Run the commands in your Raspberry Pi terminal\.

#### Check Error Logs<a name="troubleshooting-check-logs"></a>

1. Switch to the root user\.

   ```
   sudo su
   ```

1. Navigate to the /log directory\.

   ```
   cd /greengrass/ggc/var/log
   ```

1. Check `runtime.log` or `python_runtime.log`\. 

   For more information, see [Troubleshooting with Logs](gg-troubleshooting.md#troubleshooting-logs)\.

##### "Unpacking" Error in runtime\.log<a name="troubleshooting-targz-unpacking"></a>

If `runtime.log` contains an error similar to the following, ensure that your `tar.gz` source model package has a parent directory\.

```
Greengrass deployment error: unable to download the artifact model-arn: Error while processing. 
Error while unpacking the file from /tmp/greengrass/artifacts/model-arn/path to /greengrass/ggc/deployment/path/model-arn,
error: open /greengrass/ggc/deployment/path/model-arn/squeezenet/squeezenet_v1.1-0000.params: no such file or directory
```

If your package doesn't have a parent directory that contains the model files, try repackaging the model using the following command:

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

1. List the contents of the deployed Lambda in the /lambda directory\. Replace the placeholder values before running the command\.

   ```
   cd /greengrass/ggc/deployment/lambda/arn:aws:lambda:region:account:function:function-name:function-version
   ls -la
   ```

1. Verify that the directory contains the same content as the **greengrassObjectClassification\.zip** deployment package that you uploaded in [Step 4: Create and Publish a Lambda Function](#ml-console-create-lambda)\.

   Also make sure that the `.py` files and dependencies are in the root of the directory\.

 

#### Verify That the Inference Model Is Successfully Deployed<a name="troubleshooting-check-model"></a>

1. Find the process identification number \(PID\) of the Lambda runtime process:

   ```
   ps aux | grep lambda-function-name
   ```

   In the output, the PID appears in the second column of the line for the Lambda runtime process\.

1. Enter the Lambda runtime namespace\. Be sure to replace the placeholder *pid* value before running the command\.
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

Next, explore other inference apps\. AWS Greengrass provides other Lambda functions that you can use to try out local inference\. You can find the examples package in the precompiled libraries folder that you downloaded in [Step 2: Install the MXNet Framework](#install-mxnet)\.

## Configuring an NVIDIA Jetson TX2<a name="jetson-lambda-config"></a>

To run this tutorial on the GPU of an NVIDIA Jetson TX2, you must add additional local device resources and configure access for the Lambda function\.

**Note**  
Your Jetson must be configured before you can install the AWS Greengrass Core software\. For more information, see [Configuring NVIDIA Jetson TX2 for AWS Greengrass](module1.md#setup-jetson)\.

1. Add the following local device resources\. Follow the procedure in [Add Resources to the Group](#ml-console-add-resources)\.

   For each resource:
   + For **Resource type**, choose **Device**\.
   + For **Group owner file access permission**, choose **Automatically add OS group permissions of the Linux group that owns the resource**\.
   + For **Lambda function affiliations**, grant **Read and write access** to your Lambda function\.

        
[\[See the AWS documentation website for more details\]](http://docs.aws.amazon.com/greengrass/latest/developerguide/ml-console.html)

1. Edit the configuration of the Lambda function to increase **Memory limit** to 1000 MB\. Follow the procedure in [Add the Lambda Function to the Group](#ml-console-config-lambda)\.