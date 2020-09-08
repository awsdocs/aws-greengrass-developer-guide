# How to configure optimized machine learning inference using the AWS Management Console<a name="ml-dlc-console"></a>

To follow the steps in this tutorial, you must be using AWS IoT Greengrass Core v1\.10 or later\.

 You can use the SageMaker Neo deep learning compiler to optimize the prediction efficiency of native machine learning inference models in Tensorflow, Apache MXNet, PyTorch, ONNX, and XGBoost frameworks for a smaller footprint and faster performance\. You can then download the optimized model and install the SageMaker Neo deep learning runtime and deploy them to your AWS IoT Greengrass devices for faster inference\. 

This tutorial describes how to use the AWS Management Console to configure a Greengrass group to run a Lambda inference example that recognizes images from a camera locally, without sending data to the cloud\. The inference example accesses the camera module on a Raspberry Pi\. In this tutorial, you download a prepackaged model that is trained by Resnet\-50 and optimized in the Neo deep learning compiler\. You then use the model to perform local image classification on your AWS IoT Greengrass device\. 

The tutorial contains the following high\-level steps:

1. [Configure the Raspberry Pi](#config-raspberry-pi-dlc)

1. [Install the Neo deep learning runtime](#install-dlr)

1. [Create an inference Lambda function](#ml-console-dlc-create-lambda)

1. [Add the Lambda function to the group](#ml-console-dlc-config-lambda)

1. [Add a Neo\-optimized model resource to the group](#ml-console-dlc-add-resources)

1. [Add your camera device resource to the group](#ml-console-dlc-add-cam-resource)

1. [Add subscriptions to the group](#ml-console-dlc-add-subscription)

1. [Deploy the group](#ml-console-dlc-deploy-group)

1. [Test the example](#ml-console-dlc-test-app)

## Prerequisites<a name="ml-inference-prerequisites"></a>

 To complete this tutorial, you need: 
+  Raspberry Pi 4 Model B, or Raspberry Pi 3 Model B/B\+, set up and configured for use with AWS IoT Greengrass\. To set up your Raspberry Pi with AWS IoT Greengrass, run the [Greengrass Device Setup](quick-start.md) script, or make sure that you have completed [Module 1](https://docs.aws.amazon.com/greengrass/latest/developerguide/module1.html) and [Module 2](https://docs.aws.amazon.com/greengrass/latest/developerguide/module2.html) of [Getting started with AWS IoT Greengrass](gg-gs.md)\. 
**Note**  
The Raspberry Pi might require a 2\.5A [power supply](https://www.raspberrypi.org/documentation/hardware/raspberrypi/power/) to run the deep learning frameworks that are typically used for image classification\. A power supply with a lower rating might cause the device to reboot\.
+  [Raspberry Pi Camera Module V2 \- 8 megapixel, 1080p](https://www.amazon.com/Raspberry-Pi-Camera-Module-Megapixel/dp/B01ER2SKFS)\. To learn how to set up the camera, see [Connecting the camera](https://www.raspberrypi.org/documentation/usage/camera/) in the Raspberry Pi documentation\. 
+  A Greengrass group and a Greengrass core\. To learn how to create a Greengrass group or core, see [Getting started with AWS IoT Greengrass](gg-gs.md)\. 

**Note**  
 This tutorial uses a Raspberry Pi, but AWS IoT Greengrass supports other platforms, such as [Intel Atom](#atom-lambda-dlc-config) and [NVIDIA Jetson TX2](#jetson-lambda-dlc-config)\. If using the Intel Atom example, you might need to install Python 3\.6 instead of Python 3\.7\. For information about configuring your device so you can install the AWS IoT Greengrass Core software, see [Setting up other devices](setup-filter.other.md)\.   
For third party platforms that AWS IoT Greengrass does not support, you must run your Lambda function in non\-containerized mode\. To run in non\-containerized mode, you must run your Lambda function as root\. For more information, see [Considerations when choosing Lambda function containerization](lambda-group-config.md#lambda-containerization-considerations) and [Setting the default access identity for Lambda functions in a group](lambda-group-config.md#lambda-access-identity-groupsettings)\.

## Step 1: Configure the Raspberry Pi<a name="config-raspberry-pi-dlc"></a>

 In this step, install updates to the Raspbian operating system, install the camera module software and Python dependencies, and enable the camera interface\. 

Run the following commands in your Raspberry Pi terminal\.

1. Install updates to Raspbian\.

   ```
   sudo apt-get update
   sudo apt-get dist-upgrade
   ```

1. <a name="install-picamera-step"></a>Install the `picamera` interface for the camera module and other Python libraries that are required for this tutorial\.

   ```
   sudo apt-get install -y python3-dev python3-setuptools python3-pip python3-picamera
   ```

   Validate the installation:
   + Make sure that your Python 3\.7 installation includes pip\.

     ```
     python3 -m pip
     ```

     If pip isn't installed, download it from the [pip website](https://pip.pypa.io/en/stable/installing/) and then run the following command\.

     ```
     python3 get-pip.py
     ```
   + Make sure that your Python version is 3\.7 or higher\.

     ```
     python3 --version
     ```

     If the output lists an earlier version, run the following command\.

     ```
     sudo apt-get install -y python3.7-dev
     ```
   + Make sure that Setuptools and Picamera installed successfully\.

     ```
     sudo -u ggc_user bash -c 'python3 -c "import setuptools"'
     sudo -u ggc_user bash -c 'python3 -c "import picamera"'
     ```

     If the output doesn't contain errors, the validation is successful\.
**Note**  
If the Python executable installed on your device is `python3.7`, use `python3.7` instead of `python3` for the commands in this tutorial\. Make sure that your pip installation maps to the correct `python3.7` or `python3` version to avoid dependency errors\.

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

   This opens a preview window on the Raspberry Pi, saves a picture named `test.jpg` to your current directory, and displays information about the camera in the Raspberry Pi terminal\.

## Step 2: Install the Amazon SageMaker Neo deep learning runtime<a name="install-dlr"></a>

 In this step, install the Neo deep learning runtime \(DLR\) on your Raspberry Pi\. 

**Note**  
We recommend installing version 1\.1\.0 for this tutorial\.

1. <a name="ssh-rpi-step"></a>Sign in to your Raspberry Pi remotely\.

   ```
   ssh pi@your-device-ip-address
   ```

1.  Open the DLR documentation, open [Installing DLR](https://neo-ai-dlr.readthedocs.io/en/latest/install.html), and locate the wheel URL for Raspberry Pi devices\. Then, follow the instructions to install the DLR on your device\. For example, you can use pip:

   ```
   pip3 install rasp3b-wheel-url
   ```

1. After you install the DLR, validate the following configuration:
   + Make sure the `ggc_user` system account can use the DLR library\.

     ```
     sudo -u ggc_user bash -c 'python3 -c "import dlr"'
     ```
   + Make sure NumPy is installed\.

     ```
     sudo -u ggc_user bash -c 'python3 -c "import numpy"'
     ```

## Step 3: Create an inference Lambda function<a name="ml-console-dlc-create-lambda"></a>

 In this step, create a Lambda function deployment package and Lambda function\. Then, publish a function version and create an alias\. 

1. On your computer, download the DLR sample for Raspberry Pi from [Machine learning samples](what-is-gg.md#gg-ml-samples)\.

1.  Unzip the downloaded `dlr-py3-armv7l.tar.gz` file\. 

   ```
   cd path-to-downloaded-sample
   tar -xvzf dlr-py3-armv7l.tar.gz
   ```

   The `examples` directory in the extracted sample package contains function code and dependencies\.
   + `inference.py` is the inference code used in this tutorial\. You can use this code as a template to create your own inference function\.
   + <a name="ml-samples-ggc-sdk"></a>`greengrasssdk` is version 1\.5\.0 of the AWS IoT Greengrass Core SDK for Python\.
**Note**  <a name="ml-samples-ggc-sdk-upgrade"></a>
If a new version is available, you can download it and upgrade the SDK version in your deployment package\. For more information, see [ AWS IoT Greengrass Core SDK for Python](https://github.com/aws/aws-greengrass-core-sdk-python/) on GitHub\.

1.  Compress the contents of the `examples` directory into a file named `optimizedImageClassification.zip`\. This is your deployment package\. 

   ```
   cd path-to-downloaded-sample/dlr-py3-armv7l/examples
   zip -r optimizedImageClassification.zip .
   ```

    The deployment package contains your function code and dependencies\. This includes the code that invokes the Neo deep learning runtime Python APIs to perform inference with the Neo deep learning compiler models\. 
**Note**  <a name="ml-samples-function-zip"></a>
 Make sure the `.py` ﬁles and dependencies are in the root of the directory\. 

1.  Now, add the Lambda function to your Greengrass group\. 

    In the AWS IoT console, in the navigation pane, choose **Greengrass**, and then choose **Groups**\.   
![\[The navigation pane in the AWS IoT console with Groups highlighted.\]](http://docs.aws.amazon.com/greengrass/latest/developerguide/images/console-groups.png)

1. Choose the Greengrass group where you want to add the Lambda function\.

1. On the group configuration page, choose **Lambdas**, and then choose **Add Lambda**\.  
![\[The group page with Lambdas and Add Lambda highlighted.\]](http://docs.aws.amazon.com/greengrass/latest/developerguide/images/console-group-lambdas.png)

1.  On the **Add a Lambda to your Greengrass Group** page, choose **Create new Lambda**\. This opens the AWS Lambda console\.   
![\[The Add a Lambda to your Greengrass Group page with Create new Lambda highlighted.\]](http://docs.aws.amazon.com/greengrass/latest/developerguide/images/console-group-lambdas-new-lambda.png)

1. Choose **Author from scratch** and use the following values to create your function:
   + For **Function name**, enter **optimizedImageClassification**\. 
   + For **Runtime**, choose **Python 3\.7**\.

   For **Permissions**, keep the default setting\. This creates an execution role that grants basic Lambda permissions\. This role isn't used by AWS IoT Greengrass\.  
![\[The Basic information section of the Create function page.\]](http://docs.aws.amazon.com/greengrass/latest/developerguide/images/ml-dlc-inference/gg-dlr-lambda-creation.png)

1.  Choose **Create function**\. 

 

Now, upload your Lambda function deployment package and register the handler\.

1.  On the **Configuration** tab for the `optimizedImageClassification` function, for **Function code**, use the following values: 
   + For **Code entry type**, choose **Upload a \.zip file**\.
   + For **Runtime**, choose **Python 3\.7**\.
   + For **Handler**, enter **inference\.handler**\.

1. Choose **Upload**\.  
![\[The Function code section with Upload highlighted.\]](http://docs.aws.amazon.com/greengrass/latest/developerguide/images/ml-dlc-inference/gg-ml2-lambda-upload.png)

1. Choose your `optimizedImageClassification.zip` deployment package\.

1.  Choose **Save**\.

 

Next, publish the first version of your Lambda function\. Then, create an [alias for the version](https://docs.aws.amazon.com/lambda/latest/dg/versioning-aliases.html)\.

**Note**  
Greengrass groups can reference a Lambda function by alias \(recommended\) or by version\. Using an alias makes it easier to manage code updates because you don't have to change your subscription table or group definition when the function code is updated\. Instead, you just point the alias to the new function version\.

1. From the **Actions** menu, choose **Publish new version**\.  
![\[The Publish new version option in the Actions menu.\]](http://docs.aws.amazon.com/greengrass/latest/developerguide/images/ml-dlc-inference/gg-ml2-publish-new.png)

1. For **Version description**, enter **First version**, and then choose **Publish**\.

1. On the **optimizedImageClassification: 1** configuration page, from the **Actions** menu, choose **Create alias**\.  
![\[The Create alias option in the Actions menu.\]](http://docs.aws.amazon.com/greengrass/latest/developerguide/images/ml-dlc-inference/gg-ml2-create-alias.png)

1. On the **Create a new alias** page, use the following values:
   + For **Name**, enter **mlTestOpt**\.
   + For **Version**, enter **1**\.
**Note**  
AWS IoT Greengrass doesn't support Lambda aliases for **$LATEST** versions\.

1. Choose **Create**\.

   Now, add the Lambda function to your Greengrass group\.

## Step 4: Add the Lambda function to the Greengrass group<a name="ml-console-dlc-config-lambda"></a>

In this step, add the Lambda function to the group, and then configure its lifecycle\.

First, add the Lambda function to your Greengrass group\.

1.  On the **Add a Lambda to your Greengrass Group** page, choose **Use existing Lambda**\.   
![\[The Add a Lambda to your Greengrass Group page with Use existing Lambda highlighted.\]](http://docs.aws.amazon.com/greengrass/latest/developerguide/images/console-group-lambdas-existing-lambda.png)

1.  Choose **optimizedImageClassification**, and then choose **Next**\. 

1. On the **Select a Lambda version** page, choose **Alias:mlTestOpt**, and then choose **Finish**\.

 

Next, configure the lifecycle of the Lambda function\.

1. On the **Lambdas** page, choose the **optimizedImageClassification** Lambda function\.  
![\[The Lambdas page with the optimizedImageClassification Lambda function highlighted.\]](http://docs.aws.amazon.com/greengrass/latest/developerguide/images/ml-dlc-inference/gg-ml2-lambda-page.png)

1. On the **optimizedImageClassification** configuration page, choose **Edit**\.

1. On the **Group\-specific Lambda configuration** page, make the following updates\.
**Note**  
 We recommend that you run your Lambda function without containerization unless your business case requires it\. This helps enable access to your device GPU and camera without configuring device resources\. If you run without containerization, you must also grant root access to your AWS IoT Greengrass Lambda functions\. 

   1. **To run without containerization:**
      + For **Run as**, choose **Another user ID/group ID**\. For **UID**, enter **0**\. For **GUID**, enter **0**\.

        This allows your Lambda function to run as root\. For more information about running as root, see [Setting the default access identity for Lambda functions in a group](lambda-group-config.md#lambda-access-identity-groupsettings)\.
**Tip**  
You also must update your `config.json` file to grant root access to your Lambda function\. For the procedure, see [Running a Lambda function as root](lambda-group-config.md#lambda-running-as-root)\.
      + For **Containerization**, choose **No container**\.

        For more information about running without containerization, see [Considerations when choosing Lambda function containerization](lambda-group-config.md#lambda-containerization-considerations)\.
      + For **Timeout**, enter **10 seconds**\.
      + For **Lambda lifecycle**, choose **Make this function long\-lived and keep it running indefinitely**\.

        For more information, see [Lifecycle configuration for Greengrass Lambda functions](lambda-functions.md#lambda-lifecycle)\.
      + For **Read access to /sys directory**, choose **Enable**\.

   1.  **To run in containerized mode instead:** 
**Note**  
We do not recommend running in containerized mode unless your business case requires it\.
      + For **Run as**, choose **Use group default**\.
      + For **Containerization**, choose **Use group default**\.
      + For **Memory limit**, enter **1024 MB**\.
      + For **Timeout**, enter **10 seconds**\.
      + For **Lambda lifecycle**, choose **Make this function long\-lived and keep it running indefinitely**\.

        For more information, see [Lifecycle configuration for Greengrass Lambda functions](lambda-functions.md#lambda-lifecycle)\.
      + For **Read access to /sys directory**, choose **Enable**\.

1.  Choose **Update**\.

## Step 5: Add a SageMaker Neo\-optimized model resource to the Greengrass group<a name="ml-console-dlc-add-resources"></a>

 In this step, create a resource for the optimized ML inference model and upload it to an Amazon S3 bucket\. Then, locate the Amazon S3 uploaded model in the AWS IoT Greengrass console and affiliate the newly created resource with the Lambda function\. This makes it possible for the function to access its resources on the core device\. 

1.  On your computer, navigate to the `resnet50` directory in the sample package that you unzipped in [Step 3: Create an inference Lambda function](#ml-console-dlc-create-lambda)\. 
**Note**  
If using the NVIDIA Jetson example, you need to use the `resnet18` directory in the sample package instead\. For more information, see [Configuring an NVIDIA Jetson TX2](#jetson-lambda-dlc-config)\.

   ```
   cd path-to-downloaded-sample/dlr-py3-armv7l/models/resnet50
   ```

    This directory contains precompiled model artifacts for an image classification model trained with Resnet\-50\.

1. Compress the files inside the `resnet50` directory into a file named `resnet50.zip`\. 

   ```
   zip -r resnet50.zip .
   ```

1.  On the group configuration page for your AWS IoT Greengrass group, choose **Resources**\. Navigate to the **Machine Learning** section and choose **Add machine learning resource**\. On the **Create a machine learning resource** page, for **Resource name**, enter **resnet50\_model**\.  
![\[The Add Machine Learning Model page with updated properties.\]](http://docs.aws.amazon.com/greengrass/latest/developerguide/images/ml-dlc-inference/gg-ml2-add-ml-model.png)

1. For **Model source**, choose **Upload a model in S3**\.

1.  Under **Model from S3**, choose **Select**\. 
**Note**  
 Currently, optimized SageMaker models are stored automatically in Amazon S3\. You can find your optimized model in your Amazon S3 bucket using this option\. For more information about model optimization in SageMaker, see the [SageMaker Neo documentation](https://docs.aws.amazon.com/sagemaker/latest/dg/neo.html)\. 

1.  Choose **Upload a model**\. 

1.  On the Amazon S3 console tab, upload your zip file to an Amazon S3 bucket\. For information, see [ How do I upload files and folders to an S3 bucket? ](https://docs.aws.amazon.com/AmazonS3/latest/user-guide/upload-objects.html) in the *Amazon Simple Storage Service Console User Guide*\. 
**Note**  
 Your bucket name must contain the string **greengrass**\. Choose a unique name \(such as **greengrass\-dlr\-bucket\-*user\-id*\-*epoch\-time***\)\. Don't use a period \(`.`\) in the bucket name\. 

1.  In the AWS IoT Greengrass console tab, locate and choose your Amazon S3 bucket\. Locate your uploaded `resnet50.zip` file, and choose **Select**\. You might need to refresh the page to update the list of available buckets and files\. 

1.  In **Local path**, enter **/ml\_model**\.   
![\[The updated local path.\]](http://docs.aws.amazon.com/greengrass/latest/developerguide/images/ml-dlc-inference/local-path.png)

    This is the destination for the local model in the Lambda runtime namespace\. When you deploy the group, AWS IoT Greengrass retrieves the source model package and then extracts the contents to the specified directory\. 
**Note**  
 We strongly recommend that you use the exact path provided for your local path\. Using a different local model destination path in this step causes some troubleshooting commands provided in this tutorial to be inaccurate\. If you use a different path, you must set up a `MODEL_PATH` environment variable that uses the exact path you provide here\. For information about environment variables, see [AWS Lambda environment variables](https://docs.aws.amazon.com/lambda/latest/dg/env_variables.html)\. 

1. Under **Identify resource owner and set access permissions**, choose **No OS group**\.

1. Under **Lambda function affiliations**, choose **Select**\.

1. Choose **optimizedImageClassification**, choose **Read\-only access**, and then choose **Done**\.

1.  Choose **Save**\. 

## Step 6: Add your camera device resource to the Greengrass group<a name="ml-console-dlc-add-cam-resource"></a>

 In this step, create a resource for the camera module and affiliate it with the Lambda function\. This makes it possible for the Lambda function to access the resource on the core device\. 

**Note**  
If you run in non\-containerized mode, AWS IoT Greengrass can access your device GPU and camera without configuring this device resource\. 

1. On the group configuration page, choose **Resources**\.  
![\[The group configuration page with Resources highlighted.\]](http://docs.aws.amazon.com/greengrass/latest/developerguide/images/console-group-resources.png)

1. On the **Local** tab, choose **Add local resource**\.

1. On the **Create a local resource** page, use the following values:
   + For **Resource name**, enter **videoCoreSharedMemory**\.
   + For **Resource type**, choose **Device**\.
   + For **Device path**, enter **/dev/vcsm**\.

     The device path is the local absolute path of the device resource\. This path can refer only to a character device or block device under `/dev`\.
   + For **Group owner file access permission**, choose **Automatically add OS group permissions of the Linux group that owns the resource**\.

     The **Group owner file access permission** option lets you grant additional file access permissions to the Lambda process\. For more information, see [Group owner file access permission](access-local-resources.md#lra-group-owner)\.  
![\[The Create a local resource page with edited resource properties.\]](http://docs.aws.amazon.com/greengrass/latest/developerguide/images/ml-inference/local-resource-vcsm.png)

1. Under **Lambda function affiliations**, choose **Select**\.

1. Choose **optimizedImageClassification**, choose **Read and write access**, and then choose **Done**\.  
![\[Lambda function affiliation properties with Done highlighted.\]](http://docs.aws.amazon.com/greengrass/latest/developerguide/images/ml-dlc-inference/gg-ml2-local-resource-vcsm.png)

   Next, you add a local device resource for the camera interface\.

1. At the bottom of the page, choose **Add another resource**\.

1. On the **Create a local resource** page, use the following values:
   + For **Resource name**, enter **videoCoreInterface**\.
   + For **Resource type**, choose **Device**\.
   + For **device path**, enter **/dev/vchiq**\.
   + For **Group owner file access permission**, choose **Automatically add OS group permissions of the Linux group that owns the resource**\.  
![\[The Create a local resource page with edited resource properties.\]](http://docs.aws.amazon.com/greengrass/latest/developerguide/images/ml-inference/local-resource-vchiq.png)

1. Under **Lambda function affiliations**, choose **Select**\.

1.  Choose **optimizedImageClassification**, choose **Read and write access**, and then choose **Done**\. 

1.  Choose **Save**\. 

## Step 7: Add subscriptions to the Greengrass group<a name="ml-console-dlc-add-subscription"></a>

In this step, add subscriptions to the group\. These subscriptions enable the Lambda function to send prediction results to AWS IoT by publishing to an MQTT topic\.

1. On the group configuration page, choose **Subscriptions**, and then choose **Add Subscription**\.  
![\[The group page with Subscriptions and Add Subscription highlighted.\]](http://docs.aws.amazon.com/greengrass/latest/developerguide/images/console-group-subscriptions.png)

1. On the **Select your source and target** page, configure the source and target, as follows:

   1. In **Select a source**, choose **Lambdas**, and then choose **optimizedImageClassification**\.

   1. In **Select a target**, choose **Services**, and then choose **IoT Cloud**\.

   1. Choose **Next**\.  
![\[The Select your source and target page with Next highlighted.\]](http://docs.aws.amazon.com/greengrass/latest/developerguide/images/ml-dlc-inference/gg-ml2-subsc-1.png)

1.  On the **Filter your data with a topic** page, in **Optional topic filter**, enter **/resnet\-50/predictions**, and then choose **Next**\.   
![\[The Filter your data with a topic page with Next highlighted.\]](http://docs.aws.amazon.com/greengrass/latest/developerguide/images/ml-dlc-inference/gg-ml2-topic-filter-s1.png)

1. Choose **Finish**\.

1.  Add a second subscription\. On the **Select your source and target** page, configure the source and target, as follows: 

   1. In **Select a source**, choose **Services**, and then choose **IoT Cloud**\.

   1. In **Select a target**, choose **Lambdas**, and then choose **optimizedImageClassification**\.

   1. Choose **Next**\.

1.  On the **Filter your data with a topic** page, in **Optional topic filter**, enter **/resnet\-50/test**, and then choose **Next**\.

1. Choose **Finish**\.

## Step 8: Deploy the Greengrass group<a name="ml-console-dlc-deploy-group"></a>

In this step, deploy the current version of the group definition to the Greengrass core device\. The definition contains the Lambda function, resources, and subscription configurations that you added\.

1. Make sure that the AWS IoT Greengrass core is running\. Run the following commands in your Raspberry Pi terminal, as needed\.

   1. To check whether the daemon is running:

      ```
      ps aux | grep -E 'greengrass.*daemon'
      ```

      If the output contains a `root` entry for `/greengrass/ggc/packages/latest-core-version/bin/daemon`, then the daemon is running\.

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
![\[The Deployments page with a successful deployment status highlighted.\]](http://docs.aws.amazon.com/greengrass/latest/developerguide/images/ml-dlc-inference/gg-ml2-successful-deployment.png)

   For more information about deployments, see [Deploy AWS IoT Greengrass groups to an AWS IoT Greengrass core](deployments.md)\. For troubleshooting help, see [Troubleshooting AWS IoT Greengrass](gg-troubleshooting.md)\.

## Test the inference example<a name="ml-console-dlc-test-app"></a>

Now you can verify whether the deployment is configured correctly\. To test, you subscribe to the `/resnet-50/predictions` topic and publish any message to the `/resnet-50/test` topic\. This triggers the Lambda function to take a photo with your Raspberry Pi and perform inference on the image it captures\. 

**Note**  
If using the NVIDIA Jetson example, make sure to use the `resnet-18/predictions` and `resnet-18/test` topics instead\.

**Note**  
If a monitor is attached to the Raspberry Pi, the live camera feed is displayed in a preview window\.

1. On the AWS IoT console home page, choose **Test**\.  
![\[The navigation pane in the AWS IoT console with Test highlighted.\]](http://docs.aws.amazon.com/greengrass/latest/developerguide/images/console-test.png)

1.  For **Subscriptions**, choose **Subscribe to a Topic**\. Use the following values\. Leave the remaining options at their defaults\. 
   + For **Subscription topic**, enter **/resnet\-50/predictions**\.
   + For **MQTT payload display**, choose **Display payloads as strings**\.

1. Choose **Subscribe to topic**\.

1. On the `/resnet-50/predictions` page, specify the `/resnet-50/test` topic to publish to\. Choose **Publish to topic**\. 

1.  If the test is successful, the published message causes the Raspberry Pi camera to capture an image\. A message from the Lambda function appears at the bottom of the page\. This message contains the prediction result of the image, using the format: predicted class name, probability, and peak memory usage\.   
![\[The Subscriptions page showing test results with message data.\]](http://docs.aws.amazon.com/greengrass/latest/developerguide/images/ml-dlc-inference/prediction-results.png)

## Configuring an Intel Atom<a name="atom-lambda-dlc-config"></a>

 To run this tutorial on an Intel Atom device, you must provide source images, configure the Lambda function, and add another local device resource\. To use the GPU for inference, make sure the following software is installed on your device:
+ OpenCL version 1\.0 or later
+ Python 3\.7 and pip
+ [NumPy](https://pypi.org/project/numpy/)
+ [OpenCV on Wheels](https://pypi.org/project/opencv-python/)

1. Download static PNG or JPG images for the Lambda function to use for image classification\. The example works best with small image files\. 

   Save your image files in the directory that contains the `inference.py` file \(or in a subdirectory of this directory\)\. This is in the Lambda function deployment package that you upload in [Step 3: Create an inference Lambda function](#ml-console-dlc-create-lambda)\.
**Note**  
 If you're using AWS DeepLens, you can use the onboard camera or mount your own camera to perform inference on captured images instead of static images\. However, we strongly recommend you start with static images first\.   
If you use a camera, make sure that the `awscam` APT package is installed and up to date\. For more information, see [Update your AWS DeepLens device](https://docs.aws.amazon.com/deeplens/latest/dg/deeplens-manual-updates.html) in the *AWS DeepLens Developer Guide*\.

1. Edit the configuration of the Lambda function\. Follow the procedure in [Step 4: Add the Lambda function to the Greengrass group](#ml-console-dlc-config-lambda)\. 
**Note**  
 We recommend that you run your Lambda function without containerization unless your business case requires it\. This helps enable access to your device GPU and camera without configuring device resources\. If you run without containerization, you must also grant root access to your AWS IoT Greengrass Lambda functions\. 

   1. **To run without containerization:**
      + For **Run as**, choose **Another user ID/group ID**\. For **UID**, enter **0**\. For **GUID**, enter **0**\.

        This allows your Lambda function to run as root\. For more information about running as root, see [Setting the default access identity for Lambda functions in a group](lambda-group-config.md#lambda-access-identity-groupsettings)\.
**Tip**  
You also must update your `config.json` file to grant root access to your Lambda function\. For the procedure, see [Running a Lambda function as root](lambda-group-config.md#lambda-running-as-root)\.
      + For **Containerization**, choose **No container**\.

        For more information about running without containerization, see [Considerations when choosing Lambda function containerization](lambda-group-config.md#lambda-containerization-considerations)\.
      + Increase the **Timeout** value to 2 minutes\. This ensures that the request does not time out too early\. It takes a few minutes after setup to run inference\.
      +  For **Read access to /sys directory**, choose **Enable**\. 
      +  For **Lambda lifecycle**, choose **Make this function long\-lived and keep it running indefinitely**\. 

   1.  **To run in containerized mode instead:** 
**Note**  
We do not recommend running in containerized mode unless your business case requires it\.
      +  Increase the **Memory limit** value to 3000 MB\. 
      + Increase the **Timeout** value to 2 minutes\. This ensures that the request does not time out too early\. It takes a few minutes after setup to run inference\.
      +  For **Read access to /sys directory**, choose **Enable**\. 
      +  For **Lambda lifecycle**, choose **Make this function long\-lived and keep it running indefinitely**\. 

1.  Add your Neo\-optimized model resource to the group\. Upload the model resources in the `resnet18` directory of the sample package you unzipped in [Step 3: Create an inference Lambda function](#ml-console-dlc-create-lambda)\. This directory contains precompiled model artifacts for an image classification model trained with Resnet\-18\. Follow the procedure in [Step 5: Add a SageMaker Neo\-optimized model resource to the Greengrass group](#ml-console-dlc-add-resources) with the following updates\. 
   + Compress the files inside the `resnet18` directory into a file named `resnet18.zip`\.
   + On the **Create a machine learning resource** page, for **Resource name**, enter **resnet18\_model**\.
   + Upload the `resnet18.zip` file\.

1. **If running in containerized mode**, add the required local device resource to grant access to your device GPU\.
**Note**  
 If you run in non\-containerized mode, AWS IoT Greengrass can access your device GPU without configuring device resources\. 

   1. On the group configuration page, choose **Resources**\.  
![\[The group configuration page with Resources highlighted.\]](http://docs.aws.amazon.com/greengrass/latest/developerguide/images/console-group-resources.png)

   1. On the **Local** tab, choose **Add a local resource**\.

   1. Define the resource:
      + For **Resource name**, enter **renderD128**\.
      + For **Resource type**, choose **Device**\.
      + For **Device path**, enter **/dev/dri/renderD128**\.
      + For **Group owner file access permission**, choose **Automatically add OS group permissions of the Linux group that owns the resource**\.
      + For **Lambda function affiliations**, grant **Read and write access** to your Lambda function\.

## Configuring an NVIDIA Jetson TX2<a name="jetson-lambda-dlc-config"></a>

 To run this tutorial on an NVIDIA Jetson TX2, provide source images, configure the Lambda function, and add more local device resources\.

1. Make sure your Jetson device is configured so you can install the AWS IoT Greengrass Core software and use the GPU for inference\. For more information about configuring your device, see [Setting up other devices](setup-filter.other.md)\. To use the GPU for inference on an NVIDIA Jetson TX2, you must install CUDA 10\.0 and cuDNN 7\.0 on your device when you image your board with Jetpack 4\.3\.

1. Download static PNG or JPG images for the Lambda function to use for image classification\. The example works best with small image files\. 

   Save your image files in the directory that contains the `inference.py` file\. You can also save them in a subdirectory of this directory\. This directory is in the Lambda function deployment package that you upload in [Step 3: Create an inference Lambda function](#ml-console-dlc-create-lambda)\.
**Note**  
 You can instead choose to instrument a camera on the Jetson board to capture the source images\. However, we strongly recommend you start with static images first\. 

1. Edit the configuration of the Lambda function\. Follow the procedure in [Step 4: Add the Lambda function to the Greengrass group](#ml-console-dlc-config-lambda)\.
**Note**  
 We recommend that you run your Lambda function without containerization unless your business case requires it\. This helps enable access to your device GPU and camera without configuring device resources\. If you run without containerization, you must also grant root access to your AWS IoT Greengrass Lambda functions\. 

   1. **To run without containerization:**
      + For **Run as**, choose **Another user ID/group ID**\. For **UID**, enter **0**\. For **GUID**, enter **0**\.

        This allows your Lambda function to run as root\. For more information about running as root, see [Setting the default access identity for Lambda functions in a group](lambda-group-config.md#lambda-access-identity-groupsettings)\.
**Tip**  
You also must update your `config.json` file to grant root access to your Lambda function\. For the procedure, see [Running a Lambda function as root](lambda-group-config.md#lambda-running-as-root)\.
      + For **Containerization**, choose **No container**\.

        For more information about running without containerization, see [Considerations when choosing Lambda function containerization](lambda-group-config.md#lambda-containerization-considerations)\.
      + Increase the **Timeout** value to 5 minutes\. This ensures that the request does not time out too early\. It takes a few minutes after setup to run inference\.
      +  For **Lambda lifecycle**, choose **Make this function long\-lived and keep it running indefinitely**\. 
      +  For **Read access to /sys directory**, choose **Enable**\. 

   1.  **To run in containerized mode instead:** 
**Note**  
We do not recommend running in containerized mode unless your business case requires it\.
      +  Increase the **Memory limit** value\. To use the provided model in GPU mode, use at least 2000 MB\. 
      + Increase the **Timeout** value to 5 minutes\. This ensures that the request does not time out too early\. It takes a few minutes after setup to run inference\.
      +  For **Lambda lifecycle**, choose **Make this function long\-lived and keep it running indefinitely**\. 
      +  For **Read access to /sys directory**, choose **Enable**\. 

1.  Add your Neo\-optimized model resource to the group\. Upload the model resources in the `resnet18` directory of the sample package you unzipped in [Step 3: Create an inference Lambda function](#ml-console-dlc-create-lambda)\. This directory contains precompiled model artifacts for an image classification model trained with Resnet\-18\. Follow the procedure in [Step 5: Add a SageMaker Neo\-optimized model resource to the Greengrass group](#ml-console-dlc-add-resources) with the following updates\. 
   + Compress the files inside the `resnet18` directory into a file named `resnet18.zip`\.
   + On the **Create a machine learning resource** page, for **Resource name**, enter **resnet18\_model**\.
   + Upload the `resnet18.zip` file\.

1. **If running in containerized mode**, add the required local device resources to grant access to your device GPU\. 
**Note**  
 If you run in non\-containerized mode, AWS IoT Greengrass can access your device GPU without configuring device resources\. 

   1. On the group configuration page, choose **Resources**\.  
![\[The group configuration page with Resources highlighted.\]](http://docs.aws.amazon.com/greengrass/latest/developerguide/images/console-group-resources.png)

   1. On the **Local** tab, choose **Add a local resource**\.

   1. Define each resource:
      + For **Resource name** and **Device path**, use the values in the following table\. Create one device resource for each row in the table\.
      + For **Resource type**, choose **Device**\.
      + For **Group owner file access permission**, choose **Automatically add OS group permissions of the Linux group that owns the resource**\.
      + For **Lambda function affiliations**, grant **Read and write access** to your Lambda function\.

             
[\[See the AWS documentation website for more details\]](http://docs.aws.amazon.com/greengrass/latest/developerguide/ml-dlc-console.html)

1. **If running in containerized mode**, add the following local volume resource to grant access to your device camera\. Follow the procedure in [Step 5: Add a SageMaker Neo\-optimized model resource to the Greengrass group](#ml-console-dlc-add-resources)\.
**Note**  
 If you run in non\-containerized mode, AWS IoT Greengrass can access your device camera without configuring device resources\. 
   + For **Resource type**, choose **Volume**\.
   + For **Group owner file access permission**, choose **Automatically add OS group permissions of the Linux group that owns the resource**\.
   + For **Lambda function affiliations**, grant **Read and write access** to your Lambda function\.

          
[\[See the AWS documentation website for more details\]](http://docs.aws.amazon.com/greengrass/latest/developerguide/ml-dlc-console.html)

1.  Update your group subscriptions to use the correct directory\. Follow the procedure in [Step 7: Add subscriptions to the Greengrass group](#ml-console-dlc-add-subscription) with the following updates\. 
   + For your first topic filter, enter **/resnet\-18/predictions**\.
   + For your second topic filter, enter **/resnet\-18/test**\.

1.  Update your test subscriptions to use the correct directory\. Follow the procedure in [Test the inference example](#ml-console-dlc-test-app) with the following updates\. 
   +  For **Subscriptions**, choose **Subscribe to a Topic**\. For **Subscription topic**, enter **/resnet\-18/predictions**\. 
   +  On the `/resnet-18/predictions` page, specify the `/resnet-18/test` topic to publish to\. 

## Troubleshooting AWS IoT Greengrass ML inference<a name="ml-inference-troubleshooting"></a>

If the test is not successful, you can try the following troubleshooting steps\. Run the commands in your Raspberry Pi terminal\.

### Check error logs<a name="troubleshooting-check-logs"></a>

1. <a name="root-access-logs"></a>Switch to the root user and navigate to the `log` directory\. Access to AWS IoT Greengrass logs requires root permissions\.

   ```
   sudo su
   cd /greengrass/ggc/var/log
   ```

1. Check `runtime.log` for any errors\. 

   ```
   cat system/runtime.log | grep 'ERROR'
   ```

   You can also look in your user\-defined Lambda function log for any errors: 

   ```
   cat user/your-region/your-account-id/lambda-function-name.log | grep 'ERROR'
   ```

   For more information, see [Troubleshooting with logs](gg-troubleshooting.md#troubleshooting-logs)\.

 

### Verify the Lambda function is successfully deployed<a name="troubleshooting-check-lambda"></a>

1.  List the contents of the deployed Lambda in the `/lambda` directory\. Replace the placeholder values before you run the command\. 

   ```
   cd /greengrass/ggc/deployment/lambda/arn:aws:lambda:region:account:function:function-name:function-version
   ls -la
   ```

1.  Verify that the directory contains the same content as the `optimizedImageClassification.zip` deployment package that you uploaded in [Step 3: Create an inference Lambda function](#ml-console-dlc-create-lambda)\. 

    Make sure that the `.py` files and dependencies are in the root of the directory\. 

 

### Verify the inference model is successfully deployed<a name="troubleshooting-check-model"></a>

1. Find the process identification number \(PID\) of the Lambda runtime process:

   ```
   ps aux | grep lambda-function-name
   ```

   In the output, the PID appears in the second column of the line for the Lambda runtime process\.

1.  Enter the Lambda runtime namespace\. Be sure to replace the placeholder *pid* value before you run the command\. 
**Note**  
This directory and its contents are in the Lambda runtime namespace, so they aren't visible in a regular Linux namespace\.

   ```
   sudo nsenter -t pid -m /bin/bash
   ```

1. List the contents of the local directory that you specified for the ML resource\.
**Note**  
 If your ML resource path is something other than `ml_model`, you must substitute that here\. 

   ```
   cd /ml_model
   ls -ls
   ```

   You should see the following files:

   ```
       56 -rw-r--r-- 1 ggc_user ggc_group     56703 Oct 29 20:07 model.json
   196152 -rw-r--r-- 1 ggc_user ggc_group 200855043 Oct 29 20:08 model.params
      256 -rw-r--r-- 1 ggc_user ggc_group    261848 Oct 29 20:07 model.so
       32 -rw-r--r-- 1 ggc_user ggc_group     30564 Oct 29 20:08 synset.txt
   ```

 

### Lambda function cannot find `/dev/dri/renderD128`<a name="troubleshooting-atom-config"></a>

 This can occur if OpenCL cannot connect to the GPU devices it needs\. You must create device resources for the necessary devices for your Lambda function\. 

## Next steps<a name="next-dlc-steps"></a>

 Next, explore other optimized models\. For information, see the [SageMaker Neo documentation](https://docs.aws.amazon.com/sagemaker/latest/dg/neo.html)\. 