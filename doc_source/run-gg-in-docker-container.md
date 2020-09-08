# Running AWS IoT Greengrass in a Docker container<a name="run-gg-in-docker-container"></a>

AWS IoT Greengrass can be configured to run in a [Docker](https://www.docker.com/) container\.

You can download a Dockerfile [through Amazon CloudFront](what-is-gg.md#gg-docker-download) that has the AWS IoT Greengrass Core software and dependencies installed\. To modify the Docker image to run on different platform architectures or reduce the size of the Docker image, see the `README` file in the Docker package download\.

To help you get started experimenting with AWS IoT Greengrass, AWS also provides prebuilt Docker images that have the AWS IoT Greengrass Core software and dependencies installed\. You can download an image from [Docker Hub](https://hub.docker.com/r/amazon/aws-iot-greengrass) or [Amazon Elastic Container Registry](https://docs.aws.amazon.com/AmazonECR/latest/userguide/what-is-ecr.html) \(Amazon ECR\)\. These prebuilt images use Amazon Linux 2 \(x86\_64\) and Alpine Linux \(x86\_64, Armv7l, or AArch64\) base images\.

This topic describes how to download the `latest` AWS IoT Greengrass Docker image from Amazon ECR and run it on a Windows, macOS, or Linux \(x86\_64\) platform\. The topic contains the following steps:

1. [Get the AWS IoT Greengrass container image from Amazon ECR](#docker-pull-image)

1. [Create and configure the Greengrass group and core](#docker-config-gg)

1. [Run AWS IoT Greengrass locally](#docker-run-gg)

1. [Configure "No container" containerization for the group](#docker-no-container)

1. [Deploy Lambda functions to the Docker container](#docker-add-lambdas)

1. [\(Optional\) Deploy devices that interact with Greengrass in the Docker container](#docker-add-devices)

The following features aren't supported when you run AWS IoT Greengrass in a Docker container:<a name="docker-image-unsupported-features"></a>
+ [Connectors](connectors.md) that run in **Greengrass container** mode\. To run a connector in a Docker container, the connector must run in **No container** mode\. To find connectors that support **No container** mode, see [AWS\-provided Greengrass connectors](connectors-list.md)\. Some of these connectors have an isolation mode parameter that you must set to **No container**\.
+ [Local device and volume resources](access-local-resources.md)\. Your user\-defined Lambda functions that run in the Docker container must access devices and volumes on the core directly\.

These features aren't supported when the Lambda runtime environment for the Greengrass group is set to [No container](lambda-group-config.md#no-container-mode), which is required to run AWS IoT Greengrass in a Docker container\.

## Prerequisites<a name="docker-image-prerequisites"></a>

Before you start this tutorial, you must do the following\.<a name="docker-image-prereq-list"></a>
+ You must install the following software and versions on your host computer based on the AWS Command Line Interface \(AWS CLI\) version that you choose\.

------
#### [ AWS CLI version 2 ]
  + [Docker](https://docs.docker.com/install/) version 18\.09 or later\. Earlier versions might also work, but we recommend 18\.09 or later\.
  + AWS CLI version 2\.0\.0 or later\.
    + To install the AWS CLI version 2, see [Installing the AWS CLI version 2](https://docs.aws.amazon.com/cli/latest/userguide/install-cliv2.html)\.
    + To configure the AWS CLI, see [Configuring the AWS CLI](https://docs.aws.amazon.com/cli/latest/userguide/cli-chap-configure.html)\.
**Note**  
To upgrade to a later AWS CLI version 2 on a Windows computer, you must repeat the [MSI installation](https://docs.aws.amazon.com/cli/latest/userguide/install-cliv2-windows.html) process\.

------
#### [ AWS CLI version 1 ]
  + [Docker](https://docs.docker.com/install/) version 18\.09 or later\. Earlier versions might also work, but we recommend 18\.09 or later\.
  + [Python](https://www.python.org/downloads/) version 3\.6 or later\.
  + [pip](https://pip.pypa.io/en/stable/installing) version 18\.1 or later\.
  + AWS CLI version 1\.17\.10 or later
    + To install the AWS CLI version 1, see [Installing the AWS CLI version 1](https://docs.aws.amazon.com/cli/latest/userguide/install-cliv1.html)\.
    + To configure the AWS CLI, see [Configuring the AWS CLI](https://docs.aws.amazon.com/cli/latest/userguide/cli-chap-configure.html)\.
    + To upgrade to the latest version of the AWS CLI version 1, run the following command\.

      ```
      pip install awscli --upgrade --user
      ```
**Note**  
If you use the [MSI installation](https://docs.aws.amazon.com/cli/latest/userguide/install-windows.html#msi-on-windows) of the AWS CLI version 1 on Windows, be aware of the following:  
If the AWS CLI version 1 installation fails to install botocore, try using the [Python and pip installation](https://docs.aws.amazon.com/cli/latest/userguide/awscli-install-windows.html#awscli-install-windows-pip)\.
To upgrade to a later AWS CLI version 1, you must repeat the MSI installation process\.

------
+ To access Amazon Elastic Container Registry \(Amazon ECR\) resources, you must grant the following permission\. 
  + Amazon ECR requires users to grant the `ecr:GetAuthorizationToken` permission through an AWS Identity and Access Management \(IAM\) policy before they can authenticate to a registry and push or pull images from an Amazon ECR repository\. For more information, see [Amazon ECR Repository Policy Examples](https://docs.aws.amazon.com/AmazonECR/latest/userguide/repository-policy-examples.html) and [Accessing One Amazon ECR Repository](https://docs.aws.amazon.com/AmazonECR/latest/userguide/security_iam_id-based-policy-examples.html#security_iam_id-based-policy-examples-access-one-bucket) in the *Amazon Elastic Container Registry User Guide*\.

## Step 1: Get the AWS IoT Greengrass container image from Amazon ECR<a name="docker-pull-image"></a>

AWS provides Docker images that have the AWS IoT Greengrass Core software installed\. For steps that show how to pull the `latest` image from Amazon ECR, choose your operating system:

### Pull the container image \(Linux\)<a name="docker-pull-image-linux"></a>

Run the following commands in your computer terminal\.

1. <a name="docker-get-login"></a>Log in to the AWS IoT Greengrass registry in Amazon ECR\.

   ```
   aws ecr get-login-password --region  us-west-2 | docker login --username AWS --password-stdin https://216483018798.dkr.ecr.us-west-2.amazonaws.com
   ```

   If successful, the output prints `Login Succeeded`\.

1. <a name="docker-docker-pull"></a>Retrieve the AWS IoT Greengrass container image\.

   ```
   docker pull 216483018798.dkr.ecr.us-west-2.amazonaws.com/aws-iot-greengrass:latest
   ```
**Note**  
The `latest` image contains the latest stable version of the AWS IoT Greengrass Core software installed on an Amazon Linux 2 base image\. You can also pull other images from the repository\. To find all available images, check the **Tags** page on [Docker Hub](https://hub.docker.com/r/amazon/aws-iot-greengrass) or use the aws ecr list\-images command\. For example:  

   ```
   aws ecr list-images --region us-west-2 --registry-id 216483018798 --repository-name aws-iot-greengrass
   ```

1. Enable symlink and hardlink protection\. If you're experimenting with running AWS IoT Greengrass in a container, you can enable the settings for the current boot only\.
**Note**  
You might need to use sudo to run these commands\.
   + To enable the settings for the current boot only:

     ```
     echo 1 > /proc/sys/fs/protected_hardlinks
     echo 1 > /proc/sys/fs/protected_symlinks
     ```
   + To enable the settings to persist across restarts:

     ```
     echo '# AWS Greengrass' >> /etc/sysctl.conf 
     echo 'fs.protected_hardlinks = 1' >> /etc/sysctl.conf 
     echo 'fs.protected_symlinks = 1' >> /etc/sysctl.conf
     
     sysctl -p
     ```

1. <a name="docker-linux-enable-ipv4"></a>Enable IPv4 network forwarding, which is required for AWS IoT Greengrass cloud deployment and MQTT communications to work on Linux\. In the `/etc/sysctl.conf` file, set `net.ipv4.ip_forward` to 1, and then reload `sysctls`\.

   ```
   sudo nano /etc/sysctl.conf
   # set this net.ipv4.ip_forward = 1
   sudo sysctl -p
   ```
**Note**  
You can use the editor of your choice instead of nano\.

### Pull the container image \(macOS\)<a name="docker-pull-image-mac"></a>

Run the following commands in your computer terminal\.

1. <a name="docker-get-login"></a>Log in to the AWS IoT Greengrass registry in Amazon ECR\.

   ```
   aws ecr get-login-password --region  us-west-2 | docker login --username AWS --password-stdin https://216483018798.dkr.ecr.us-west-2.amazonaws.com
   ```

   If successful, the output prints `Login Succeeded`\.

1. <a name="docker-docker-pull"></a>Retrieve the AWS IoT Greengrass container image\.

   ```
   docker pull 216483018798.dkr.ecr.us-west-2.amazonaws.com/aws-iot-greengrass:latest
   ```
**Note**  
The `latest` image contains the latest stable version of the AWS IoT Greengrass Core software installed on an Amazon Linux 2 base image\. You can also pull other images from the repository\. To find all available images, check the **Tags** page on [Docker Hub](https://hub.docker.com/r/amazon/aws-iot-greengrass) or use the aws ecr list\-images command\. For example:  

   ```
   aws ecr list-images --region us-west-2 --registry-id 216483018798 --repository-name aws-iot-greengrass
   ```

### Pull the container image \(Windows\)<a name="docker-pull-image-windows"></a>

Run the following commands in a command prompt\. Before you can use Docker commands on Windows, Docker Desktop must be running\.

1. <a name="docker-get-login"></a>Log in to the AWS IoT Greengrass registry in Amazon ECR\.

   ```
   aws ecr get-login-password --region  us-west-2 | docker login --username AWS --password-stdin https://216483018798.dkr.ecr.us-west-2.amazonaws.com
   ```

   If successful, the output prints `Login Succeeded`\.

1. <a name="docker-docker-pull"></a>Retrieve the AWS IoT Greengrass container image\.

   ```
   docker pull 216483018798.dkr.ecr.us-west-2.amazonaws.com/aws-iot-greengrass:latest
   ```
**Note**  
The `latest` image contains the latest stable version of the AWS IoT Greengrass Core software installed on an Amazon Linux 2 base image\. You can also pull other images from the repository\. To find all available images, check the **Tags** page on [Docker Hub](https://hub.docker.com/r/amazon/aws-iot-greengrass) or use the aws ecr list\-images command\. For example:  

   ```
   aws ecr list-images --region us-west-2 --registry-id 216483018798 --repository-name aws-iot-greengrass
   ```

## Step 2: Create and configure the Greengrass group and core<a name="docker-config-gg"></a>

The Docker image has the AWS IoT Greengrass Core software installed, but you must create a Greengrass group and core\. This includes downloading certificates and the core configuration file\.
+ Follow the steps in [Configure AWS IoT Greengrass on AWS IoT](gg-config.md)\. Skip the step where you download the AWS IoT Greengrass Core software\. The software and its runtime dependencies are already set up in the Docker image\.

## Step 3: Run AWS IoT Greengrass locally<a name="docker-run-gg"></a>

After your group is configured, you're ready to configure and start the core\. For steps that show how to do this, choose your operating system:

### Run Greengrass locally \(Linux\)<a name="docker-run-gg-linux"></a>

Run the following commands in your computer terminal\.

1. <a name="docker-untar-files"></a>Decompress the certificates and configuration file \(that you downloaded when you created your Greengrass group\) into a known location, such as `/tmp`\. For example:

   ```
   tar xvzf hash-setup.tar.gz -C /tmp/
   ```

1. <a name="docker-root-ca"></a>Review [Server Authentication](https://docs.aws.amazon.com/iot/latest/developerguide/server-authentication.html) in the *AWS IoT Developer Guide* and choose the appropriate root CA certificate\. We recommend that you use Amazon Trust Services \(ATS\) endpoints and ATS root CA certificates\.

   Run the following commands to download the root CA certificate to the directory where you decompressed the certificates and configuration file\. Certificates enable your device to connect to AWS IoT over TLS\.

   Replace `/tmp` with the path to the directory\.
**Important**  
Your root CA certificate type must match your endpoint\. Use an ATS root CA certificate with an ATS endpoint \(preferred\) or a VeriSign root CA certificate with a legacy endpoint\. Only some AWS Regions support legacy endpoints\. For more information, see [Service endpoints must match the root CA certificate type](gg-core.md#certificate-endpoints)\.
   + For ATS endpoints \(preferred\), download the appropriate ATS root CA certificate\. The following example downloads `AmazonRootCA1.pem`\.

     ```
     cd /tmp/certs/
     sudo wget -O root.ca.pem https://www.amazontrust.com/repository/AmazonRootCA1.pem
     ```
   + For legacy endpoints, download a VeriSign root CA certificate\. Although legacy endpoints are acceptable for the purposes of this tutorial, we recommend that you use an ATS endpoint and ATS root CA certificate\.

     ```
     cd /tmp/certs/
     sudo wget -O root.ca.pem https://www.websecurity.digicert.com/content/dam/websitesecurity/digitalassets/desktop/pdfs/roots/VeriSign-Class%203-Public-Primary-Certification-Authority-G5.pem
     ```
**Note**  
The `wget -O` parameter is the capital letter O\.

1. <a name="docker-docker-run"></a>Start AWS IoT Greengrass and bind\-mount the certificates and configuration file in the Docker container\.

   Replace `/tmp` with the path where you decompressed your certificates and configuration file\.

   ```
   docker run --rm --init -it --name aws-iot-greengrass \
   --entrypoint /greengrass-entrypoint.sh \
   -v /tmp/certs:/greengrass/certs \
   -v /tmp/config:/greengrass/config \
   -p 8883:8883 \
   216483018798.dkr.ecr.us-west-2.amazonaws.com/aws-iot-greengrass:latest
   ```

   The output should look like this example:

   ```
   Setting up greengrass daemon
   Validating hardlink/softlink protection
   Waiting for up to 30s for Daemon to start
   
   Greengrass successfully started with PID: 10
   ```

### Run Greengrass locally \(macOS\)<a name="docker-run-gg-mac"></a>

Run the following commands in your computer terminal\.

1. <a name="docker-untar-files"></a>Decompress the certificates and configuration file \(that you downloaded when you created your Greengrass group\) into a known location, such as `/tmp`\. For example:

   ```
   tar xvzf hash-setup.tar.gz -C /tmp/
   ```

1. <a name="docker-root-ca"></a>Review [Server Authentication](https://docs.aws.amazon.com/iot/latest/developerguide/server-authentication.html) in the *AWS IoT Developer Guide* and choose the appropriate root CA certificate\. We recommend that you use Amazon Trust Services \(ATS\) endpoints and ATS root CA certificates\.

   Run the following commands to download the root CA certificate to the directory where you decompressed the certificates and configuration file\. Certificates enable your device to connect to AWS IoT over TLS\.

   Replace `/tmp` with the path to the directory\.
**Important**  
Your root CA certificate type must match your endpoint\. Use an ATS root CA certificate with an ATS endpoint \(preferred\) or a VeriSign root CA certificate with a legacy endpoint\. Only some AWS Regions support legacy endpoints\. For more information, see [Service endpoints must match the root CA certificate type](gg-core.md#certificate-endpoints)\.
   + For ATS endpoints \(preferred\), download the appropriate ATS root CA certificate\. The following example downloads `AmazonRootCA1.pem`\.

     ```
     cd /tmp/certs/
     sudo wget -O root.ca.pem https://www.amazontrust.com/repository/AmazonRootCA1.pem
     ```
   + For legacy endpoints, download a VeriSign root CA certificate\. Although legacy endpoints are acceptable for the purposes of this tutorial, we recommend that you use an ATS endpoint and ATS root CA certificate\.

     ```
     cd /tmp/certs/
     sudo wget -O root.ca.pem https://www.websecurity.digicert.com/content/dam/websitesecurity/digitalassets/desktop/pdfs/roots/VeriSign-Class%203-Public-Primary-Certification-Authority-G5.pem
     ```
**Note**  
The `wget -O` parameter is the capital letter O\.

1. <a name="docker-docker-run"></a>Start AWS IoT Greengrass and bind\-mount the certificates and configuration file in the Docker container\.

   Replace `/tmp` with the path where you decompressed your certificates and configuration file\.

   ```
   docker run --rm --init -it --name aws-iot-greengrass \
   --entrypoint /greengrass-entrypoint.sh \
   -v /tmp/certs:/greengrass/certs \
   -v /tmp/config:/greengrass/config \
   -p 8883:8883 \
   216483018798.dkr.ecr.us-west-2.amazonaws.com/aws-iot-greengrass:latest
   ```

   The output should look like this example:

   ```
   Setting up greengrass daemon
   Validating hardlink/softlink protection
   Waiting for up to 30s for Daemon to start
   
   Greengrass successfully started with PID: 10
   ```

### Run Greengrass locally \(Windows\)<a name="docker-run-gg-windows"></a>

1. Use a utility such as WinZip or 7\-Zip to decompress the certificates and configuration file that you downloaded when you created your Greengrass group\. For more information, see the [WinZip](https://www.winzip.com/win/en/tar-file.html) documentation\.

   Locate the downloaded `hash-setup.tar.gz` file on your computer and then decompress the file into `C:\Users\%USERNAME%\Downloads\`\.

1. Review [Server Authentication](https://docs.aws.amazon.com/iot/latest/developerguide/server-authentication.html) in the *AWS IoT Developer Guide* and choose the appropriate root CA certificate\. We recommend that you use Amazon Trust Services \(ATS\) endpoints and ATS root CA certificates\.

   Run the following commands to download the root CA certificate to the directory where you decompressed the certificates and configuration file\. Certificates enable your device to connect to AWS IoT over TLS\.
**Important**  
Your root CA certificate type must match your endpoint\. Use an ATS root CA certificate with an ATS endpoint \(preferred\) or a VeriSign root CA certificate with a legacy endpoint\. Only some AWS Regions support legacy endpoints\. For more information, see [Service endpoints must match the root CA certificate type](gg-core.md#certificate-endpoints)\.
   + For ATS endpoints \(preferred\), download the appropriate ATS root CA certificate\. The following example downloads `AmazonRootCA1.pem`\.
     + If you have [curl](https://curl.haxx.se/download.html) installed, run the following commands in your command prompt\.

       ```
       cd C:\Users\%USERNAME%\Downloads\certs
       curl https://www.amazontrust.com/repository/AmazonRootCA1.pem -o root.ca.pem
       ```
     + Otherwise, in a web browser, open the [Amazon Root CA 1](https://www.amazontrust.com/repository/AmazonRootCA1.pem) certificate\. Save the document as `root.ca.pem` in the `C:\Users\%USERNAME%\Downloads\certs` directory, which contains the decompressed certificates\.
**Note**  
Depending on your browser, save the file directly from the browser or copy the displayed key to the clipboard and save it in Notepad\.
   + For legacy endpoints, download a VeriSign root CA certificate\. Although legacy endpoints are acceptable for the purposes of this tutorial, we recommend that you use an ATS endpoint and ATS root CA certificate\.
     + If you have [curl](https://curl.haxx.se/download.html) installed, run the following commands in your command prompt\.

       ```
       cd C:\Users\%USERNAME%\Downloads\certs
       curl https://www.websecurity.digicert.com/content/dam/websitesecurity/digitalassets/desktop/pdfs/roots/VeriSign-Class%203-Public-Primary-Certification-Authority-G5.pem -o root.ca.pem
       ```
     + Otherwise, in a web browser, open the [VeriSign Class 3 Public Primary G5 root CA certificate](https://www.websecurity.digicert.com/content/dam/websitesecurity/digitalassets/desktop/pdfs/roots/VeriSign-Class%203-Public-Primary-Certification-Authority-G5.pem)\. Save the document as `root.ca.pem` in the `C:\Users\%USERNAME%\Downloads\certs` directory, which contains the decompressed certificates\.
**Note**  
Depending on your browser, save the file directly from the browser or copy the displayed key to the clipboard and save it in Notepad\.

1. Start AWS IoT Greengrass and bind\-mount the certificates and configuration file in the Docker container\. Run the following commands in your command prompt\.

   ```
   docker run --rm --init -it --name aws-iot-greengrass --entrypoint /greengrass-entrypoint.sh -v c:/Users/%USERNAME%/Downloads/certs:/greengrass/certs -v c:/Users/%USERNAME%/Downloads/config:/greengrass/config -p 8883:8883 216483018798.dkr.ecr.us-west-2.amazonaws.com/aws-iot-greengrass:latest
   ```

   When Docker prompts you to share your `C:\` drive with the Docker daemon, allow it to bind\-mount the `C:\` directory inside the Docker container\. For more information, see [Shared drives](https://docs.docker.com/docker-for-windows/#shared-drives) in the Docker documentation\. 

   The output should look like this example:

   ```
   Setting up greengrass daemon
   Validating hardlink/softlink protection
   Waiting for up to 30s for Daemon to start
   
   Greengrass successfully started with PID: 10
   ```

**Note**  
If the container doesn't open the shell and exits immediately, you can debug the issue by bind\-mounting the Greengrass runtime logs when you start the image\. For more information, see [To persist Greengrass runtime logs outside of the Docker container](#debugging-docker-persist-logs)\.

## Step 4: Configure "No container" containerization for the Greengrass group<a name="docker-no-container"></a>

When you run AWS IoT Greengrass in a Docker container, all Lambda functions must run without containerization\. In this step, you set the default containerization for the group to **No container**\. You must do this before you deploy the group for the first time\.

1. <a name="console-gg-groups"></a>In the AWS IoT console, choose **Greengrass**, and then choose **Groups**\.

1. <a name="group-choose-group"></a>Choose the group whose settings you want to change\.

1. <a name="group-choose-settings"></a>Choose **Settings**\.

1. Under **Lambda runtime environment**, choose **No container**\.

1. Choose **Update default Lambda execution configuration**\. Review the message in the confirmation window, and then choose **Continue**\.

For more information, see [Setting default containerization for Lambda functions in a group](lambda-group-config.md#lambda-containerization-groupsettings)\.

**Note**  
By default, Lambda functions use the group containerization setting\. If you override the **No container** setting for any Lambda functions when AWS IoT Greengrass is running in a Docker container, the deployment fails\.

## Step 5: Deploy Lambda functions to the AWS IoT Greengrass Docker container<a name="docker-add-lambdas"></a>

You can deploy long\-lived Lambda functions to the Greengrass Docker container\.
+ Follow the steps in [Module 3 \(part 1\): Lambda functions on AWS IoT Greengrass](module3-I.md) to deploy a long\-lived Hello World Lambda function to the container\.

## Step 6: \(Optional\) Deploy devices that interact with Greengrass running in the Docker container<a name="docker-add-devices"></a>

You can also deploy Greengrass devices that interact with AWS IoT Greengrass when it's running in a Docker container\.
+ Follow the steps in [Module 4: Interacting with devices in an AWS IoT Greengrass group](module4.md) to deploy devices that connect to the core and send MQTT messages\.

## Stopping the AWS IoT Greengrass Docker container<a name="docker-stop"></a>

To stop the AWS IoT Greengrass Docker container, press Ctrl\+C in your terminal or command prompt\. This action sends `SIGTERM` to the Greengrass daemon process to tear down the Greengrass daemon process and all Lambda processes that were started by the daemon process\. The Docker container is initialized with `/dev/init` process as PID 1, which helps in removing any leftover zombie processes\. For more information, see the [Docker run reference](https://docs.docker.com/engine/reference/commandline/run/#options)\.

## Troubleshooting AWS IoT Greengrass in a Docker container<a name="troubleshooting-docker-gg"></a>

Use the following information to help troubleshoot issues with running AWS IoT Greengrass in a Docker container\.

### Error: Cannot perform an interactive login from a non TTY device\.<a name="docker-troubleshootin-ecr-get-login-password"></a>

**Solution:** This error can occur when you run the `aws ecr get-login-password` command\. Make sure that you installed the latest AWS CLI version 2 or version 1\. We recommend that you use the AWS CLI version 2\. For more information, see [Installing the AWS CLI](https://docs.aws.amazon.com/cli/latest/userguide/cli-chap-install.html) in the *AWS Command Line Interface User Guide*\.

### Error: Unknown options: \-no\-include\-email\.<a name="docker-troubleshooting-cli-version"></a>

**Solution:** This error can occur when you run the `aws ecr get-login` command\. Make sure that you have the latest AWS CLI version installed \(for example, run: `pip install awscli --upgrade --user`\)\. If you're using Windows and you installed the CLI using the MSI installer, you must repeat the installation process\. For more information, see [Installing the AWS Command Line Interface on Microsoft Windows](https://docs.aws.amazon.com/cli/latest/userguide/awscli-install-windows.html) in the *AWS Command Line Interface User Guide*\.

### Warning: IPv4 is disabled\. Networking will not work\.<a name="docker-troubleshooting-ipv4-disabled"></a>

**Solution:** You might receive this warning or a similar message when running AWS IoT Greengrass on a Linux computer\. Enable IPv4 network forwarding as described in this [step](#docker-linux-enable-ipv4)\. AWS IoT Greengrass cloud deployment and MQTT communications don't work when IPv4 forwarding isn't enabled\. For more information, see [Configure namespaced kernel parameters \(sysctls\) at runtime](https://docs.docker.com/engine/reference/commandline/run/#configure-namespaced-kernel-parameters-sysctls-at-runtime) in the Docker documentation\.

### Error: A firewall is blocking file Sharing between windows and the containers\.<a name="docker-troubleshooting-firewall"></a>

**Solution:** You might receive this error or a `Firewall Detected` message when running Docker on a Windows computer\. This can also occur if you are signed in on a virtual private network \(VPN\) and your network settings are preventing the shared drive from being mounted\. In that situation, turn off VPN and re\-run the Docker container\.

### Error: An error occurred \(AccessDeniedException\) when calling the GetAuthorizationToken operation: User: arn:aws:iam::<account\-id>:user/<user\-name> is not authorized to perform: ecr:GetAuthorizationToken on resource: \*<a name="docker-troubleshooting-ecr-perms"></a>

You might receive this error when running the `aws ecr get-login-password` command if you don't have sufficient permissions to access an Amazon ECR repository\. For more information, see [Amazon ECR Repository Policy Examples](https://docs.aws.amazon.com/AmazonECR/latest/userguide/repository-policy-examples.html) and [Accessing One Amazon ECR Repository](https://docs.aws.amazon.com/AmazonECR/latest/userguide/security_iam_id-based-policy-examples.html) in the *Amazon ECR User Guide*\.

For general AWS IoT Greengrass troubleshooting help, see [Troubleshooting AWS IoT Greengrass](gg-troubleshooting.md)\.

### Debugging AWS IoT Greengrass in a Docker container<a name="debugging-docker-gg"></a>

To debug issues with a Docker container, you can persist the Greengrass runtime logs or attach an interactive shell to the Docker container\.

#### To persist Greengrass runtime logs outside of the Docker container<a name="debugging-docker-persist-logs"></a>

You can run the AWS IoT Greengrass Docker container after bind\-mounting the `/greengrass/ggc/var/log` directory\. The logs persist even after the container exits or is removed\.

**On Linux or macOS**  
[Stop any Greengrass Docker containers](#docker-stop) running on the host, and then run the following command in a terminal\. This bind\-mounts the Greengrass `log` directory and starts the Docker image\.   
Replace `/tmp` with the path where you decompressed your certificates and configuration file\.  

```
docker run --rm --init -it --name aws-iot-greengrass \
      --entrypoint /greengrass-entrypoint.sh \
      -v /tmp/certs:/greengrass/certs \
      -v /tmp/config:/greengrass/config \
      -v /tmp/log:/greengrass/ggc/var/log \
      -p 8883:8883 \
      216483018798.dkr.ecr.us-west-2.amazonaws.com/aws-iot-greengrass:latest
```
You can then check your logs at `/tmp/log` on your host to see what happened while Greengrass was running inside the Docker container\.

**On Windows**  
[Stop any Greengrass Docker containers](#docker-stop) running on the host, and then run the following command in a command prompt\. This bind\-mounts the Greengrass `log` directory and starts the Docker image\.  

```
cd C:\Users\%USERNAME%\Downloads
mkdir log
docker run --rm --init -it --name aws-iot-greengrass --entrypoint /greengrass-entrypoint.sh -v c:/Users/%USERNAME%/Downloads/certs:/greengrass/certs -v c:/Users/%USERNAME%/Downloads/config:/greengrass/config -v c:/Users/%USERNAME%/Downloads/log:/greengrass/ggc/var/log -p 8883:8883 216483018798.dkr.ecr.us-west-2.amazonaws.com/aws-iot-greengrass:latest
```
You can then check your logs at `C:/Users/%USERNAME%/Downloads/log` on your host to see what happened while Greengrass was running inside the Docker container\.

#### To attach an interactive shell to the Docker container<a name="debugging-docker-attach-shell"></a>

You can attach an interactive shell to a running AWS IoT Greengrass Docker container\. This can help you investigate the state of the Greengrass Docker container\.

**On Linux or macOS**  
While the Greengrass Docker container is running, run the following command in a separate terminal\.  

```
docker exec -it $(docker ps -a -q -f "name=aws-iot-greengrass") /bin/bash
```

**On Windows**  
While the Greengrass Docker container is running, run the following commands in a separate command prompt\.  

```
docker ps -a -q -f "name=aws-iot-greengrass"
```
Replace *gg\-container\-id* with the `container_id` result from the previous command\.  

```
docker exec -it gg-container-id /bin/bash
```