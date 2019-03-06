# How to Configure Optimized Machine Learning Inference Using the AWS Management Console<a name="ml-dlc-console"></a>

To follow the steps in this tutorial, you must be using AWS IoT Greengrass Core v1\.6 or later\.

 You can use the Amazon SageMaker Neo deep learning compiler to optimize the prediction efficiency of native machine learning inference models in many frameworks\. You can then download the optimized model and install the Amazon SageMaker Neo deep learning runtime and deploy them to your AWS IoT Greengrass devices for faster inference\. 

This tutorial describes how to use the AWS Management Console to configure a Greengrass group to run a Lambda inference example that recognizes images from a camera locally, without sending data to the cloud\. The inference example accesses the camera module on a Raspberry Pi\. In this tutorial, you download a prepackaged model that is trained by Resnet\-50 and optimized in the Neo deep learning compiler\. You then use the model to perform local image classification on your AWS IoT Greengrass device\. 

The tutorial contains the following high\-level steps:

1. [Configure the Rasberry Pi](#config-raspberry-pi-dlc)\.

1. [Install the Amazon SageMaker Neo deep learning runtime](#install-dlr)\.

1. [Create an inference Lambda function](#ml-console-dlc-create-lambda)\.

1. [Add the Lambda function to the group](#ml-console-dlc-config-lambda)\.

1. [Add the Amazon SageMaker Neo optimized model resource to the group](#ml-console-dlc-add-resources)\.

1. [Add the camera device resource to the group](#ml-console-dlc-add-cam-resource)\.

1. [Add subscriptions to the group](#ml-console-dlc-add-subscription)\.

1. [Deploy the group](#ml-console-dlc-deploy-group)\.

## Prerequisites<a name="ml-inference-prerequisites"></a>

 To complete this tutorial, you need: 
+  [Raspberry Pi 3 Model B](https://www.raspberrypi.org/products/raspberry-pi-3-model-b/#buy-now-modal) set up and configured for use with AWS IoT Greengrass\. To learn how to set up your Raspberry Pi with AWS IoT Greengrass, see [Module 1](https://docs.aws.amazon.com/greengrass/latest/developerguide/module1.html) and [Module 2](https://docs.aws.amazon.com/greengrass/latest/developerguide/module2.html) of [Getting Started with AWS IoT Greengrass](gg-gs.md)\. 
+  [Raspberry Pi Camera Module V2 \- 8 Megapixel, 1080p](https://www.amazon.com/Raspberry-Pi-Camera-Module-Megapixel/dp/B01ER2SKFS)\. To learn how to set up the camera, see [Connecting the camera](https://www.raspberrypi.org/documentation/usage/camera/) in the Raspberry Pi documentation\. 
+  A Greengrass group and a Greengrass core\. To learn how to create a Greengrass group or core, see [Getting Started with AWS IoT Greengrass](gg-gs.md)\. 

**Note**  
 This tutorial uses a Raspberry Pi, but AWS IoT Greengrass supports other platforms, such as [Intel Atom](#atom-lambda-dlc-config) and [NVIDIA Jetson TX2](#jetson-lambda-dlc-config)\. 

## Step 1: Configure the Raspberry Pi<a name="config-raspberry-pi-dlc"></a>

 In this step, you install updates to the Raspbian operating system, install the camera module software and Python dependencies, and enable the camera interface\. 

Run the following commands in your Raspberry Pi terminal\.

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

   This opens a preview window on the Raspberry Pi, saves a picture named `test.jpg` to your current directory, and displays information about the camera in the Raspberry Pi terminal\.

## Step 2: Install the Amazon SageMaker Neo deep learning runtime<a name="install-dlr"></a>

 In this step, you download the Neo deep learning runtime and install it onto your Raspberry Pi\. 

1. On your computer, open the [AWS IoT Core console](https://console.aws.amazon.com/iotv2)\.

1. In the navigation pane, choose **Software**\.  
![\[The AWS IoT Core console with Software highlighted.\]](http://docs.aws.amazon.com/greengrass/latest/developerguide/images/console-software.png)

1. In the **Machine learning inference** section, for ** Runtimes and precompiled framework libraries**, choose **Configure download**\.

1.  On the **Machine learning inference** page, under **Software configurations**, for Deep Learning Runtime Raspberry Pi version 1\.0\.0, choose **Download**\. 

1. Transfer the downloaded `dlr-1.0-py2-armv7l.tar.gz` file from your computer to your Raspberry Pi\. You can also use the following scp command with a path to save your file, such as `/home/pi/`:

   ```
   scp dlr-1.0-py2-armv7l.tar.gz pi@your-device-ip-address:path-to-save-file
   ```

1.  Use the following commands to remotely sign in to your Raspberry Pi and extract the installer files\. 

   ```
   ssh pi@your-device-ip-address
   ​cd path-to-save-file
   tar -xvzf dlr-1.0-py2-armv7l.tar.gz
   ```

1.  Install the Neo deep learning runtime\. 

   ```
   cd dlr-1.0-py2-armv7l/
   chmod 755 install-dlr.sh
   sudo ./install-dlr.sh
   ```

    This package contains an `examples` directory that contains several files you use to run this tutorial\. This directory also contains version 1\.2\.0 of the AWS IoT Greengrass Core SDK for Python\. You can also download the latest version of the SDK in the AWS IoT Core console\. 

## Step 3: Create an Inference Lambda Function<a name="ml-console-dlc-create-lambda"></a>

 In this step, you create a deployment package and a Lambda function that is configured to use the deployment package\. Then, you publish a function version and create an alias\. 

1.  On your computer, unzip the downloaded `dlr-1.0-py2-armv7l.tar.gz` file you previously copied to your Raspberry Pi\. 

   ```
   cd path-to-downloaded-runtime
   tar -xvzf dlr-1.0-py2-armv7l.tar.gz
   ```

1.  The resulting `dlr-1.0-py2-armv7l` directory contains an `examples` folder containing files including `inference.py`, the example code we use in this tutorial for inference\. You can view this code as a usage example to create your own inference code\. 

    Compress the files in the `examples` folder into a file named `optimizedImageClassification.zip`\. 
**Note**  
 When you create the \.zip file, verify that the \.py ﬁles and dependencies are in the root of the directory\. 

   ```
   cd path-to-downloaded-runtime/dlr-1.0-py2-armv7l/examples
   zip -r optimizedImageClassification.zip .
   ```

    This `.zip` file is your deployment package\. This package contains the function code and dependencies, including the code example that invokes the Neo deep learning runtime Python APIs to perform inference with the Neo deep learning compiler models\. You upload this deployment package later\. 

1.  Now, create the Lambda function\. 

    In the AWS IoT Core console, in the navigation pane, choose **Greengrass**, and then choose **Groups**\.   
![\[The navigation pane in the AWS IoT Core console with Groups highlighted.\]](http://docs.aws.amazon.com/greengrass/latest/developerguide/images/console-groups.png)

1. Choose the Greengrass group where you want to add the Lambda function\.

1. On the group configuration page, choose **Lambdas**, and then choose **Add Lambda**\.  
![\[The group page with Lambdas and Add Lambda highlighted.\]](http://docs.aws.amazon.com/greengrass/latest/developerguide/images/console-group-lambdas.png)

1.  On the **Add a Lambda to your Greengrass Group** page, choose **Create new Lambda**\. This opens the AWS Lambda console\.   
![\[The Add a Lambda to your Greengrass Group page with Create new Lambda highlighted.\]](http://docs.aws.amazon.com/greengrass/latest/developerguide/images/console-group-lambdas-new-lambda.png)

1. Choose **Author from scratch** and use the following values to create your function:    
[\[See the AWS documentation website for more details\]](http://docs.aws.amazon.com/greengrass/latest/developerguide/ml-dlc-console.html)  
![\[The Author from Scratch section of the Create function page.\]](http://docs.aws.amazon.com/greengrass/latest/developerguide/images/ml-dlc-inference/gg-dlr-lambda-creation.png)

1.  Choose **Create function**\. 

 

Now, upload your Lambda function deployment package and register the handler\.

1.  On the **Configuration** tab for the `optimizedImageClassification` function, for **Function code**, use the following values:     
[\[See the AWS documentation website for more details\]](http://docs.aws.amazon.com/greengrass/latest/developerguide/ml-dlc-console.html)

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
[\[See the AWS documentation website for more details\]](http://docs.aws.amazon.com/greengrass/latest/developerguide/ml-dlc-console.html)
**Note**  
AWS IoT Greengrass doesn't support Lambda aliases for **$LATEST** versions\.

1. Choose **Create**\.

   Now, add the Lambda function to your Greengrass group\.

## Step 4: Add the Lambda Function to the Greengrass Group<a name="ml-console-dlc-config-lambda"></a>

In this step, you add the Lambda function to the group and then configure its lifecycle\.

First, add the Lambda function to your Greengrass group\.

1.  On the **Add a Lambda to your Greengrass Group** page, choose **Use existing Lambda**\.   
![\[The Add a Lambda to your Greengrass Group page with Use existing Lambda highlighted.\]](http://docs.aws.amazon.com/greengrass/latest/developerguide/images/console-group-lambdas-existing-lambda.png)

1.  Choose **optimizedImageClassification**, and then choose **Next**\. 

1. On the **Select a Lambda version** page, choose **Alias:mlTestOpt**, and then choose **Finish**\.

 

Next, configure the lifecycle of the Lambda function\.

1. On the **Lambdas** page, choose the **optimizedImageClassification** Lambda function\.  
![\[The Lambdas page with the optimizedImageClassification Lambda function highlighted.\]](http://docs.aws.amazon.com/greengrass/latest/developerguide/images/ml-dlc-inference/gg-ml2-lambda-page.png)

1. On the **optimizedImageClassification** configuration page, choose **Edit**\.

1. On the **Group\-specific Lambda configuration** page, use the following values:    
[\[See the AWS documentation website for more details\]](http://docs.aws.amazon.com/greengrass/latest/developerguide/ml-dlc-console.html)

   For more information, see [Lifecycle Configuration for Greengrass Lambda Functions](lambda-functions.md#lambda-lifecycle)\.

1.  Choose **Update**\.

## Step 5: Add Amazon SageMaker Neo Optimized Model Resource to the Greengrass Group<a name="ml-console-dlc-add-resources"></a>

 In this step, you create a resource for the optimized ML inference model and upload it to an Amazon S3 bucket\. Then you locate the Amazon S3 uploaded model in the AWS IoT Greengrass console and affiliate the newly created resource with the Lambda function\. This makes it possible for the function to access its resources on the core device\. 

1.  On your computer, navigate to the Neo deep learning runtime installer package that you unpacked earlier\. Navigate to the `resnet50` directory\. 

   ```
   cd path-to-downloaded-runtime/dlr-1.0-py2-armv7l/models/resnet50
   ```

    This directory contains precompiled model artifacts for an image classification model trained with Resnet\-50\. Compress the files inside the `resnet50` directory to create `resnet50.zip`\. 

   ```
   zip -r resnet50.zip .
   ```

1.  On the group configuration page for your AWS IoT Greengrass group, choose **Resources**\. Navigate to the **Machine Learning** section and choose **Add machine learning resource**\. On the **Create a machine learning resource** page, for **Resource name**, type **resnet50\_model**\.  
![\[The Add Machine Learning Model page with updated properties.\]](http://docs.aws.amazon.com/greengrass/latest/developerguide/images/ml-dlc-inference/gg-ml2-add-ml-model.png)

1. For **Model source**, choose **Upload a model in S3**\.

1.  Under **Model from S3**, choose **Select**\. 
**Note**  
 Currently, Amazon SageMaker models are automatically stored in Amazon S3 when optimized\. You can find your optimized model in your Amazon S3 bucket using this option\. For more information about model optimization in Amazon SageMaker, please refer to the [Amazon SageMaker Neo documentation](https://docs.aws.amazon.com/sagemaker/latest/dg/neo.html)\. 

1.  Choose **Upload a model**\. This opens up a new tab to the Amazon S3 console\. 

1.  In the Amazon S3 console tab, upload your zip file to an Amazon S3 bucket\. For information, see [ How Do I Upload Files and Folders to an S3 Bucket? ](https://docs.aws.amazon.com/AmazonS3/latest/user-guide/upload-objects.html) 
**Note**  
 Your bucket name must contain the string **greengrass** in order for the bucket to be accessible\. Choose a unique name \(such as **greengrass\-dlr\-bucket\-*user\-id*\-*epoch\-time***\)\. Don't use a period \(`.`\) in the bucket name\. 

1.  In the AWS IoT Greengrass console tab, locate and choose your Amazon S3 bucket\. Locate your uploaded `resnet50.zip` file, and choose **Select**\. You may need to refresh the page to update the list of available buckets and files\. 

1.  In **Local path**, enter **/ml\_model**\.   
![\[The updated local path.\]](http://docs.aws.amazon.com/greengrass/latest/developerguide/images/ml-dlc-inference/local-path.png)

    This is the destination for the local model in the Lambda runtime namespace\. When you deploy the group, AWS IoT Greengrass retrieves the source model package and then extracts the contents to the specified directory\. 
**Note**  
 We strongly recommend that you use the exact path provided for your local path\. Using a different local model destination path in this step causes some troubleshooting commands provided in this tutorial to be inaccurate\. If you use a different path, you must set up a `MODEL_PATH` environment variable that uses the exact path you provide here\. For information about environment variables, see [AWS Lambda Environment Variables](https://docs.aws.amazon.com/lambda/latest/dg/env_variables.html)\. 

1. Under **Lambda function affiliations**, choose **Select**\.

1. Choose **optimizedImageClassification**, choose **Read\-only access**, and then choose **Done**\.

1.  Choose **Save**\. 

## Step 6: Add Your Camera Device Resource to the Greengrass Group<a name="ml-console-dlc-add-cam-resource"></a>

 In this step, you create a resource for the camera module and affiliate it with the Lambda function, allowing the resource to be accessible on the AWS IoT Greengrass core\. 

1. On the group configuration page, choose **Resources**\.  
![\[The group configuration page with Resources highlighted.\]](http://docs.aws.amazon.com/greengrass/latest/developerguide/images/ml-dlc-inference/gg-ml2-group-config-resources.png)

1. On the **Local Resources** tab, choose **Add local resource**\.

1. On the **Create a local resource** page, use the following values:    
[\[See the AWS documentation website for more details\]](http://docs.aws.amazon.com/greengrass/latest/developerguide/ml-dlc-console.html)

   The **Device path** is the local absolute path of the device resource\. This path can only refer to a character device or block device under /dev\.

   The **Group owner file access permission** option lets you grant additional file access permissions to the Lambda process\. For more information, see [Group Owner File Access Permission](access-local-resources.md#lra-group-owner)\.  
![\[The Create a local resource page with edited resource properties.\]](http://docs.aws.amazon.com/greengrass/latest/developerguide/images/ml-inference/local-resource-vcsm.png)

1. Under **Lambda function affiliations**, choose **Select**\.

1. Choose **optimizedImageClassification**, choose **Read and write access**, and then choose **Done**\.  
![\[Lambda function affiliation properties with Done highlighted.\]](http://docs.aws.amazon.com/greengrass/latest/developerguide/images/ml-dlc-inference/gg-ml2-local-resource-vcsm.png)

   Next, you add a local device resource for the camera interface\.

1. At the bottom of the page, choose **Add another resource**\.

1. On the **Create a local resource** page, use the following values:    
[\[See the AWS documentation website for more details\]](http://docs.aws.amazon.com/greengrass/latest/developerguide/ml-dlc-console.html)  
![\[The Create a local resource page with edited resource properties.\]](http://docs.aws.amazon.com/greengrass/latest/developerguide/images/ml-inference/local-resource-vchiq.png)

1. Under **Lambda function affiliations**, choose **Select**\.

1.  Choose **optimizedImageClassification**, choose **Read and write access**, and then choose **Done**\. 

1.  Choose **Save**\. 

## Step 7: Add Subscriptions to the Greengrass Group<a name="ml-console-dlc-add-subscription"></a>

In this step, you add subscriptions to the group\. These subscriptions enable the Lambda function to send prediction results to AWS IoT by publishing to an MQTT topic\.

1. On the group configuration page, choose **Subscriptions**, and then choose **Add Subscription**\.  
![\[The group page with Subscriptions and Add Subscription highlighted.\]](http://docs.aws.amazon.com/greengrass/latest/developerguide/images/console-group-subscriptions.png)

1. On the **Select your source and target** page, configure the source and target, as follows:

   1. In **Select a source**, choose **Lambdas**, and then choose **optimizedImageClassification**\.

   1. In **Select a target**, choose **Services**, and then choose **IoT Cloud**\.

   1. Choose **Next**\.  
![\[The Select your source and target page with Next highlighted.\]](http://docs.aws.amazon.com/greengrass/latest/developerguide/images/ml-dlc-inference/gg-ml2-subsc-1.png)

1.  On the **Filter your data with a topic** page, in **Optional topic filter**, type **/resnet\-50/predictions**, and then choose **Next**\.   
![\[The Filter your data with a topic page with Next highlighted.\]](http://docs.aws.amazon.com/greengrass/latest/developerguide/images/ml-dlc-inference/gg-ml2-topic-filter-s1.png)

1. Choose **Finish**\.

1.  Add a second subscription\. On the **Select your source and target** page, configure the source and target, as follows: 

   1. In **Select a source**, choose **Services**, and then choose **IoT Cloud**\.

   1. In **Select a target**, choose **Lambdas**, and then choose **optimizedImageClassification**\.

   1. Choose **Next**\.

1.  On the **Filter your data with a topic** page, in **Optional topic filter**, type **/resnet\-50/test**, and then choose **Next**\.

1. Choose **Finish**\.

## Step 8: Deploy the Greengrass Group<a name="ml-console-dlc-deploy-group"></a>

In this step, you deploy the current version of the group definition to the Greengrass core device\. The definition contains the Lambda function, resources, and subscription configurations that you added\.

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
If prompted, grant permission to create the AWS IoT Greengrass service role on your behalf, which allows AWS IoT Greengrass to access other AWS services\. You need to do this only one time per account\.

    The **Deployments** page shows the deployment time stamp, version ID, and status\. When completed, the status displayed for the deployment should be **Successfully completed**\.   
![\[The Deployments page with a successful deployment status highlighted.\]](http://docs.aws.amazon.com/greengrass/latest/developerguide/images/ml-dlc-inference/gg-ml2-successful-deployment.png)

   For troubleshooting help, see [Troubleshooting AWS IoT Greengrass](gg-troubleshooting.md)\.

## Test the Inference Example<a name="test-app"></a>

Now you can verify whether the deployment is configured correctly\. To test, you subscribe to the `/resnet-50/predictions` topic and publish any message to the `/resnet-50/test` topic\. This triggers the Lambda function to take a photo with your Raspberry Pi and perform inference on the image it captures\. 

**Note**  
If a monitor is attached to the Raspberry Pi, the live camera feed is displayed in a preview window\.

1. On the AWS IoT Core console home page, choose **Test**\.  
![\[The navigation pane in the AWS IoT Core console with Test highlighted.\]](http://docs.aws.amazon.com/greengrass/latest/developerguide/images/console-test.png)

1.  For **Subscriptions**, choose **Subscribe to a Topic**\. Use the following values\. Leave the remaining options at their defaults:     
[\[See the AWS documentation website for more details\]](http://docs.aws.amazon.com/greengrass/latest/developerguide/ml-dlc-console.html)

1. Choose **Subscribe to topic**\.

1.  In the `/resnet-50/predictions` page, specify the `/resnet-50/test` topic to publish to\. Choose **Publish to topic**\. 

1.  If the test is successful, the published message causes the Raspberry Pi camera to capture an image\. A message from the Lambda function appears at the bottom of the page\. This message contains the prediction result of the image, using the format: predicted class name, probability, and peak memory usage\.   
![\[The Subscriptions page showing test results with message data.\]](http://docs.aws.amazon.com/greengrass/latest/developerguide/images/ml-dlc-inference/prediction-results.png)

## Configuring an Intel Atom<a name="atom-lambda-dlc-config"></a>

 To run this tutorial on an Intel Atom device, you provide source images and configure the Lambda function\. In order to use the GPU for inference, you must have OpenCL version 1\.0 or later installed on your device\. You must also add local device resources\. 

1. Download static PNG or JPG images for the Lambda function to use for image classification\. The example works best with small image files\. 

   Save your image files in the directory that contains the `inference.py` file \(or in a subdirectory of this directory\)\. This is in the Lambda function deployment package that you upload in [Step 3: Create an Inference Lambda Function](#ml-console-dlc-create-lambda)\.
**Note**  
 If using Amazon DeepLens, you can choose to instead use the onboard camera or mount your own camera to capture images and perform inference on them\. However, we strongly recommend you attempt the example with static images first\. 

1.  Edit the configuration of the Lambda function\. Follow the procedure in [Step 4: Add the Lambda Function to the Greengrass Group](#ml-console-dlc-config-lambda)\. 

   1.  Increase the **Memory limit** value to 3000 MB\. 

   1. Increase the **Timeout** value to 2 minutes\. This will ensure that the request does not time out too early\. The first time inference is run after completing setup on the Intel Atom will take a few moments\.

   1.  Choose **Enable** for the **Read access to /sys directory** option\. 

   1.  For **Lambda lifecycle**, choose **Make this function long\-lived and keep it running indefinitely**\. 

1. Add the following local device resource\.

   1.  On the Navigation pane, choose **Resources** and **Add a local resource**\.   
![\[The Add a local resource page.\]](http://docs.aws.amazon.com/greengrass/latest/developerguide/images/ml-dlc-inference/gg-ml2-add-local-resource.png)

   1. For the resource:
      + For **Resource type**, choose **Device**\.
      + For **Group owner file access permission**, choose **Automatically add OS group permissions of the Linux group that owns the resource**\.
      + For **Lambda function affiliations**, grant **Read and write access** to your Lambda function\.

             
[\[See the AWS documentation website for more details\]](http://docs.aws.amazon.com/greengrass/latest/developerguide/ml-dlc-console.html)

## Configuring an NVIDIA Jetson TX2<a name="jetson-lambda-dlc-config"></a>

 To run this tutorial on an NVIDIA Jetson TX2, you provide source images and configure the Lambda function\. In order to use the GPU for inference, you must install CUDA 9\.0 and cuDNN 7\.0 on your device when you image your board with Jetpack 3\.3\. You must also add local device resources\. 

 To learn how to configure your Jetson so you can install the AWS IoT Greengrass core software, see [Setting Up Other Devices](module1.md#setup-filter.other)\.

1. Download static PNG or JPG images for the Lambda function to use for image classification\. The example works best with small image files\. 

   Save your image files in the directory that contains the `inference.py` file \(or in a subdirectory of this directory\)\. This is in the Lambda function deployment package that you upload in [Step 3: Create an Inference Lambda Function](#ml-console-dlc-create-lambda)\.
**Note**  
 You can instead choose to instrument a camera on the Jetson board to capture the source images\. However, we strongly recommend you attempt the example with static images first\. 

1. Edit the configuration of the Lambda function\. Follow the procedure in [Step 4: Add the Lambda Function to the Greengrass Group](#ml-console-dlc-config-lambda)\.

   1.  Increase the **Memory limit** value\. To use the provided model in GPU mode, use 2048 MB\. 

   1. Increase the **Timeout** value to 5 minutes\. This will ensure that the request does not time out too early\. The first time inference is run after completing setup on the Jetson will take a few moments\.

   1.  For **Lambda lifecycle**, choose **Make this function long\-lived and keep it running indefinitely**\. 

   1.  For **Read access to /sys directory**, choose **Enable**\. 

1. Add the following local device resources\. 

   1.  On the Navigation pane, choose **Resources** and **Add a local resource**\.   
![\[The Add a local resource page.\]](http://docs.aws.amazon.com/greengrass/latest/developerguide/images/ml-dlc-inference/gg-ml2-add-local-resource.png)

   1. For each resource:
      + For **Resource type**, choose **Device**\.
      + For **Group owner file access permission**, choose **Automatically add OS group permissions of the Linux group that owns the resource**\.
      + For **Lambda function affiliations**, grant **Read and write access** to your Lambda function\.

             
[\[See the AWS documentation website for more details\]](http://docs.aws.amazon.com/greengrass/latest/developerguide/ml-dlc-console.html)

**Note**  
 The first inference after completing set up may take a few moments to complete\. 

## Troubleshooting AWS IoT Greengrass ML Inference<a name="ml-inference-troubleshooting"></a>

If the test is not successful, you can try the following troubleshooting steps\. Run the commands in your Raspberry Pi terminal\.

### Check Error Logs<a name="troubleshooting-check-logs"></a>

1. Switch to the root user\.

   ```
   sudo su
   ```

1. Navigate to the `/log` directory\.

   ```
   cd /greengrass/ggc/var/log
   ```

1. Check `runtime.log` for any errors\. 

   ```
   cat system/runtime.log | grep 'ERROR'
   ```

   You can also look in your user lambda log for any errors: 

   ```
   cat user/your-region/your-account-id/lambda-function-name.log | grep 'ERROR'
   ```

   For more information, see [Troubleshooting with Logs](gg-troubleshooting.md#troubleshooting-logs)\.

 

### Verify That the Lambda Function Is Successfully Deployed<a name="troubleshooting-check-lambda"></a>

1.  List the contents of the deployed Lambda in the `/lambda` directory\. Replace the placeholder values before you run the command\. 

   ```
   cd /greengrass/ggc/deployment/lambda/arn:aws:lambda:region:account:function:function-name:function-version
   ls -la
   ```

1.  Verify that the directory contains the same content as the `optimizedImageClassification.zip` deployment package that you uploaded in [Step 3: Create an Inference Lambda Function](#ml-console-dlc-create-lambda)\. 

    Make sure that the `.py` files and dependencies are in the root of the directory\. 

 

### Verify That the Inference Model Is Successfully Deployed<a name="troubleshooting-check-model"></a>

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

 This can occur if OpenCL cannot connect to the GPU devices it needs\. You must create device resources for the necessary devices for your lambda function\. 

## Next Steps<a name="next-dlc-steps"></a>

 Next, explore other optimized models\. For information, refer to the [Amazon SageMaker Neo documentation](https://docs.aws.amazon.com/sagemaker/latest/dg/neo.html)\. 