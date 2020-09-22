# Docker application deployment connector<a name="docker-app-connector"></a>

The Greengrass Docker application deployment connector makes it easier to run your Docker images on an AWS IoT Greengrass core\. The connector uses Docker Compose to start a multi\-container Docker application from a `docker-compose.yml` file\. Specifically, the connector runs `docker-compose` commands to manage Docker containers on a single core device\. For more information, see [Overview of Docker Compose](https://docs.docker.com/compose/) in the Docker documentation\. The connector can access Docker images stored in Docker container registries, such as Amazon Elastic Container Registry \(Amazon ECR\), Docker Hub, and private Docker trusted registries\.

After you deploy the Greengrass group, the connector pulls the latest images and starts the Docker containers\. It runs the `docker-compose pull` and `docker-compose up` command\. Then, the connector publishes the status of the command to an [output MQTT topic](#docker-app-connector-data-output)\. It also logs status information about running Docker containers\. This makes it possible for you to monitor your application logs in Amazon CloudWatch\. For more information, see [Monitoring with AWS IoT Greengrass logs](greengrass-logs-overview.md)\. The connector also starts Docker containers each time the Greengrass daemon restarts\. The number of Docker containers that can run on the core depends on your hardware\.

The Docker containers run outside of the Greengrass domain on the core device, so they can't access the core's inter\-process communication \(IPC\)\. However, you can configure some communication channels with Greengrass components, such as local Lambda functions\. For more information, see [Communicating with Docker containers](#docker-app-connector-communicating)\.

You can use the connector for scenarios such as hosting a web server or MySQL server on your core device\. Local services in your Docker applications can communicate with each other, other processes in the local environment, and cloud services\. For example, you can run a web server on the core that sends requests from Lambda functions to a web service in the cloud\.

This connector runs in [No container](lambda-group-config.md#no-container-mode) isolation mode, so you can deploy it to a Greengrass group that runs without Greengrass containerization\.

This connector has the following versions\.


| Version | ARN | 
| --- | --- | 
| 6 | `arn:aws:greengrass:region::/connectors/DockerApplicationDeployment/versions/6` | 
| 5 | `arn:aws:greengrass:region::/connectors/DockerApplicationDeployment/versions/5` | 
| 4 | `arn:aws:greengrass:region::/connectors/DockerApplicationDeployment/versions/4` | 
| 3 | `arn:aws:greengrass:region::/connectors/DockerApplicationDeployment/versions/3` | 
| 2 | `arn:aws:greengrass:region::/connectors/DockerApplicationDeployment/versions/2` | 
| 1 | `arn:aws:greengrass:region::/connectors/DockerApplicationDeployment/versions/1` | 

For information about version changes, see the [Changelog](#docker-app-connector-changelog)\.

## Requirements<a name="docker-app-connector-req"></a>

This connector has the following requirements:
+ AWS IoT Greengrass Core software v1\.10 or later\.
**Note**  
This connector is not supported on OpenWrt distributions\.
+ [Python](https://www.python.org/) version 3\.7 installed on the core device and added to the PATH environment variable\.
+ A minimum of 36 MB RAM on the Greengrass core for the connector to monitor running Docker containers\. The total memory requirement depends on the number of Docker containers that run on the core\.
+ [Docker Engine](https://docs.docker.com/install/) 1\.9\.1 or later installed on the Greengrass core\. Version 19\.0\.3 is the latest version that is verified to work with the connector\.

  The `docker` executable must be in the `/usr/bin` or `/usr/local/bin` directory\.
**Important**  
We recommend that you install a credentials store to secure the local copies of your Docker credentials\. For more information, see [Security notes](#docker-app-connector-security)\.

  For information about installing Docker on Amazon Linux distributions, see [Docker basics for Amazon ECS](https://docs.aws.amazon.com/AmazonECS/latest/developerguide/docker-basics.html) in the *Amazon Elastic Container Service Developer Guide*\.
+ [Docker Compose](https://docs.docker.com/compose/install/) installed on the Greengrass core\. The `docker-compose` executable must be in the `/usr/bin` or `/usr/local/bin` directory\.

  The following Docker Compose versions are verified to work with the connector\.    
[\[See the AWS documentation website for more details\]](http://docs.aws.amazon.com/greengrass/latest/developerguide/docker-app-connector.html)
+ A single Docker Compose file \(for example, `docker-compose.yml`\), stored in Amazon S3\. The format must be compatible with the version of Docker Compose installed on the core\. You should test the file before you use it on your core\. If you edit the file after you deploy the Greengrass group, you must redeploy the group to update your local copy on the core\.
+ A Linux user with permission to call the local Docker daemon and write to the directory that stores the local copy of your Compose file\. For more information, see [Setting up the Docker user on the core](#docker-app-connector-linux-user)\.
+ The [Greengrass group role](group-role.md) configured to allow the `s3:GetObject` action on the S3 bucket that contains your Compose file\. This permission is shown in the following example IAM policy\.

  ```
  {
      "Version": "2012-10-17",
      "Statement": [
          {
              "Sid": "AllowAccessToComposeFileS3Bucket",
              "Action": [
                  "s3:GetObject"
              ],
              "Effect": "Allow",
              "Resource": "arn:aws:s3:::bucket-name/*" 
          }
      ]
  }
  ```

  <a name="set-up-group-role"></a>For the group role requirement, you must configure the role to grant the required permissions and make sure the role has been added to the group\. For more information, see [Managing the Greengrass group role \(console\)](group-role.md#manage-group-role-console) or [Managing the Greengrass group role \(CLI\)](group-role.md#manage-group-role-cli)\.
+ <a name="docker-app-connector-ecr-perms"></a>If your Docker Compose file references a Docker image stored in Amazon ECR, the [Greengrass group role](group-role.md) configured to allow the following:
  + `ecr:GetDownloadUrlForLayer` and `ecr:BatchGetImage` actions on your Amazon ECR repositories that contain the Docker images\.
  + `ecr:GetAuthorizationToken` action on your resources\.

  Repositories must be in the same AWS account and AWS Region as the connector\.
**Important**  
Permissions in the group role can be assumed by all Lambda functions and connectors in the Greengrass group\. For more information, see [Security notes](#docker-app-connector-security)\.

  These permissions are shown in the following example policy\.

  ```
  {
      "Version": "2012-10-17",
      "Statement": [
          {
              "Sid": "AllowGetEcrRepositories",
              "Effect": "Allow",
              "Action": [
                  "ecr:GetDownloadUrlForLayer",
                  "ecr:BatchGetImage"
              ],
              "Resource": [
                  "arn:aws:ecr:region:account-id:repository/repository-name"
              ]	
          },
          {
              "Sid": "AllowGetEcrAuthToken",
              "Effect": "Allow",
              "Action": "ecr:GetAuthorizationToken",
              "Resource": "*"
          }
      ]
  }
  ```

  For more information, see [Amazon ECR repository policy examples](https://docs.aws.amazon.com/AmazonECR/latest/userguide/RepositoryPolicyExamples.html) in the *Amazon ECR User Guide*\.

  <a name="set-up-group-role"></a>For the group role requirement, you must configure the role to grant the required permissions and make sure the role has been added to the group\. For more information, see [Managing the Greengrass group role \(console\)](group-role.md#manage-group-role-console) or [Managing the Greengrass group role \(CLI\)](group-role.md#manage-group-role-cli)\.
+ If your Docker Compose file references a Docker image from [AWS Marketplace](https://aws.amazon.com/marketplace), the connector also has the following requirements:
  + You must be subscribed to AWS Marketplace container products\. For more information, see [Finding and subscribing to container products](https://docs.aws.amazon.com/marketplace/latest/buyerguide/buyer-finding-and-subscribing-to-container-products.html) in the *AWS Marketplace Subscribers Guide*\.
  + AWS IoT Greengrass must be configured to support local secrets, as described in [Secrets Requirements](secrets.md#secrets-reqs)\. The connector uses this feature only to retrieve your secrets from AWS Secrets Manager, not to store them\.
  + You must create a secret in Secrets Manager for each AWS Marketplace registry that stores a Docker image referenced in your Compose file\. For more information, see [Accessing Docker images from private repositories](#access-private-repositories)\.
+ If your Docker Compose file references a Docker image from private repositories in registries other than Amazon ECR, such as Docker Hub, the connector also has the following requirements:
  + AWS IoT Greengrass must be configured to support local secrets, as described in [Secrets Requirements](secrets.md#secrets-reqs)\. The connector uses this feature only to retrieve your secrets from AWS Secrets Manager, not to store them\.
  + You must create a secret in Secrets Manager for each private repository that stores a Docker image referenced in your Compose file\. For more information, see [Accessing Docker images from private repositories](#access-private-repositories)\.
+ The Docker daemon must be running when you deploy a Greengrass group that contains this connector\.

### Accessing Docker images from private repositories<a name="access-private-repositories"></a>

If you use credentials to access your Docker images, then you must allow the connector to access them\. The way you do this depends on where the Docker image is located\.

For Docker images stored Amazon ECR, you grant permission to get your authorization token in the Greengrass group role\. For more information, see [Requirements](#docker-app-connector-req)\.

For Docker images stored in other private repositories or registries, you must create a secret in AWS Secrets Manager to store your login information\. This includes Docker images that you subscribed to in AWS Marketplace\. Create one secret for each repository\. If you update your secrets in Secrets Manager, the changes propagate to the core the next time that you deploy the group\.

**Note**  
Secrets Manager is a service that you can use to securely store and manage your credentials, keys, and other secrets in the AWS Cloud\. For more information, see [What is AWS Secrets Manager?](https://docs.aws.amazon.com/secretsmanager/latest/userguide/intro.html) in the *AWS Secrets Manager User Guide*\.

Each secret must contain the following keys:


| Key | Value | 
| --- | --- | 
| `username` | The user name used to access the repository or registry\. | 
| `password` | The password used to access the repository or registry\. | 
| `registryUrl` | The endpoint of the registry\. This must match the corresponding registry URL in the Compose file\. | 

**Note**  
To allow AWS IoT Greengrass to access a secret by default, the name of the secret must start with *greengrass\-*\. Otherwise, your Greengrass service role must grant access\. For more information, see [Allow AWS IoT Greengrass to get secret values](secrets.md#secrets-config-service-role)\.

**To get login information for Docker images from AWS Marketplace**  
Use the `aws ecr get-login` command to get your user name, password, and registry URL for Docker images from AWS Marketplace\.  

```
aws ecr get-login --no-include-email --region region --registry-ids registry-id
```
You can find the registry ID on the container product launch page on the AWS Marketplace website\. Under **Container Images**, choose **View container image details**\.
The output contains the login information that you use to create a secret\. For example, in the following output, the `-u` value is the user name, the `-p` value is the password, and the registry URL is the URL at the end of the output\.  

```
docker login -u AWS -p eyGuYXlsbGkxU0NveDNKaTY4ak...c0MzFyMTIxfQ== https://123456789012.dkr.ecr.region.amazonaws.com
```
Use this login information to create a secret for each AWS Marketplace registry that stores Docker images referenced in your Compose file\. For more information, see [get\-login](https://docs.aws.amazon.com/cli/latest/reference/ecr/get-login.html) in the *AWS CLI Command Reference*\.

**To create secrets \(console\)**  
In the AWS Secrets Manager console, choose **Other type of secrets**\. Under **Specify the key\-value pairs to be stored for this secret**, add rows for `username`, `password`, and `registryUrl`\. For more information, see [Creating a basic secret](https://docs.aws.amazon.com/secretsmanager/latest/userguide/manage_create-basic-secret.html) in the *AWS Secrets Manager User Guide*\.  

![\[Creating a secret with username, password, and registryUrl keys.\]](http://docs.aws.amazon.com/greengrass/latest/developerguide/images/connectors/secret-docker-trusted-registry.png)

**To create secrets \(CLI\)**  
In the AWS CLI, use the Secrets Manager `create-secret` command, as shown in the following example\. For more information, see [create\-secret](https://docs.aws.amazon.com/cli/latest/reference/secretsmanager/create-secret.html) in the *AWS CLI Command Reference*\.  

```
aws secretsmanager create-secret --name greengrass-MySecret --secret-string [{"username":"Mary_Major"},{"password":"abc123xyz456"},{"registryUrl":"https://docker.io"}]
```

**Important**  
It is your responsibility to secure the `DockerComposeFileDestinationPath` directory that stores your Docker Compose file and the credentials for your Docker images from private repositories\. For more information, see [Security notes](#docker-app-connector-security)\.

## Parameters<a name="docker-app-connector-param"></a>

This connector provides the following parameters:

------
#### [ Version 6 ]<a name="docker-app-connector-parameters-v1"></a>

`DockerComposeFileS3Bucket`  
The name of the S3 bucket that contains your Docker Compose file\. When you create the bucket, make sure to follow the [rules for bucket names](https://docs.aws.amazon.com/AmazonS3/latest/dev/BucketRestrictions.html) described in the *Amazon Simple Storage Service Developer Guide*\.  
Display name in the AWS IoT console: **Docker Compose file in S3**  
In the console, the **Docker Compose file in S3** property combines the `DockerComposeFileS3Bucket`, `DockerComposeFileS3Key`, and `DockerComposeFileS3Version` parameters\.
Required: `true`  
Type: `string`  
Valid pattern `[a-zA-Z0-9\\-\\.]{3,63}`

`DockerComposeFileS3Key`  
The object key for your Docker Compose file in Amazon S3\. For more information, including object key naming guidelines, see [Object key and metadata](https://docs.aws.amazon.com/AmazonS3/latest/dev/UsingMetadata.html) in the *Amazon Simple Storage Service Developer Guide*\.  
In the console, the **Docker Compose file in S3** property combines the `DockerComposeFileS3Bucket`, `DockerComposeFileS3Key`, and `DockerComposeFileS3Version` parameters\.
Required: `true`  
Type: `string`  
Valid pattern `.+`

`DockerComposeFileS3Version`  
The object version for your Docker Compose file in Amazon S3\. For more information, including object key naming guidelines, see [Using versioning](https://docs.aws.amazon.com/AmazonS3/latest/dev/Versioning.html) in the *Amazon Simple Storage Service Developer Guide*\.  
In the console, the **Docker Compose file in S3** property combines the `DockerComposeFileS3Bucket`, `DockerComposeFileS3Key`, and `DockerComposeFileS3Version` parameters\.
Required: `false`  
Type: `string`  
Valid pattern `.+`

`DockerComposeFileDestinationPath`  
The absolute path of the local directory used to store a copy of the Docker Compose file\. This must be an existing directory\. The user specified for `DockerUserId` must have permission to create a file in this directory\. For more information, see [Setting up the Docker user on the AWS IoT Greengrass core](#docker-app-connector-linux-user)\.  
This directory stores your Docker Compose file and the credentials for your Docker images from private repositories\. It is your responsibility to secure this directory\. For more information, see [Security notes](#docker-app-connector-security)\.
Display name in the AWS IoT console: **Directory path for local Compose file**  
Required: `true`  
Type: `string`  
Valid pattern `\/.*\/?`  
Example: `/home/username/myCompose`

`DockerUserId`  
The UID of the Linux user that the connector runs as\. This user must belong to the `docker` Linux group on the core device and have write permissions to the `DockerComposeFileDestinationPath` directory\. For more information, see [Setting up the Docker user on the core](#docker-app-connector-linux-user)\.  
<a name="avoid-running-as-root"></a>We recommend that you avoid running as root unless absolutely necessary\. If you do specify the root user, you must allow Lambda functions to run as root on the AWS IoT Greengrass core\. For more information, see [Running a Lambda function as root](lambda-group-config.md#lambda-running-as-root)\.
Display name in the AWS IoT console: **Docker user ID**  
Required: `false`  
Type: `string`  
Valid pattern: `^[0-9]{1,5}$`

`AWSSecretsArnList`  
The Amazon Resource Names \(ARNs\) of the secrets in AWS Secrets Manager that contain the login information used to access your Docker images in private repositories\. For more information, see [Accessing Docker images from private repositories](#access-private-repositories)\.  
Display name in the AWS IoT console: **Credentials for private repositories**  
Required: `false`\. This parameter is required to access Docker images stored in private repositories\.  
Type: `array` of `string`  
Valid pattern: `[( ?,? ?"(arn:(aws(-[a-z]+)):secretsmanager:[a-z0-9-]+:[0-9]{12}:secret:([a-zA-Z0-9\]+/)[a-zA-Z0-9/_+=,.@-]+-[a-zA-Z0-9]+)")]`

`DockerContainerStatusLogFrequency`  
The frequency \(in seconds\) at which the connector logs status information about the Docker containers running on the core\. The default is 300 seconds \(5 minutes\)\.  
Display name in the AWS IoT console: **Logging frequency**  
Required: `false`  
Type: `string`  
Valid pattern: `^[1-9]{1}[0-9]{0,3}$`

`ForceDeploy`  
Indicates whether to force the Docker deployment if it fails due to the improper cleanup of the last deployment\. The default is `False`\.  
Display name in the AWS IoT console: **Force deployment**  
Required: `false`  
Type: `string`  
Valid pattern: `^(true|false)$`

`DockerPullBeforeUp`  
Indicates whether the deployer should run `docker-compose pull` before running `docker-compose up` for a pull\-down\-up behavior\. The default is `True`\.  
Display name in the AWS IoT console: **Docker Pull Before Up**  
Required: `false`  
Type: `string`  
Valid pattern: `^(true|false)$`

`StopContainersOnNewDeployment`  
Indicates whether the connector should stop Docker Deployer managed docker containers when GGC is stopped \(when a new group deployment is made, or the kernel is shutdown\)\. The default is `True`\.  
Display name in the AWS IoT console: **Docker stop on new deployment**  
 We recommend leaving this parameter to its default `True` value\. Setting this parameter to `False` will cause your Docker container to continue running even after terminating the AWS IoT Greengrass core or starting a new deployment\. If you set this parameter to `False`, you must ensure that your Docker containers are maintained as necessary in the event of a `docker-compose` service name change or addition\.   
 For more information please refer to the `docker-compose` compose file documentation\. 
Required: `false`  
Type: `string`  
Valid pattern: `^(true|false)$`

------
#### [ Version 5 ]<a name="docker-app-connector-parameters-v1"></a>

`DockerComposeFileS3Bucket`  
The name of the S3 bucket that contains your Docker Compose file\. When you create the bucket, make sure to follow the [rules for bucket names](https://docs.aws.amazon.com/AmazonS3/latest/dev/BucketRestrictions.html) described in the *Amazon Simple Storage Service Developer Guide*\.  
Display name in the AWS IoT console: **Docker Compose file in S3**  
In the console, the **Docker Compose file in S3** property combines the `DockerComposeFileS3Bucket`, `DockerComposeFileS3Key`, and `DockerComposeFileS3Version` parameters\.
Required: `true`  
Type: `string`  
Valid pattern `[a-zA-Z0-9\\-\\.]{3,63}`

`DockerComposeFileS3Key`  
The object key for your Docker Compose file in Amazon S3\. For more information, including object key naming guidelines, see [Object key and metadata](https://docs.aws.amazon.com/AmazonS3/latest/dev/UsingMetadata.html) in the *Amazon Simple Storage Service Developer Guide*\.  
In the console, the **Docker Compose file in S3** property combines the `DockerComposeFileS3Bucket`, `DockerComposeFileS3Key`, and `DockerComposeFileS3Version` parameters\.
Required: `true`  
Type: `string`  
Valid pattern `.+`

`DockerComposeFileS3Version`  
The object version for your Docker Compose file in Amazon S3\. For more information, including object key naming guidelines, see [Using versioning](https://docs.aws.amazon.com/AmazonS3/latest/dev/Versioning.html) in the *Amazon Simple Storage Service Developer Guide*\.  
In the console, the **Docker Compose file in S3** property combines the `DockerComposeFileS3Bucket`, `DockerComposeFileS3Key`, and `DockerComposeFileS3Version` parameters\.
Required: `false`  
Type: `string`  
Valid pattern `.+`

`DockerComposeFileDestinationPath`  
The absolute path of the local directory used to store a copy of the Docker Compose file\. This must be an existing directory\. The user specified for `DockerUserId` must have permission to create a file in this directory\. For more information, see [Setting up the Docker user on the AWS IoT Greengrass core](#docker-app-connector-linux-user)\.  
This directory stores your Docker Compose file and the credentials for your Docker images from private repositories\. It is your responsibility to secure this directory\. For more information, see [Security notes](#docker-app-connector-security)\.
Display name in the AWS IoT console: **Directory path for local Compose file**  
Required: `true`  
Type: `string`  
Valid pattern `\/.*\/?`  
Example: `/home/username/myCompose`

`DockerUserId`  
The UID of the Linux user that the connector runs as\. This user must belong to the `docker` Linux group on the core device and have write permissions to the `DockerComposeFileDestinationPath` directory\. For more information, see [Setting up the Docker user on the core](#docker-app-connector-linux-user)\.  
<a name="avoid-running-as-root"></a>We recommend that you avoid running as root unless absolutely necessary\. If you do specify the root user, you must allow Lambda functions to run as root on the AWS IoT Greengrass core\. For more information, see [Running a Lambda function as root](lambda-group-config.md#lambda-running-as-root)\.
Display name in the AWS IoT console: **Docker user ID**  
Required: `false`  
Type: `string`  
Valid pattern: `^[0-9]{1,5}$`

`AWSSecretsArnList`  
The Amazon Resource Names \(ARNs\) of the secrets in AWS Secrets Manager that contain the login information used to access your Docker images in private repositories\. For more information, see [Accessing Docker images from private repositories](#access-private-repositories)\.  
Display name in the AWS IoT console: **Credentials for private repositories**  
Required: `false`\. This parameter is required to access Docker images stored in private repositories\.  
Type: `array` of `string`  
Valid pattern: `[( ?,? ?"(arn:(aws(-[a-z]+)):secretsmanager:[a-z0-9-]+:[0-9]{12}:secret:([a-zA-Z0-9\]+/)[a-zA-Z0-9/_+=,.@-]+-[a-zA-Z0-9]+)")]`

`DockerContainerStatusLogFrequency`  
The frequency \(in seconds\) at which the connector logs status information about the Docker containers running on the core\. The default is 300 seconds \(5 minutes\)\.  
Display name in the AWS IoT console: **Logging frequency**  
Required: `false`  
Type: `string`  
Valid pattern: `^[1-9]{1}[0-9]{0,3}$`

`ForceDeploy`  
Indicates whether to force the Docker deployment if it fails due to the improper cleanup of the last deployment\. The default is `False`\.  
Display name in the AWS IoT console: **Force deployment**  
Required: `false`  
Type: `string`  
Valid pattern: `^(true|false)$`

`DockerPullBeforeUp`  
Indicates whether the deployer should run `docker-compose pull` before running `docker-compose up` for a pull\-down\-up behavior\. The default is `True`\.  
Display name in the AWS IoT console: **Docker Pull Before Up**  
Required: `false`  
Type: `string`  
Valid pattern: `^(true|false)$`

------
#### [ Versions 2 \- 4 ]<a name="docker-app-connector-parameters-v1"></a>

`DockerComposeFileS3Bucket`  
The name of the S3 bucket that contains your Docker Compose file\. When you create the bucket, make sure to follow the [rules for bucket names](https://docs.aws.amazon.com/AmazonS3/latest/dev/BucketRestrictions.html) described in the *Amazon Simple Storage Service Developer Guide*\.  
Display name in the AWS IoT console: **Docker Compose file in S3**  
In the console, the **Docker Compose file in S3** property combines the `DockerComposeFileS3Bucket`, `DockerComposeFileS3Key`, and `DockerComposeFileS3Version` parameters\.
Required: `true`  
Type: `string`  
Valid pattern `[a-zA-Z0-9\\-\\.]{3,63}`

`DockerComposeFileS3Key`  
The object key for your Docker Compose file in Amazon S3\. For more information, including object key naming guidelines, see [Object key and metadata](https://docs.aws.amazon.com/AmazonS3/latest/dev/UsingMetadata.html) in the *Amazon Simple Storage Service Developer Guide*\.  
In the console, the **Docker Compose file in S3** property combines the `DockerComposeFileS3Bucket`, `DockerComposeFileS3Key`, and `DockerComposeFileS3Version` parameters\.
Required: `true`  
Type: `string`  
Valid pattern `.+`

`DockerComposeFileS3Version`  
The object version for your Docker Compose file in Amazon S3\. For more information, including object key naming guidelines, see [Using versioning](https://docs.aws.amazon.com/AmazonS3/latest/dev/Versioning.html) in the *Amazon Simple Storage Service Developer Guide*\.  
In the console, the **Docker Compose file in S3** property combines the `DockerComposeFileS3Bucket`, `DockerComposeFileS3Key`, and `DockerComposeFileS3Version` parameters\.
Required: `false`  
Type: `string`  
Valid pattern `.+`

`DockerComposeFileDestinationPath`  
The absolute path of the local directory used to store a copy of the Docker Compose file\. This must be an existing directory\. The user specified for `DockerUserId` must have permission to create a file in this directory\. For more information, see [Setting up the Docker user on the AWS IoT Greengrass core](#docker-app-connector-linux-user)\.  
This directory stores your Docker Compose file and the credentials for your Docker images from private repositories\. It is your responsibility to secure this directory\. For more information, see [Security notes](#docker-app-connector-security)\.
Display name in the AWS IoT console: **Directory path for local Compose file**  
Required: `true`  
Type: `string`  
Valid pattern `\/.*\/?`  
Example: `/home/username/myCompose`

`DockerUserId`  
The UID of the Linux user that the connector runs as\. This user must belong to the `docker` Linux group on the core device and have write permissions to the `DockerComposeFileDestinationPath` directory\. For more information, see [Setting up the Docker user on the core](#docker-app-connector-linux-user)\.  
<a name="avoid-running-as-root"></a>We recommend that you avoid running as root unless absolutely necessary\. If you do specify the root user, you must allow Lambda functions to run as root on the AWS IoT Greengrass core\. For more information, see [Running a Lambda function as root](lambda-group-config.md#lambda-running-as-root)\.
Display name in the AWS IoT console: **Docker user ID**  
Required: `false`  
Type: `string`  
Valid pattern: `^[0-9]{1,5}$`

`AWSSecretsArnList`  
The Amazon Resource Names \(ARNs\) of the secrets in AWS Secrets Manager that contain the login information used to access your Docker images in private repositories\. For more information, see [Accessing Docker images from private repositories](#access-private-repositories)\.  
Display name in the AWS IoT console: **Credentials for private repositories**  
Required: `false`\. This parameter is required to access Docker images stored in private repositories\.  
Type: `array` of `string`  
Valid pattern: `[( ?,? ?"(arn:(aws(-[a-z]+)):secretsmanager:[a-z0-9-]+:[0-9]{12}:secret:([a-zA-Z0-9\]+/)[a-zA-Z0-9/_+=,.@-]+-[a-zA-Z0-9]+)")]`

`DockerContainerStatusLogFrequency`  
The frequency \(in seconds\) at which the connector logs status information about the Docker containers running on the core\. The default is 300 seconds \(5 minutes\)\.  
Display name in the AWS IoT console: **Logging frequency**  
Required: `false`  
Type: `string`  
Valid pattern: `^[1-9]{1}[0-9]{0,3}$`

`ForceDeploy`  
Indicates whether to force the Docker deployment if it fails due to the improper cleanup of the last deployment\. The default is `False`\.  
Display name in the AWS IoT console: **Force deployment**  
Required: `false`  
Type: `string`  
Valid pattern: `^(true|false)$`

------
#### [ Version 1 ]<a name="docker-app-connector-parameters-v1"></a>

`DockerComposeFileS3Bucket`  
The name of the S3 bucket that contains your Docker Compose file\. When you create the bucket, make sure to follow the [rules for bucket names](https://docs.aws.amazon.com/AmazonS3/latest/dev/BucketRestrictions.html) described in the *Amazon Simple Storage Service Developer Guide*\.  
Display name in the AWS IoT console: **Docker Compose file in S3**  
In the console, the **Docker Compose file in S3** property combines the `DockerComposeFileS3Bucket`, `DockerComposeFileS3Key`, and `DockerComposeFileS3Version` parameters\.
Required: `true`  
Type: `string`  
Valid pattern `[a-zA-Z0-9\\-\\.]{3,63}`

`DockerComposeFileS3Key`  
The object key for your Docker Compose file in Amazon S3\. For more information, including object key naming guidelines, see [Object key and metadata](https://docs.aws.amazon.com/AmazonS3/latest/dev/UsingMetadata.html) in the *Amazon Simple Storage Service Developer Guide*\.  
In the console, the **Docker Compose file in S3** property combines the `DockerComposeFileS3Bucket`, `DockerComposeFileS3Key`, and `DockerComposeFileS3Version` parameters\.
Required: `true`  
Type: `string`  
Valid pattern `.+`

`DockerComposeFileS3Version`  
The object version for your Docker Compose file in Amazon S3\. For more information, including object key naming guidelines, see [Using versioning](https://docs.aws.amazon.com/AmazonS3/latest/dev/Versioning.html) in the *Amazon Simple Storage Service Developer Guide*\.  
In the console, the **Docker Compose file in S3** property combines the `DockerComposeFileS3Bucket`, `DockerComposeFileS3Key`, and `DockerComposeFileS3Version` parameters\.
Required: `false`  
Type: `string`  
Valid pattern `.+`

`DockerComposeFileDestinationPath`  
The absolute path of the local directory used to store a copy of the Docker Compose file\. This must be an existing directory\. The user specified for `DockerUserId` must have permission to create a file in this directory\. For more information, see [Setting up the Docker user on the AWS IoT Greengrass core](#docker-app-connector-linux-user)\.  
This directory stores your Docker Compose file and the credentials for your Docker images from private repositories\. It is your responsibility to secure this directory\. For more information, see [Security notes](#docker-app-connector-security)\.
Display name in the AWS IoT console: **Directory path for local Compose file**  
Required: `true`  
Type: `string`  
Valid pattern `\/.*\/?`  
Example: `/home/username/myCompose`

`DockerUserId`  
The UID of the Linux user that the connector runs as\. This user must belong to the `docker` Linux group on the core device and have write permissions to the `DockerComposeFileDestinationPath` directory\. For more information, see [Setting up the Docker user on the core](#docker-app-connector-linux-user)\.  
<a name="avoid-running-as-root"></a>We recommend that you avoid running as root unless absolutely necessary\. If you do specify the root user, you must allow Lambda functions to run as root on the AWS IoT Greengrass core\. For more information, see [Running a Lambda function as root](lambda-group-config.md#lambda-running-as-root)\.
Display name in the AWS IoT console: **Docker user ID**  
Required: `false`  
Type: `string`  
Valid pattern: `^[0-9]{1,5}$`

`AWSSecretsArnList`  
The Amazon Resource Names \(ARNs\) of the secrets in AWS Secrets Manager that contain the login information used to access your Docker images in private repositories\. For more information, see [Accessing Docker images from private repositories](#access-private-repositories)\.  
Display name in the AWS IoT console: **Credentials for private repositories**  
Required: `false`\. This parameter is required to access Docker images stored in private repositories\.  
Type: `array` of `string`  
Valid pattern: `[( ?,? ?"(arn:(aws(-[a-z]+)):secretsmanager:[a-z0-9-]+:[0-9]{12}:secret:([a-zA-Z0-9\]+/)[a-zA-Z0-9/_+=,.@-]+-[a-zA-Z0-9]+)")]`

`DockerContainerStatusLogFrequency`  
The frequency \(in seconds\) at which the connector logs status information about the Docker containers running on the core\. The default is 300 seconds \(5 minutes\)\.  
Display name in the AWS IoT console: **Logging frequency**  
Required: `false`  
Type: `string`  
Valid pattern: `^[1-9]{1}[0-9]{0,3}$`

------

### Create Connector Example \(AWS CLI\)<a name="docker-app-connector-create"></a>

The following CLI command creates a `ConnectorDefinition` with an initial version that contains the Greengrass Docker application deployment connector\.

```
aws greengrass create-connector-definition --name MyGreengrassConnectors --initial-version '{
    "Connectors": [
        {
            "Id": "MyDockerAppplicationDeploymentConnector",
            "ConnectorArn": "arn:aws:greengrass:region::/connectors/DockerApplicationDeployment/versions/5",
            "Parameters": {
                "DockerComposeFileS3Bucket": "myS3Bucket",
                "DockerComposeFileS3Key": "production-docker-compose.yml",
                "DockerComposeFileS3Version": "123",
                "DockerComposeFileDestinationPath": "/home/username/myCompose",
                "DockerUserId": "1000",
                "AWSSecretsArnList": "[\"arn:aws:secretsmanager:region:account-id:secret:greengrass-secret1-hash\",\"arn:aws:secretsmanager:region:account-id:secret:greengrass-secret2-hash\"]",
                "DockerContainerStatusLogFrequency": "30",
                "ForceDeploy": "True",
                "DockerPullBeforeUp": "True"
            }
        }
    ]
}'
```

**Note**  
The Lambda function in this connector has a [long\-lived](lambda-functions.md#lambda-lifecycle) lifecycle\.

## Input data<a name="docker-app-connector-data-input"></a>

This connector doesn't require or accept input data\.

## Output data<a name="docker-app-connector-data-output"></a>

This connector publishes the status of the `docker-compose up` command as output data\.

<a name="topic-filter"></a>**Topic filter in subscription**  
`dockerapplicationdeploymentconnector/message/status`

**Example output: Success**  

```
{
  "status":"success",
  "GreengrassDockerApplicationDeploymentStatus":"Successfully triggered docker-compose up", 
  "S3Bucket":"myS3Bucket",
  "ComposeFileName":"production-docker-compose.yml",
  "ComposeFileVersion":"123"
}
```

**Example output: Failure**  

```
{
  "status":"fail",
  "error_message":"description of error",
  "error":"InvalidParameter"
}
```
The error type can be `InvalidParameter` or `InternalError`\.

## Setting up the Docker user on the AWS IoT Greengrass core<a name="docker-app-connector-linux-user"></a>

The Greengrass Docker application deployment connector runs as the user you specify for the `DockerUserId` parameter\. If you don't specify a value, the connector runs as `ggc_user`, which is the default Greengrass access identity\.

To allow the connector to interact with the Docker daemon, the Docker user must belong to the `docker` Linux group on the core\. The Docker user must also have write permissions to the `DockerComposeFileDestinationPath` directory\. This is where the connector stores your local `docker-compose.yml` file and Docker credentials\.

**Note**  
We recommend that you create a Linux user instead of using the default `ggc_user`\. Otherwise, any Lambda function in the Greengrass group can access the Compose file and Docker credentials\.
<a name="avoid-running-as-root"></a>We recommend that you avoid running as root unless absolutely necessary\. If you do specify the root user, you must allow Lambda functions to run as root on the AWS IoT Greengrass core\. For more information, see [Running a Lambda function as root](lambda-group-config.md#lambda-running-as-root)\.

1. Create the user\. You can run the `useradd` command and include the optional `-u` option to assign a UID\. For example:

   ```
   sudo useradd -u 1234 user-name
   ```

1. Add the user to the `docker` group on the core\. For example:

   ```
   sudo usermod -aG docker user-name
   ```

   For more information, including how to create the `docker` group, see [Manage Docker as a non\-root user](https://docs.docker.com/install/linux/linux-postinstall/#manage-docker-as-a-non-root-user) in the Docker documentation\.

1. Give the user permissions to write to the directory specifed for the `DockerComposeFileDestinationPath` parameter\. For example:

   1. To set the user as the owner of the directory\. This example uses the UID from step 1\.

      ```
      chown 1234 docker-compose-file-destination-path
      ```

   1. To give read and write permissions to the owner\.

      ```
      chmod 700 docker-compose-file-destination-path
      ```

      For more information, see [How To Manage File And Folder Permissions In Linux](https://www.linux.com/tutorials/how-manage-file-and-folder-permissions-linux/) in the Linux Foundation documentation\.

   1. If you didn't assign a UID when you created the user, or if you used an existing user, run the `id` command to look up the UID\.

      ```
      id -u user-name
      ```

      You use the UID to configure the `DockerUserId` parameter for the connector\.

## Usage information<a name="docker-app-connector-usage-info"></a>

When you use the Greengrass Docker application deployment connector, you should be aware of the following implementation\-specific usage information\.
+ Fixed prefix for project names\. The connector prepends the `greengrassdockerapplicationdeployment` prefix to the names of the Docker containers that it starts\. The connector uses this prefix as the project name in the `docker-compose` commands that it runs\.
+ Logging behavior\. The connector writes status information and troubleshooting information to a log file\. You can configure AWS IoT Greengrass to send logs to CloudWatch Logs and to write logs locally\. For more information, see [Logging for connectors](connectors.md#connectors-logging)\. This is the path to the local log for the connector:

  ```
  /greengrass-root/ggc/var/log/user/region/aws/DockerApplicationDeployment.log
  ```

  You must have root permissions to access local logs\.
+ Updating Docker images\. Docker caches images on the core device\. If you update a Docker image and want to propagate the change to the core device, make sure to change the tag for the image in the Compose file\. Changes take effect after the Greengrass group is deployed\.
+ 10\-minute timeout for cleanup operations\. When the Greengrass daemon stops \(during a restart\), the `docker-compose down` command is triggered\. All Docker containers have a maximum of 10 minutes after `docker-compose down` is triggered to perform any cleanup operations\. If the cleanup isn't complete in 10 minutes, you must clean up the remaining containers manually\. For more information, see [docker rm](https://docs.docker.com/engine/reference/commandline/rm/) in the Docker CLI documentation\.
+ Running Docker commands\. To troubleshoot issues, you can run Docker commands in a terminal window on the core device\. For example, run the following command to see the Docker containers that were started by the connector:

  ```
  docker ps --filter name="greengrassdockerapplicationdeployment"
  ```
+ Reserved resource ID\. The connector uses the `DOCKER_DEPLOYER_SECRET_RESOURCE_RESERVED_ID_index` ID for the Greengrass resources it creates in the Greengrass group\. Resource IDs must be unique in the group, so don't assign a resource ID that might conflict with this reserved resource ID\.

## Communicating with Docker containers<a name="docker-app-connector-communicating"></a>

AWS IoT Greengrass supports the following communication channels between Greengrass components and Docker containers:
+ Greengrass Lambda functions can use REST APIs to communicate with processes in Docker containers\. You can set up a server in a Docker container that opens a port\. Lambda functions can communicate with the container on this port\.
+ Processes in Docker containers can exchange MQTT messages through the local Greengrass message broker\. You can set up the Docker container as a Greengrass device in the Greengrass group and then create subscriptions to allow the container to communicate with Greengrass Lambda functions, devices, and other connectors in the group, or with AWS IoT and the local shadow service\. For more information, see [Configure MQTT communication with Docker containers](#docker-app-connector-mqtt-communication)\.
+ Greengrass Lambda functions can update a shared file to pass information to Docker containers\. You can use the Compose file to bind mount the shared file path for a Docker container\.

### Configure MQTT communication with Docker containers<a name="docker-app-connector-mqtt-communication"></a>

You can configure a Docker container as a Greengrass device and add it to a Greengrass group\. Then, you can create subscriptions that allow MQTT communication between the Docker container and Greengrass components or AWS IoT\. In the following procedure, you create a subscription that allows the Docker container device to receive shadow update messages from the local shadow service\. You can follow this pattern to create other subscriptions\.

**Note**  
In this procedure, we assume you have already created a Greengrass group and a Greengrass core \(v1\.10 or later\)\. To learn how to create a Greengrass group and core, see [Getting started with AWS IoT Greengrass](gg-gs.md)\.

**To configure a Docker container as a Greengrass device and add it to a Greengrass group**

1. Create a directory on the core device to store the certificates and keys used to authenticate the Greengrass device\.

   The file path must be mounted on the Docker container you want to start\. The following snippet shows how to mount a file path in your Compose file\. In this example, *path\-to\-device\-certs* represents the directory you created in this step\.

   ```
   version: '3.3'
   services:
     myService:
       image: user-name/repo:image-tag
       volumes:
         -  /path-to-device-certs/:/path-accessible-in-container
   ```

1. <a name="console-gg-groups"></a>In the AWS IoT console, choose **Greengrass**, and then choose **Groups**\.

1. <a name="group-choose-target-group"></a>Choose the target group\.

1. <a name="gg-group-add-device"></a>On the group configuration page, choose **Devices**, and then choose **Add Device**\.  
![\[Screenshot of the Devices page with the Add your first Device button highlighted.\]](http://docs.aws.amazon.com/greengrass/latest/developerguide/images/gg-get-started-066.png)

1. <a name="gg-group-create-device"></a>On the **Add a Device** page, choose **Create New Device**\.

1. On the **Create a Registry entry for a device** page, enter a name for the device, and then choose **Next**\.

1. <a name="gg-group-generate-default-device-certs"></a>On the **Set up security** page, for **1\-Click**, choose **Use Defaults**\. This option generates a device certificate with an attached [AWS IoT policy](https://docs.aws.amazon.com/iot/latest/developerguide/iot-policies.html) and public and private key\.

1. On the **Download security credentials** page, download the certificates and keys to the directory you created in step 1, and then choose **Finish**\.  
![\[Screenshot of the Download security credentials page with the Download these resources as a tar.gz button highlighted.\]](http://docs.aws.amazon.com/greengrass/latest/developerguide/images/gg-get-started-070.png)

1. Decompress the `hash-setup.tar.gz` file\. For example, run the following command\. The *hash* placeholder is the hash in the name of the `tar.gz` file you downloaded \(for example, `bcc5afd26d`\)\.

   ```
   cd /path-to-device-certs
   tar -xzf hash-setup.tar.gz
   ```

1. Review [Server Authentication](https://docs.aws.amazon.com/iot/latest/developerguide/server-authentication.html) in the *AWS IoT Developer Guide* and choose the appropriate root CA certificate\. We recommend that you use Amazon Trust Services \(ATS\) endpoints and ATS root CA certificates\.
**Important**  
Your root CA certificate type must match your endpoint\. Use an ATS root CA certificate with an ATS endpoint \(preferred\) or a VeriSign root CA certificate with a legacy endpoint\. Only some AWS Regions support legacy endpoints\. For more information, see [Service endpoints must match the root CA certificate type](gg-core.md#certificate-endpoints)\.

   Download the appropriate ATS root CA certificate to the core device\. For example, you can use the following `wget` commands to download `AmazonRootCA1.pem` to your file path\.

   ```
   cd /path-to-device-certs
   sudo wget -O root.ca.pem https://www.amazontrust.com/repository/AmazonRootCA1.pem
   ```

Next, create a subscription in the group\. For this example, you create a subscription allows the Docker container device to receive MQTT messages from the local shadow service\.

**Note**  
The maximum size of a shadow document is 8 KB\. For more information, see [AWS IoT quotas](https://docs.aws.amazon.com/iot/latest/developerguide/limits-iot.html) in the *AWS IoT Developer Guide*\.

**To create a subscription that allows the Docker container device to receive MQTT messages from the local shadow service**

1. <a name="shared-subscriptions-addsubscription"></a>On the group configuration page, choose **Subscriptions**, and then choose **Add Subscription**\.  
![\[The group page with Subscriptions and Add Subscription highlighted.\]](http://docs.aws.amazon.com/greengrass/latest/developerguide/images/console-group-subscriptions.png)

1. On the **Select your source and target** page, configure the source and target, as follows:

   1. For **Select a source**, choose **Services**, and then choose **Local Shadow Service**\.

   1. For **Select a target**, choose **Devices**, and then choose your device\.

   1. Choose **Next**\.

   1. On the **Filter your data with a topic** page, for **Topic filter**, choose **$aws/things/TestCore/shadow/update/accepted**, and then choose **Next**\.

   1. Choose **Finish**\.

Include the following code snippet in the Docker image that you reference in your Compose file\. This is the Greengrass device code\. Also, add code in your Docker container that starts the Greengrass device inside the container\. It can run as a separate process in the image or in a separate thread\.

```
from AWSIoTPythonSDK.core.greengrass.discovery.providers import DiscoveryInfoProvider

# Discover Greengrass cores.
discoveryInfoProvider = DiscoveryInfoProvider()
discoveryInfoProvider.configureEndpoint(host)

# Configure these paths based on the download location of the certificates.
discoveryInfoProvider.configureCredentials(rootCAPath, certificatePath, privateKeyPath)
discoveryInfoProvider.configureTimeout(10)  # 10 seconds.

# Get discovery info from AWS IoT.

# thingName is the name you registered for the device.
discoveryInfo = discoveryInfoProvider.discover(thingName)
caList = discoveryInfo.getAllCas()
coreList = discoveryInfo.getAllCores()

# Try to connect to the Greengrass core.
for connectivityInfo in coreInfo.connectivityInfoList:
    currentHost = connectivityInfo.host
    currentPort = connectivityInfo.port
    myAWSIoTMQTTClient.configureEndpoint(currentHost, currentPort)
    try:
    	myAWSIoTMQTTClient.connect()
	    connected = True
        break
	except BaseException as e:
		print("Error in connect!")
if not connected:
	print("Cannot connect to core %s. Exiting..." % coreInfo.coreThingArn)
    sys.exit(-2)

# Handle the MQTT message received from GGShadowService.
def customCallback(client, userdata, message):
    print("Received a message on MQTT")
    print(message)

# Subscribe to the MQTT topic.

# The topic is the "$aws/things/TestCore/shadow/update/accepted".
myAWSIoTMQTTClient.subscribe(topic, 1, customCallback)

# Keep the process alive to listen for messages.
while True:
	time.sleep(1)
```

## Security notes<a name="docker-app-connector-security"></a>

When you use the Greengrass Docker application deployment connector, be aware of the following security considerations\.

  
**Local storage of the Docker Compose file**  
The connector stores a copy of your Compose file in the directory specified for the `DockerComposeFileDestinationPath` parameter\.  
It's your responsibility to secure this directory\. You should use file system permissions to restrict access to the directory\.

  
**Local storage of the Docker credentials**  
If your Docker images are stored in private repositories, the connector stores your Docker credentials in the directory specified for the `DockerComposeFileDestinationPath` parameter\.  
It's your responsibility to secure these credentials\. For example, you should use [credential\-helper](https://docs.docker.com/engine/reference/commandline/login/#credentials-store) on the core device when you install Docker Engine\.

  
**Install Docker Engine from a trusted source**  
It's your responsibility to install Docker Engine from a trusted source\. This connector uses the Docker daemon on the core device to access your Docker assets and manage Docker containers\.

  
**Scope of Greengrass group role permissions**  
Permissions that you add in the Greengrass group role can be assumed by all Lambda functions and connectors in the Greengrass group\. This connector requires access to your Docker Compose file stored in an S3 bucket\. It also requires access to your Amazon ECR authorization token if your Docker images are stored in a private repository in Amazon ECR\.

## Licenses<a name="docker-app-connector-license"></a>

The Greengrass Docker application deployment connector includes the following third\-party software/licensing:<a name="boto-3-licenses"></a>
+ [AWS SDK for Python \(Boto3\)](https://pypi.org/project/boto3/)/Apache License 2\.0
+ [botocore](https://pypi.org/project/botocore/)/Apache License 2\.0
+ [dateutil](https://pypi.org/project/python-dateutil/1.4/)/PSF License
+ [docutils](https://pypi.org/project/docutils/)/BSD License, GNU General Public License \(GPL\), Python Software Foundation License, Public Domain
+ [jmespath](https://pypi.org/project/jmespath/)/MIT License
+ [s3transfer](https://pypi.org/project/s3transfer/)/Apache License 2\.0
+ [urllib3](https://pypi.org/project/urllib3/)/MIT License

This connector is released under the [Greengrass Core Software License Agreement](https://greengrass-release-license.s3.us-west-2.amazonaws.com/greengrass-license-v1.pdf)\.

## Changelog<a name="docker-app-connector-changelog"></a>

The following table describes the changes in each version of the connector\.


| Version | Changes | 
| --- | --- | 
| 6 | Added `StopContainersOnNewDeployment` to override container clean up when a new deployment is made or GGC stops\. Safer shutdown and start up mechanisms\. YAML validation bug fix\. | 
| 5 | Images are pulled before running `docker-compose down`\. | 
| 4 | Added pull\-before\-up behavior to update Docker images\. | 
| 3 | Fixed an issue with finding environment variables\. | 
| 2 | Added the `ForceDeploy` parameter\. | 
| 1 | Initial release\.  | 

<a name="one-conn-version"></a>A Greengrass group can contain only one version of the connector at a time\. For information about upgrading a connector version, see [Upgrading connector versions](connectors.md#upgrade-connector-versions)\.

## See also<a name="docker-app-connector-see-also"></a>
+ [Integrate with services and protocols using Greengrass connectors](connectors.md)
+ [Getting started with Greengrass connectors \(console\)](connectors-console.md)
+ [Getting started with Greengrass connectors \(CLI\)](connectors-cli.md)