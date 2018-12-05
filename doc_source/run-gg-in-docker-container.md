# Running AWS IoT Greengrass in a Docker Container<a name="run-gg-in-docker-container"></a>

You can now run AWS IoT Greengrass in a Docker container\. You can follow the instructions in the Docker file available [here, through Cloudfront](https://d1onfpft10uf5o.cloudfront.net/greengrass-core/downloads/1.7.0/aws-greengrass-docker-1.7.0.tar.gz), or you can access our pre\-built Docker image by using the following steps:
+ Get the AWS IoT Greengrass container image from Amazon ECR\.
+ Create and configure the Greengrass group\. This includes downloading certificates and the core\-specific config file\. For more information, see [Configure AWS IoT Greengrass on AWS IoT](gg-config.md)\. Skip step 8b of the following procedure\. You have the AWS IoT Greengrass container image from Amazon ECR\.
+ Run AWS IoT Greengrass locally\.
+ Before you deploy for the first time, you must configure the containerization for the group to use no container\. For more information, see [Setting Default Containerization for Lambda Functions in a Group](lambda-group-config.md#lambda-containerization-groupsettings)\.
**Note**  
When you run AWS IoT Greengrass in a Docker container, all Lambda functions must run without containerization\. By default, Lambda functions use the group setting for containerization\. If you override that default for one or more Lambda functions, the deployment fails when you are running AWS IoT Greengrass in a Docker container\.

**To pull the AWS IoT Greengrass container image from Amazon ECR**

1. Run the login command to authenticate your Docker client to the `aws-greengrass-docker` registry\.

   If your are using a macOS or Linux computer, use the AWS CLI\. For more information, see [Installing the AWS Command Line Interface](https://docs.aws.amazon.com/cli/latest/userguide/installing.html) and [Configuring the AWS CLI](https://docs.aws.amazon.com/cli/latest/userguide/cli-chap-getting-started.html) in the *AWS Command Line Interface User Guide*\.

   ```
   $(aws ecr get-login --registry-ids 216483018798 --no-include-email --region us-west-2)
   ```

   If you are using a Windows computer, use AWS Tools for PowerShell\. For more information, see [Setting up the AWS Tools for PowerShell on a Windows\-based Computer](https://docs.aws.amazon.com/powershell/latest/userguide/pstools-getting-set-up-windows.html)\. For example:

   ```
   Invoke-Expression -Command (Get-ECRLoginCommand -RegistryId 216483018798 -Region us-west-2 -AccessKey AWS_ACCESS_KEY_ID -SecretKey AWS_SECRET_ACCESS_KEY -SessionToken AWS_SESSION_TOKEN).Command
   ```
**Note**  
You can get the temporary AWS credentials from AWS IAM console\. For more information, see [Requesting Temporary Security Credentials](https://docs.aws.amazon.com//IAM/latest/UserGuide/id_credentials_temp_request.html#api_assumerole)\.

   The result of this command \(on either platform\) is a docker login command that you use to authenticate your Docker client to the AWS IoT Greengrass container image in the Amazon ECR registry\.
**Note**  
If you receive an `"Unknown options: -no-include-email"` error when you use the AWS CLI, make sure that you have the latest version installed\.

1. Retrieve the AWS IoT Greengrass container image by using the docker pull command:

   ```
   docker pull 216483018798.dkr.ecr.us-west-2.amazonaws.com/aws-greengrass-docker/amazonlinux:latest
   ```

   The `latest` tag corresponds to the latest AWS IoT Greengrass container that is available\.
**Note**  
You can list the images that are available in the AWS IoT Greengrass repository by using the aws ecr list\-images\. For example:  

   ```
   aws ecr list-images --region us-west-2 --registry-id 216483018798 --repository-name aws-greengrass-docker/amazonlinux
   ```

1. If you are using a Linux computer to run Docker, you must enable symlink and hardlink protection\. On macOS or Windows computers, this step is not required because Docker runs in a Virtual machine that already comes with these file\-system settings enabled\. On a Linux computer, these link protection settings are essential to running the AWS IoT Greengrass core software\.

   To enable the settings only for the current boot, use the following commands:

   ```
   echo 1 > /proc/sys/fs/protected_hardlinks
   echo 1 > /proc/sys/fs/protected_symlinks
   ```

   To enable the settings to last across restarts, instead use the following commands:

   ```
   echo '# AWS Greengrass' >> /etc/sysctl.conf 
   echo 'fs.protected_hardlinks = 1' >> /etc/sysctl.conf 
   echo 'fs.protected_symlinks = 1' >> /etc/sysctl.conf
   
   sysctl -p
   ```
**Note**  
You may need to use sudo to run these commands\.

1. If you are using a Linux computer to run Docker, you must also enable IPv4 network forwarding\. On macOS or Windows computers, this step is not required because Docker runs in a Virtual Machine that already comes with this network setting enabled\. On a Linux computer, if you don't enable this setting, you receive an error message such as `WARNING: IPv4 is disabled. Networking will not work.` As a result, AWS IoT Greengrass cloud deployment and MQTT communications will not work\. For more information, see [Docker help for configuring namespaced kernel parameters \(sysctls\) at runtime](https://docs.docker.com/engine/reference/commandline/run/#configure-namespaced-kernel-parameters-sysctls-at-runtime)\.

   To enable IPv4 forwarding, edit the `/etc/sysctl.conf` file to set `net.ipv4.ip_forward` to 1 and then reload sysctls with the following commands:

   ```
   sudo nano /etc/sysctl.conf
   # set this net.ipv4.ip_forward = 1
   sudo sysctl -p
   ```
**Note**  
You can use the editor of your choice in place of nano\.

**To run AWS IoT Greengrass locally**

1. Create and configure the Greengrass group\. This process includes downloading certificates and the core\-specific config file\. For more information, see [Configure AWS IoT Greengrass on AWS IoT](gg-config.md)\. You can skip step 8b of the procedure because AWS IoT Greengrass core and its runtime dependencies are already setup in the Docker image\.

1. If you are using a macOS or Linux computer, decompress the certificates and config files that you downloaded when you created your Greengrass group into a known location, such as `/tmp`\. For example:

   ```
   tar xvzf guid-setup.tar.gz -C /tmp/
   ```

   If you are using a Windows computer, use a utility such as WinZIP or 7Zip to decompress the certificates and config files that you downloaded\. Locate the downloaded *guid*\-setup\.tar\.gz file on your computer and then decompress the file to a known location, such as C:\\Users\\*user*\\Downloads\\\. For more information, see [WinZip help](https://www.winzip.com/win/en/tar-file.html)\.

1. Review the documentation on [Server Authentication in AWS IoT Core](https://docs.aws.amazon.com/iot/latest/developerguide/managing-device-certs.html), follow the links to download the appropriate CA certificates, and then copy them to your device\. These certificates enable your device to communicate with AWS IoT using the MQTT messaging protocol over TLS\. Make sure the Docker host is connected to the internet, and then follow the steps appropriate for the computer on which you are running Docker\.

   If you are using a macOS or Linux computer, complete the following steps:

   1. Run the following command:

      ```
      cd /tmp/certs/
      sudo wget -O root.ca.pem https://www.amazontrust.com/repository/AmazonRootCA1.pem
      ```
**Note**  
Note that `-O` is the capital letter O\.

   1. Run the following command to confirm that the `root.ca.pem` file is not empty:

      ```
      cat root.ca.pem
      ```

   1. If the `root.ca.pem` file is empty, check the `wget` URL and try again\.

   If you are using a Windows computer, complete the following steps:

   1. Open the following link in your web browser: [AmazonRootCA1\.pem](https://www.amazontrust.com/repository/AmazonRootCA1.pem) and copy the displayed key to the clipboard\.

   1. Open Notepad and paste the copied key\.

   1. Save the document as `root.ca.pem` in the directory where you decompressed the certificate files \(for example, C:\\Users\\*user*\\Downloads\\certs\)\.

1. In the same console that you used to decompress the certificates, if you are using a macOS or Linux computer, use the following command:

   ```
   docker run --rm --init -t --name aws-greengrass-docker \
   --entrypoint /greengrass-entrypoint.sh \
   -v /tmp/certs:/greengrass/certs \
   -v /tmp/config:/greengrass/config \
   -p 8883:8883 \
   216483018798.dkr.ecr.us-west-2.amazonaws.com/aws-greengrass-docker/amazonlinux:latest
   ```

   Replace `/tmp` with the path where you downloaded your certificates and config files\.

   If you are using a Windows computer, open PowerShell and run the AWS IoT Greengrass Docker container by using the following command:

   ```
   docker run --rm --init -t --name aws-greengrass-docker \
   --entrypoint /greengrass-entrypoint.sh \
   -v c:/Users/User/Downloads/certs:/greengrass/certs \
   -v c:/Users/User/Downloads/config:/greengrass/config \
   -p 8883:8883 \
   216483018798.dkr.ecr.us-west-2.amazonaws.com/aws-greengrass-docker/amazonlinux:latest
   ```
**Note**  
Docker will prompt you to share your C:\\ drive with Docker daemon\. Allow it to bind\-mount C:\\ directory inside the Docker container\. For more information about shared drives, see: [Docker help for shared drives](https://docs.docker.com/docker-for-windows/#shared-drives)\. 

   The output should look like this example:

   ```
   Setting up greengrass daemon
   Validating hardlink/softlink protection
   Waiting for up to 30s for Daemon to start
   
   Greengrass successfully started with PID: 10
   ```
**Note**  
This command starts AWS IoT Greengrass and bind mounts the certificates and config files\. 

1. Go to the AWS IoT Greengrass console and deploy your Lambda function to the AWS IoT Greengrass core running inside the Docker container\. For more information, see [Deploy Cloud Configurations to an AWS IoT Greengrass Core Device](configs-core.md)\. Make sure that you have set the containerization for the group to **No container**\. For more information, see [Setting Default Containerization for Lambda Functions in a Group](lambda-group-config.md#lambda-containerization-groupsettings)\.

To stop the AWS IoT Greengrass Docker Container, press Ctrl\+C\. That action will send SIGTERM to the Greengrass daemon process to tear down the Greengrass daemon process and all Lambda processes that were started by the daemon process\. The Docker container is initialized with /dev/init process as PID 1, which helps in removing any leftover zombie processes\. For more information, see [Docker run reference](https://docs.docker.com/engine/reference/commandline/run/#options)\.

## Troubleshooting When Running AWS IoT Greengrass in a Docker Container<a name="troubleshooting-docker-gg"></a>

 If you see the message, `Firewall Detected while Sharing Drives` when running Docker on a Windows computer, see the following [Docker article](https://success.docker.com/article/error-a-firewall-is-blocking-file-sharing-between-windows-and-the-containers) for troubleshooting help\. This error can also occur if you are logged in on a Virtual Private Network \(VPN\) and your network settings are preventing the shared drive from being mounted\. In that situation, turn off VPN and re\-run the Docker container\. 