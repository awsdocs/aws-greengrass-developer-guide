# Optional: Configuring your Docker container for IDT for AWS IoT Greengrass<a name="docker-config-setup"></a>

AWS IoT Greengrass provides a Docker image and Dockerfile that make it easier to run the AWS IoT Greengrass Core software in a Docker container\. After you set up the AWS IoT Greengrass container, you can run IDT tests\. Currently, only x86\_64 Docker architectures are supported to run IDT for AWS IoT Greengrass\.

This feature requires IDT v2\.3\.0 or later\.

The process of setting up the Docker container to run IDT tests depends on whether you use the Docker image or Dockerfile provided by AWS IoT Greengrass\.
+ [Use the Docker image](#docker-config-setup-docker-image)\. The Docker image has the AWS IoT Greengrass Core software and dependencies installed\.
+ [Use the Dockerfile](#docker-config-setup-dockerfile)\. The Dockerfile contains source code you can use to build custom AWS IoT Greengrass container images\. The image can be modified to run on different platform architectures or to reduce the image size\.
**Note**  
To run IDT tests on your own custom container images, your image must include the dependencies defined in the Dockerfile provided by AWS IoT Greengrass\.

The following features aren't available when you run AWS IoT Greengrass in a Docker container:<a name="docker-image-unsupported-features"></a>
+ [Connectors](connectors.md) that run in **Greengrass container** mode\. To run a connector in a Docker container, the connector must run in **No container** mode\. To find connectors that support **No container** mode, see [AWS\-provided Greengrass connectors](connectors-list.md)\. Some of these connectors have an isolation mode parameter that you must set to **No container**\.
+ [Local device and volume resources](access-local-resources.md)\. Your user\-defined Lambda functions that run in the Docker container must access devices and volumes on the core directly\.

## Configure the Docker image provided by AWS IoT Greengrass<a name="docker-config-setup-docker-image"></a>

Follow these steps to configure the AWS IoT Greengrass Docker image to run IDT tests\.

**Prerequisities**

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

 

1. Download the Docker image and configure the container\. You can download the prebuilt image from [Docker Hub](https://hub.docker.com/r/amazon/aws-iot-greengrass) or [Amazon Elastic Container Registry](https://docs.aws.amazon.com/AmazonECR/latest/userguide/what-is-ecr.html) \(Amazon ECR\) and run it on Windows, macOS, and Linux \(x86\_64\) platforms\.

   To download the Docker image from Amazon ECR, complete all of the steps in [Step 1: Get the AWS IoT Greengrass container image from Amazon ECR](run-gg-in-docker-container.md#docker-pull-image)\. Then, return to this topic to continue the configuration\.

1. <a name="docker-linux-non-root"></a>Linux users only: Make sure the user that runs IDT has permission to run Docker commands\. For more information, see [Manage Docker as a non\-root user](https://docs.docker.com/install/linux/linux-postinstall/#manage-docker-as-a-non-root-user) in the Docker documentation\.

1. <a name="docker-run-gg-container"></a>To run the AWS IoT Greengrass container, use the command for your operating system:

------
#### [ Linux ]

   ```
   docker run --rm --init -it -d --name aws-iot-greengrass \
   -p 8883:8883 \
   -v <host-path-to-kernel-config-file>:<container-path> \
   <image-repository>:<tag>
   ```
   + Replace *<host\-path\-to\-kernel\-config\-file>* with the path to the kernel configuration file on the host and *<container\-path>* with the path where the volume is mounted in the container\.

     The kernel config file on the host is usually located in `/proc/config.gz` or `/boot/config-<kernel-release-date>`\. You can run `uname -r` to find the *<kernel\-release\-date>* value\.

     **Example:** To mount the config file from `/boot/config-<kernel-release-date>`

     ```
     -v /boot/config-4.15.0-74-generic:/boot/config-4.15.0-74-generic \
     ```

     **Example:** To mount the config file from `proc/config.gz`

     ```
     -v /proc/config.gz:/proc/config.gz \
     ```
   + Replace *<image\-repository>*:*<tag>* in the command with the name of the repository and tag of the target image\.

     **Example:** To point to the latest version of the AWS IoT Greengrass Core software

     ```
     216483018798.dkr.ecr.us-west-2.amazonaws.com/aws-iot-greengrass:latest
     ```

     To get the list of AWS IoT Greengrass Docker images, run the following command\.

     ```
     aws ecr list-images --region us-west-2 --registry-id 216483018798 --repository-name aws-iot-greengrass
     ```

------
#### [ macOS ]

   ```
   docker run --rm --init -it -d --name aws-iot-greengrass \
   -p 8883:8883 \
   <image-repository>:<tag>
   ```
   + Replace *<image\-repository>*:*<tag>* in the command with the name of the repository and tag of the target image\.

     **Example:** To point to the latest version of the AWS IoT Greengrass Core software

     ```
     216483018798.dkr.ecr.us-west-2.amazonaws.com/aws-iot-greengrass:latest
     ```

     To get the list of AWS IoT Greengrass Docker images, run the following command:

     ```
     aws ecr list-images --region us-west-2 --registry-id 216483018798 --repository-name aws-iot-greengrass
     ```

------
#### [ Windows ]

   ```
   docker run --rm --init -it -d --name aws-iot-greengrass \
   -p 8883:8883 \
   <image-repository>:<tag>
   ```
   + Replace *<image\-repository>*:*<tag>* in the command with the name of the repository and tag of the target image\.

     **Example:** To point to the latest version of the AWS IoT Greengrass Core software

     ```
     216483018798.dkr.ecr.us-west-2.amazonaws.com/aws-iot-greengrass:latest
     ```

     To get the list of AWS IoT Greengrass Docker images, run the following command:

     ```
     aws ecr list-images --region us-west-2 --registry-id 216483018798 --repository-name aws-iot-greengrass
     ```

------
**Important**  
When testing with IDT, do not include the `--entrypoint /greengrass-entrypoint.sh \` argument that's used to run the image for general AWS IoT Greengrass use\.

1. <a name="docker-config-next-steps"></a>Next step: [Configure your AWS credentials and `device.json` file](set-config.md)\.

## Configure the dockerfile provided by AWS IoT Greengrass<a name="docker-config-setup-dockerfile"></a>

Follow these steps to configure the Docker image built from the AWS IoT Greengrass Dockerfile to run IDT tests\.

1. From [AWS IoT Greengrass Docker software](what-is-gg.md#gg-docker-download), download the Dockerfile package to your host computer and extract it\.

1. Open `README.md`\. The next three steps refer to sections in this file\.

1. Make sure that you meet the requirements in the **Prerequisites** section\.

1. Linux users only: Complete the **Enable Symlink and Hardlink Protection** and **Enable IPv4 Network Forwarding** steps\.

1. To build the Docker image, complete all of the steps in **Step 1\. Build the AWS IoT Greengrass Docker Image**\. Then, return to this topic to continue the configuration\.

1. <a name="docker-run-gg-container"></a>To run the AWS IoT Greengrass container, use the command for your operating system:

------
#### [ Linux ]

   ```
   docker run --rm --init -it -d --name aws-iot-greengrass \
   -p 8883:8883 \
   -v <host-path-to-kernel-config-file>:<container-path> \
   <image-repository>:<tag>
   ```
   + Replace *<host\-path\-to\-kernel\-config\-file>* with the path to the kernel configuration file on the host and *<container\-path>* with the path where the volume is mounted in the container\.

     The kernel config file on the host is usually located in `/proc/config.gz` or `/boot/config-<kernel-release-date>`\. You can run `uname -r` to find the *<kernel\-release\-date>* value\.

     **Example:** To mount the config file from `/boot/config-<kernel-release-date>`

     ```
     -v /boot/config-4.15.0-74-generic:/boot/config-4.15.0-74-generic \
     ```

     **Example:** To mount the config file from `proc/config.gz`

     ```
     -v /proc/config.gz:/proc/config.gz \
     ```
   + Replace *<image\-repository>*:*<tag>* in the command with the name of the repository and tag of the target image\.

     **Example:** To point to the latest version of the AWS IoT Greengrass Core software

     ```
     216483018798.dkr.ecr.us-west-2.amazonaws.com/aws-iot-greengrass:latest
     ```

     To get the list of AWS IoT Greengrass Docker images, run the following command\.

     ```
     aws ecr list-images --region us-west-2 --registry-id 216483018798 --repository-name aws-iot-greengrass
     ```

------
#### [ macOS ]

   ```
   docker run --rm --init -it -d --name aws-iot-greengrass \
   -p 8883:8883 \
   <image-repository>:<tag>
   ```
   + Replace *<image\-repository>*:*<tag>* in the command with the name of the repository and tag of the target image\.

     **Example:** To point to the latest version of the AWS IoT Greengrass Core software

     ```
     216483018798.dkr.ecr.us-west-2.amazonaws.com/aws-iot-greengrass:latest
     ```

     To get the list of AWS IoT Greengrass Docker images, run the following command:

     ```
     aws ecr list-images --region us-west-2 --registry-id 216483018798 --repository-name aws-iot-greengrass
     ```

------
#### [ Windows ]

   ```
   docker run --rm --init -it -d --name aws-iot-greengrass \
   -p 8883:8883 \
   <image-repository>:<tag>
   ```
   + Replace *<image\-repository>*:*<tag>* in the command with the name of the repository and tag of the target image\.

     **Example:** To point to the latest version of the AWS IoT Greengrass Core software

     ```
     216483018798.dkr.ecr.us-west-2.amazonaws.com/aws-iot-greengrass:latest
     ```

     To get the list of AWS IoT Greengrass Docker images, run the following command:

     ```
     aws ecr list-images --region us-west-2 --registry-id 216483018798 --repository-name aws-iot-greengrass
     ```

------
**Important**  
When testing with IDT, do not include the `--entrypoint /greengrass-entrypoint.sh \` argument that's used to run the image for general AWS IoT Greengrass use\.

1. <a name="docker-config-next-steps"></a>Next step: [Configure your AWS credentials and `device.json` file](set-config.md)\.

## Troubleshooting your Docker container setup for IDT for AWS IoT Greengrass<a name="docker-config-setup-troubleshooting"></a>

Use the following information to help troubleshoot issues with running a Docker container for IDT for AWS IoT Greengrass testing\.

### WARNING: Error loading config file:/home/user/\.docker/config\.json \- stat /home/<user>/\.docker/config\.json: permission denied<a name="docker-config-permissions-linux"></a>

If you get this error when running `docker` commands on Linux, run the following command\. Replace *<user>* in the following command with the user that runs IDT\.

```
sudo chown <user>:<user> /home/<user>/.docker -R
sudo chmod g+rwx /home/<user>/.docker -R
```