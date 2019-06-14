# Running AWS IoT Greengrass in a Docker Container<a name="run-gg-in-docker-container"></a>

This feature is available for AWS IoT Greengrass Core v1\.7 and later\.

AWS IoT Greengrass can be configured to run in a [Docker](https://www.docker.com/) container\. 

**Note**  
Connectors, local device and volume resources, and local machine learning model resources can't be used in a Docker container\. These features aren't supported when the Lambda runtime environment for the Greengrass group is set to [**No container**](lambda-group-config.md#no-container-mode), which is required to run AWS IoT Greengrass in a Docker container\.

You can download a Dockerfile [through Amazon CloudFront](what-is-gg.md#gg-docker-download) that has the AWS IoT Greengrass Core software and dependencies installed\. To modify the Docker image to run on different platform architectures or reduce the size of the Docker image, see the `README` file in the Docker package download\.

To help you get started quickly and experiment with AWS IoT Greengrass, AWS also provides a prebuilt Docker image that has the AWS IoT Greengrass Core software and dependencies installed\. You can download the prebuilt image from [Docker Hub](https://hub.docker.com/r/amazon/aws-iot-greengrass) or [Amazon Elastic Container Registry](https://docs.aws.amazon.com/AmazonECR/latest/userguide/what-is-ecr.html) \(Amazon ECR\) and run it on Windows, macOS, and Linux \(x86\_64\) platforms\. This topic describes how to download the image from Amazon ECR\. It contains the following steps:

1. [Get the AWS IoT Greengrass Container Image from Amazon ECR](#docker-pull-image)

1. [Create and Configure the Greengrass Group and Core](#docker-config-gg)

1. [Run AWS IoT Greengrass Locally](#docker-run-gg)

1. [Configure "No container" Containerization for the Group](#docker-no-container)

1. [Deploy Lambda Functions to the Docker Container](#docker-add-lambdas)

1. [\(Optional\) Deploy Devices that Interact with Greengrass in the Docker Container](#docker-add-devices)

## Prerequisites<a name="docker-image-prerequisites"></a>

To complete this tutorial, the following software and versions must be installed on your host computer\.
+ [Docker](https://docs.docker.com/install/), version 18\.09 or later\. Earlier versions might also work, but version 18\.09 or later is preferred\.
+ [Python](https://www.python.org/downloads/), version 3\.6 or later\.
+ [pip](https://pip.pypa.io/en/stable/installing) version 18\.1 or later\.
+ AWS CLI version 1\.16 or later\.
  + To install and configure the CLI, see [Installing the AWS Command Line Interface](https://docs.aws.amazon.com/cli/latest/userguide/installing.html) and [Configuring the AWS CLI](https://docs.aws.amazon.com/cli/latest/userguide/cli-chap-getting-started.html) in the *AWS Command Line Interface User Guide*\.
  + To upgrade to the latest version of the AWS CLI, run the following command:

    ```
    pip install awscli --upgrade --user
    ```
**Note**  
If you use the [ MSI installation](https://docs.aws.amazon.com/cli/latest/userguide/awscli-install-windows.html#install-msi-on-windows) of the AWS CLI on Windows, be aware of the following:  
If the installation fails to install botocore, try using the [Python and pip installation](https://docs.aws.amazon.com/cli/latest/userguide/awscli-install-windows.html#awscli-install-windows-pip)\.
To upgrade to a newer CLI version, you must repeat the MSI installation process\.

## Step 1: Get the AWS IoT Greengrass Container Image from Amazon ECR<a name="docker-pull-image"></a>

AWS IoT Greengrass provides a Docker image that has the AWS IoT Greengrass Core software installed\. For steps that show how to pull the container image from Amazon ECR, choose your operating system:

### Pull the Container Image \(Linux\)<a name="docker-pull-image-linux"></a>

Run the following commands in your computer terminal\.

1. <a name="docker-get-login"></a>Get the required login command, which contains an authorization token for the AWS IoT Greengrass registry in Amazon ECR\.

   ```
   aws ecr get-login --registry-ids 216483018798 --no-include-email --region us-west-2
   ```

   The output is the `docker login` command that you use in the next step\.

1. <a name="docker-docker-login"></a>Authenticate your Docker client to the AWS IoT Greengrass container image in the registry by running the `docker login` command from the `get-login` output\. The command should be similar to the following example\.

   ```
   docker login -u AWS -p abCzYZ123... https://216483018798.dkr.ecr.us-west-2.amazonaws.com
   ```

1. <a name="docker-docker-pull"></a>Retrieve the AWS IoT Greengrass container image\.

   ```
   docker pull 216483018798.dkr.ecr.us-west-2.amazonaws.com/aws-iot-greengrass:latest
   ```
**Note**  
The `latest` tag corresponds to the latest AWS IoT Greengrass container\. You can also pull other versions from the repository\. To list all images that are available in the AWS IoT Greengrass repository, use the aws ecr list\-images command\. For example:  

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

### Pull the Container Image \(macOS\)<a name="docker-pull-image-mac"></a>

Run the following commands in your computer terminal\.

1. <a name="docker-get-login"></a>Get the required login command, which contains an authorization token for the AWS IoT Greengrass registry in Amazon ECR\.

   ```
   aws ecr get-login --registry-ids 216483018798 --no-include-email --region us-west-2
   ```

   The output is the `docker login` command that you use in the next step\.

1. <a name="docker-docker-login"></a>Authenticate your Docker client to the AWS IoT Greengrass container image in the registry by running the `docker login` command from the `get-login` output\. The command should be similar to the following example\.

   ```
   docker login -u AWS -p abCzYZ123... https://216483018798.dkr.ecr.us-west-2.amazonaws.com
   ```

1. <a name="docker-docker-pull"></a>Retrieve the AWS IoT Greengrass container image\.

   ```
   docker pull 216483018798.dkr.ecr.us-west-2.amazonaws.com/aws-iot-greengrass:latest
   ```
**Note**  
The `latest` tag corresponds to the latest AWS IoT Greengrass container\. You can also pull other versions from the repository\. To list all images that are available in the AWS IoT Greengrass repository, use the aws ecr list\-images command\. For example:  

   ```
   aws ecr list-images --region us-west-2 --registry-id 216483018798 --repository-name aws-iot-greengrass
   ```

### Pull the Container Image \(Windows\)<a name="docker-pull-image-windows"></a>

Run the following commands in a command prompt\. Before you can use Docker commands on Windows, Docker Desktop must be running\.

1. <a name="docker-get-login"></a>Get the required login command, which contains an authorization token for the AWS IoT Greengrass registry in Amazon ECR\.

   ```
   aws ecr get-login --registry-ids 216483018798 --no-include-email --region us-west-2
   ```

   The output is the `docker login` command that you use in the next step\.

1. <a name="docker-docker-login"></a>Authenticate your Docker client to the AWS IoT Greengrass container image in the registry by running the `docker login` command from the `get-login` output\. The command should be similar to the following example\.

   ```
   docker login -u AWS -p abCzYZ123... https://216483018798.dkr.ecr.us-west-2.amazonaws.com
   ```

1. <a name="docker-docker-pull"></a>Retrieve the AWS IoT Greengrass container image\.

   ```
   docker pull 216483018798.dkr.ecr.us-west-2.amazonaws.com/aws-iot-greengrass:latest
   ```
**Note**  
The `latest` tag corresponds to the latest AWS IoT Greengrass container\. You can also pull other versions from the repository\. To list all images that are available in the AWS IoT Greengrass repository, use the aws ecr list\-images command\. For example:  

   ```
   aws ecr list-images --region us-west-2 --registry-id 216483018798 --repository-name aws-iot-greengrass
   ```

## Step 2: Create and Configure the Greengrass Group and Core<a name="docker-config-gg"></a>

The Docker image has the AWS IoT Greengrass Core software installed, but you must create a Greengrass group and core\. This includes downloading certificates and the core configuration file\.
+ Follow the steps in [Configure AWS IoT Greengrass on AWS IoT](gg-config.md)\. Skip step 8b where you download the AWS IoT Greengrass Core software\. The software and its runtime dependencies are already set up in the Docker image\.

## Step 3: Run AWS IoT Greengrass Locally<a name="docker-run-gg"></a>

After your group is configured, you're ready to configure and start the core\. For steps that show how to do this, choose your operating system:

### Run Greengrass Locally \(Linux\)<a name="docker-run-gg-linux"></a>

Run the following commands in your computer terminal\.

1. <a name="docker-untar-files"></a>Decompress the certificates and configuration file \(that you downloaded when you created your Greengrass group\) into a known location, such as `/tmp`\. For example:

   ```
   tar xvzf hash-setup.tar.gz -C /tmp/
   ```

1. <a name="docker-root-ca"></a>Review the documentation about [Server Authentication in AWS IoT Core](https://docs.aws.amazon.com/iot/latest/developerguide/managing-device-certs.html#server-authentication) and choose the appropriate root CA certificate\. We recommend that you use Amazon Trust Services \(ATS\) endpoints and ATS root CA certificates\.

   Run the following commands to download the root CA certificate to the directory where you decompressed the certificates and configuration file\. Certificates enable your device to connect to AWS IoT over TLS\.

   Replace `/tmp` with the path to the directory\.
**Important**  
Your root CA certificate type must match your endpoint\. Use an ATS root CA certificate with an ATS endpoint \(preferred\) or a Verisign root CA certificate with a legacy endpoint\. For more information, see [Endpoints Must Match the Certificate Type](gg-core.md#certificate-endpoints)\.
   + For ATS endpoints \(preferred\), download the appropriate ATS root CA certificate\. The following example downloads `AmazonRootCA1.pem`\.

     ```
     cd /tmp/certs/
     sudo wget -O root.ca.pem https://www.amazontrust.com/repository/AmazonRootCA1.pem
     ```
   + For legacy endpoints, download a Verisign root CA certificate\. Although legacy endpoints are acceptable for the purposes of this tutorial, we recommend that you create an ATS endpoint and download an ATS root CA certificate\.

     ```
     cd /tmp/certs/
     sudo wget -O root.ca.pem https://www.symantec.com/content/en/us/enterprise/verisign/roots/VeriSign-Class%203-Public-Primary-Certification-Authority-G5.pem
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

### Run Greengrass Locally \(macOS\)<a name="docker-run-gg-mac"></a>

Run the following commands in your computer terminal\.

1. <a name="docker-untar-files"></a>Decompress the certificates and configuration file \(that you downloaded when you created your Greengrass group\) into a known location, such as `/tmp`\. For example:

   ```
   tar xvzf hash-setup.tar.gz -C /tmp/
   ```

1. <a name="docker-root-ca"></a>Review the documentation about [Server Authentication in AWS IoT Core](https://docs.aws.amazon.com/iot/latest/developerguide/managing-device-certs.html#server-authentication) and choose the appropriate root CA certificate\. We recommend that you use Amazon Trust Services \(ATS\) endpoints and ATS root CA certificates\.

   Run the following commands to download the root CA certificate to the directory where you decompressed the certificates and configuration file\. Certificates enable your device to connect to AWS IoT over TLS\.

   Replace `/tmp` with the path to the directory\.
**Important**  
Your root CA certificate type must match your endpoint\. Use an ATS root CA certificate with an ATS endpoint \(preferred\) or a Verisign root CA certificate with a legacy endpoint\. For more information, see [Endpoints Must Match the Certificate Type](gg-core.md#certificate-endpoints)\.
   + For ATS endpoints \(preferred\), download the appropriate ATS root CA certificate\. The following example downloads `AmazonRootCA1.pem`\.

     ```
     cd /tmp/certs/
     sudo wget -O root.ca.pem https://www.amazontrust.com/repository/AmazonRootCA1.pem
     ```
   + For legacy endpoints, download a Verisign root CA certificate\. Although legacy endpoints are acceptable for the purposes of this tutorial, we recommend that you create an ATS endpoint and download an ATS root CA certificate\.

     ```
     cd /tmp/certs/
     sudo wget -O root.ca.pem https://www.symantec.com/content/en/us/enterprise/verisign/roots/VeriSign-Class%203-Public-Primary-Certification-Authority-G5.pem
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

### Run Greengrass Locally \(Windows\)<a name="docker-run-gg-windows"></a>

1. Use a utility such as WinZip or 7\-Zip to decompress the certificates and configuration file that you downloaded when you created your Greengrass group\. For more information, see the [WinZip](https://www.winzip.com/win/en/tar-file.html) documentation\.

   Locate the downloaded `hash-setup.tar.gz` file on your computer and then decompress the file into `C:\Users\%USERNAME%\Downloads\`\.

1. Review the documentation about [Server Authentication in AWS IoT Core](https://docs.aws.amazon.com/iot/latest/developerguide/managing-device-certs.html#server-authentication) and choose the appropriate root CA certificate\. We recommend that you use Amazon Trust Services \(ATS\) endpoints and ATS root CA certificates\.

   Run the following commands to download the root CA certificate to the directory where you decompressed the certificates and configuration file\. Certificates enable your device to connect to AWS IoT over TLS\.
**Important**  
Your root CA certificate type must match your endpoint\. Use an ATS root CA certificate with an ATS endpoint \(preferred\) or a Verisign root CA certificate with a legacy endpoint\. For more information, see [Endpoints Must Match the Certificate Type](gg-core.md#certificate-endpoints)\.
   + For ATS endpoints \(preferred\), download the appropriate ATS root CA certificate\. The following example downloads `AmazonRootCA1.pem`\.
     + If you have [curl](https://curl.haxx.se/download.html) installed, run the following commands in your command prompt\.

       ```
       cd C:\Users\%USERNAME%\Downloads\certs
       curl https://www.amazontrust.com/repository/AmazonRootCA1.pem -o root.ca.pem
       ```
     + Otherwise, in a web browser, open the [Amazon Root CA 1](https://www.amazontrust.com/repository/AmazonRootCA1.pem) certificate\. Save the document as `root.ca.pem` in the `C:\Users\%USERNAME%\Downloads\certs` directory, which contains the decompressed certificates\.
**Note**  
Depending on your browser, save the file directly from the browser or copy the displayed key to the clipboard and save it in Notepad\.
   + For legacy endpoints, download a Verisign root CA certificate\. Although legacy endpoints are acceptable for the purposes of this tutorial, we recommend that you create an ATS endpoint and download an ATS root CA certificate\.
     + If you have [curl](https://curl.haxx.se/download.html) installed, run the following commands in your command prompt\.

       ```
       cd C:\Users\%USERNAME%\Downloads\certs
       curl https://www.symantec.com/content/en/us/enterprise/verisign/roots/VeriSign-Class%203-Public-Primary-Certification-Authority-G5.pem -o root.ca.pem
       ```
     + Otherwise, in a web browser, open the [Verisign Class 3 Public Primary G5 root CA certificate](https://www.symantec.com/content/en/us/enterprise/verisign/roots/VeriSign-Class%203-Public-Primary-Certification-Authority-G5.pem)\. Save the document as `root.ca.pem` in the `C:\Users\%USERNAME%\Downloads\certs` directory, which contains the decompressed certificates\.
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
If the container doesn't open the shell and exits immediately, you can debug the issue by bind\-mounting the Greengrass runtime logs when you start the image\. For more information, see [To Persist Greengrass Runtime Logs Outside of the Docker Container](#debugging-docker-persist-logs)\.

## Step 4: Configure "No container" Containerization for the Greengrass Group<a name="docker-no-container"></a>

When you run AWS IoT Greengrass in a Docker container, all Lambda functions must run without containerization\. In this step, you set the default containerization for the group to **No container**\. You must do this before you deploy the group for the first time\.

1. <a name="console-gg-groups"></a>In the AWS IoT console, choose **Greengrass**, and then choose **Groups**\.

1. <a name="group-choose-group"></a>Choose the group whose settings you want to change\.

1. <a name="group-choose-settings"></a>Choose **Settings**\.

1. Under **Lambda runtime environment**, choose **No container**\.

1. Choose **Update default Lambda execution configuration**\. Review the message in the confirmation window, and then choose **Continue**\.

For more information, see [Setting Default Containerization for Lambda Functions in a Group](lambda-group-config.md#lambda-containerization-groupsettings)\.

**Note**  
By default, Lambda functions use the group containerization setting\. If you override the **No container** setting for any Lambda functions when AWS IoT Greengrass is running in a Docker container, the deployment fails\.

## Step 5: Deploy Lambda Functions to the AWS IoT Greengrass Docker Container<a name="docker-add-lambdas"></a>

You can deploy long\-lived Lambda functions to the Greengrass Docker container\.
+ Follow the steps in [Module 3 \(Part 1\): Lambda Functions on AWS IoT Greengrass](module3-I.md) to deploy a long\-lived Hello World Lambda function to the container\.

## Step 6: \(Optional\) Deploy Devices that Interact with Greengrass Running in the Docker Container<a name="docker-add-devices"></a>

You can also deploy Greengrass devices that interact with AWS IoT Greengrass when it's running in a Docker container\.
+ Follow the steps in [Module 4: Interacting with Devices in an AWS IoT Greengrass Group](module4.md) to deploy devices that connect to the core and send MQTT messages\.

## Stopping the AWS IoT Greengrass Docker Container<a name="docker-stop"></a>

To stop the AWS IoT Greengrass Docker container, press Ctrl\+C in your terminal or command prompt\. This action sends `SIGTERM` to the Greengrass daemon process to tear down the Greengrass daemon process and all Lambda processes that were started by the daemon process\. The Docker container is initialized with `/dev/init` process as PID 1, which helps in removing any leftover zombie processes\. For more information, see the [Docker run reference](https://docs.docker.com/engine/reference/commandline/run/#options)\.

## Troubleshooting AWS IoT Greengrass in a Docker Container<a name="troubleshooting-docker-gg"></a>

Use the following information to help troubleshoot issues with running AWS IoT Greengrass in a Docker container\.


| Symptom | Solution | 
| --- | --- | 
| <a name="docker-troubleshooting-cli-version"></a> You receive the error: `Unknown options: -no-include-email` when running the `aws ecr get-login` command\.  |  Make sure that you have the latest AWS CLI version installed \(for example, run: `pip install awscli --upgrade --user`\)\. If you're using Windows and you installed the CLI using the MSI installer, you must repeat the installation process\. For more information, see [Installing the AWS Command Line Interface on Microsoft Windows](https://docs.aws.amazon.com/cli/latest/userguide/awscli-install-windows.html) in the *AWS Command Line Interface User Guide*\.  | 
| <a name="docker-troubleshooting-ipv4-disabled"></a> You receive an error such as: `WARNING: IPv4 is disabled. Networking will not work` on a Linux computer\.  |  Enable IPv4 network forwarding as described in this [step](#docker-linux-enable-ipv4)\. AWS IoT Greengrass cloud deployment and MQTT communications don't work when IPv4 forwarding isn't enabled\. For more information, see [Configure namespaced kernel parameters \(sysctls\) at runtime](https://docs.docker.com/engine/reference/commandline/run/#configure-namespaced-kernel-parameters-sysctls-at-runtime) in the Docker documentation\.  | 
| <a name="docker-troubleshooting-firewall"></a> You receive the message: `Firewall Detected while Sharing Drives` when running Docker on a Windows computer\.  |  See the [Error: A firewall is blocking file sharing between Windows and the containers](https://success.docker.com/article/error-a-firewall-is-blocking-file-sharing-between-windows-and-the-containers) Docker support issue\. This error can also occur if you are signed in on a virtual private network \(VPN\) and your network settings are preventing the shared drive from being mounted\. In that situation, turn off VPN and re\-run the Docker container\.  | 

For general AWS IoT Greengrass troubleshooting help, see [Troubleshooting AWS IoT Greengrass](gg-troubleshooting.md)\.

### Debugging AWS IoT Greengrass in a Docker Container<a name="debugging-docker-gg"></a>

To debug issues with a Docker container, you can persist the Greengrass runtime logs or attach an interactive shell to the Docker container\.

#### To Persist Greengrass Runtime Logs Outside of the Docker Container<a name="debugging-docker-persist-logs"></a>

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

#### To Attach an Interactive Shell to the Docker Container<a name="debugging-docker-attach-shell"></a>

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