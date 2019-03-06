# OTA Updates of AWS IoT Greengrass Core Software<a name="core-ota-update"></a>

This feature is available for AWS IoT Greengrass Core v1\.3 and later\.

The AWS IoT Greengrass core software comes packaged with an OTA Update Agent that is capable of updating the core's software or the OTA Update Agent itself to the latest respective versions\. You can start an update by invoking the CreateSoftwareUpdateJob API or from the Greengrass console\. Updating the Greengrass core software provides the following benefits:
+ Fix security vulnerabilities\.
+ Address software stability issues\.
+ Deploy new or improved features\.

An OTA update makes all these benefits available without having to perform the update manually or having the device which is running the core software physically present\. The OTA Update Agent also performs a rollback in case of a failed OTA update\. Performing an OTA update is optional but can help you manage your AWS IoT Greengrass core devices\. Look for announcements of new versions of the core's software on the Greengrass developer forum\.

In order to support an OTA update of Greengrass core software by using the OTA Update Agent, your Greengrass core device must:
+ Have available local storage three times the amount of the core's runtime usage requirement\. 
+ Not have trusted boot enabled in the partition containing the Greengrass core platform software\. \(The AWS IoT Greengrass core can be installed and run on a partition with trusted boot enabled, but cannot perform an OTA update\.\) 
+ Have read/write permissions on the partition containing the Greengrass core platform software\.
+ Have a connection to the AWS cloud\.
+ Have a correctly configured AWS IoT Greengrass core and appropriate certificates\.

Before launching an OTA Update of Greengrass core software, it is important to note the impact that it will have on the devices in your Greengrass group, both on the core device and on client devices connected locally to that core:
+ The core will be shut down during the update\.
+ Any Lambda functions running on the core will be shut down\. If those functions write to local resources, they might leave those resources in an incorrect state unless shut down properly\.
+ During the core's downtime, all its connections with the cloud will be lost and messages routed through the core by client devices will be lost\.
+ Credential caches will be lost\.
+ Queues which hold pending work for Lambda functions will be lost\.
+ Long\-lived Lambda functions will lose their dynamic state information and all pending work will be dropped\. 

The following state information will be preserved during an OTA Update:
+ Local shadows
+ Greengrass logs
+ OTA Agent logs

## Greengrass OTA Agent<a name="ota-agent"></a>

The Greengrass OTA Agent is the software component on the device which handles update jobs created and deployed in the cloud\. The Greengrass OTA Agent is distributed in the same software package as the Greengrass core software\. The agent is located in `./greengrass/ota/ota_agent/ggc-ota` and creates its logs in `/var/log/greengrass/ota/ggc_ota.txt`\.

You can start the Greengrass OTA Agent by executing the binary manually or by integrating it as part of an init script such as a systemd service file\. The binary should be run as root\. Once started, the Greengrass OTA Agent will begin listening for Greengrass update jobs from the cloud and execute them sequentially\. The Greengrass OTA Agent will ignore all other IoT job types\.

Do not start multiple OTA Agent instances as this may cause conflicts\.

If your Greengrass core or Greengrass OTA Agent is managed by an init system, see [Integration With Init Systems](#integration-with-init) for related configurations\.

**CreateSoftwareUpdateJob API**

The CreateSoftwareUpdateJob API creates a software update for a core or for several cores\. This API can be used to update the OTA Agent as well as the Greengrass core software\. It makes use of the AWS IoT Jobs feature which provides additional commands to manage a Greengrass core software update job\. See [Jobs](https://docs.aws.amazon.com/iot/latest/developerguide/iot-jobs.html) for more information on how to manage a Greengrass Update\. 

The following example shows how to create a Greengrass core software update job using the CLI: 

```
aws greengrass create-software-update-job \
    --update-targets-architecture x86_64 \
    --update-targets arn:aws:iot:us-east-1:123456789012:thing/myDevice \
    --update-targets-operating-system ubuntu \
    --software-to-update core \
    --s3-url-signer-role arn:aws:iam::123456789012:role/IotS3UrlPresigningRole \
    --update-agent-log-level WARN \
    --amzn-client-token myClientToken1
```

The create\-software\-update\-job command returns a JSON object containing the job id and job ARN:

```
{
    "IotJobId": "Greengrass-OTA-c3bd7f36-ee80-4d42-8321-a1da0e5b1303",
    "IotJobArn": "arn:aws:iot:us-east-1:123456789012:job/Greengrass-OTA-c3bd7f36-ee80-4d42-8321-a1da0e5b1303"
}
```

The `create-software-update-job` command has the following parameters:

`--update-targets-architecture`  
The architecture of the core device\. Must be one of `armv7l`, `x86_64` or `aarch64`\.

`--update-targets`  
A list of the targets to which the OTA update should be applied\. The list can contain the ARNS of things which are cores, and the ARNs of thing groups whose members are cores\. See [IoT thing groups](https://docs.aws.amazon.com/iot/latest/developerguide/thing-groups.html) for more information on how to place cores in an IoT thing group\. 

`--update-targets-operating-system`  
The operating system of the core device\. Must be one of `ubuntu`, `amazon_linux` or `raspbian`\.

`--software-to-update`  
Specifies whether the core's software or the OTA Agent software should be updated\. Must be one of `core` or `ota_agent`\.

`--s3-url-signer-role`  <a name="s3-url-signer-role"></a>
The IAM role which is used to presign the S3 URL which links to the Greengrass software update\. You must provide a role that has the appropriate policy attached\. Here is an example policy document with the minimum required permissions:  

```
{
    "Version": "2012-10-17",
    "Statement": [
        {
            "Sid": "AllowsIotToAccessGreengrassOTAUpdateArtifacts",
            "Effect": "Allow",
            "Action": [
                "s3:GetObject"
            ],
            "Resource": [
                "arn:aws:s3:::us-east-1-greengrass-updates/*",
                "arn:aws:s3:::us-west-2-greengrass-updates/*",
                "arn:aws:s3:::ap-northeast-1-greengrass-updates/*",
                "arn:aws:s3:::ap-southeast-2-greengrass-updates/*",
                "arn:aws:s3:::eu-central-1-greengrass-updates/*",
                "arn:aws:s3:::eu-west-1-greengrass-updates/*"
            ]
        }
    ]
}
```
Here is an example Assume Role policy document with the minimum required trusted entities:  

```
{
     "Version": "2012-10-17",
     "Statement": [
         {
             "Action": "sts:AssumeRole",
             "Principal": {
                  "Service": "iot.amazonaws.com"
             },
             "Effect": "Allow",
             "Sid": "AllowIotToAssumeRole"
        }
    ]
}
```

`--amzn-client-token`  
\[Optional\] A client token used to make idempotent requests\. Provide a unique token to prevent duplicate updates from being created due to internal retries\. 

`--update-agent-log-level`  
\[Optional\] The logging level for log statements generated by the OTA Agent\. Must be one of `NONE`, `TRACE`, `DEBUG`, `VERBOSE`, `INFO`, `WARN`, `ERROR`, or `FATAL`\. The default is `ERROR`\.

Here is an example IAM policy with the minimum permissions required to call the API: 

```
{
    "Version": "2012-10-17",
    "Statement": [
        {
            "Sid": "AllowCreateSoftwareUpdateJob",
            "Action": [
                "greengrass:CreateSoftwareUpdateJob"
            ],
            "Effect": "Allow",
            "Resource": "*"
        },
        {
            "Effect": "Allow",
            "Action": [
                "iam:PassRole"
            ],
            "Resource": "arn:aws:s3:us-east-1:123456789012:role/IotS3UrlPresigningRole"
        },
        {
            "Effect": "Allow",
            "Action": [
                "iot:CreateJob"
            ],
            "Resource": "*"
        }
    ]
}
```

**Note**  
 Because Greengrass is only supported on a subset of the architecture and operating system combinations possible with this command, `CreateSoftwareUpdateJob` will reject requests except for the following supported platforms:   
ubuntu/x86\_64 
ubuntu/aarch64 
amazon\_linux/x86\_64 
raspbian/armv7l 

## Integration with Init systems<a name="integration-with-init"></a>

During an OTA update, binaries, some of which may be running, will be updated and restarted\. This may cause conflicts if an init system is monitoring the state of either the AWS IoT Greengrass core software or the Greengrass OTA Agent during the update\. To help integrate the OTA update mechanism with your monitoring strategies, Greengrass provides the opportunity for user\-defined shell scripts to run before and after an update\. To tell the OTA agent to run these shell scripts, you must include the `managedRespawn = true` flag in the `./greengrass/config/config.json` file\. For example:

```
{

   "coreThing": {                                                     
       …
   },
   "runtime": {                                                       
        …
   },
   "managedRespawn": true                                            

}
```

When the `managedRespawn` flag is set, the scripts must exist in the directory or the OTA Agent will fail the update\. The directory tree should look as follows:

```
<greengrass_root>
|-- certs
|-- config
|   |-- config.json
|-- ggc
|-- usr/scripts
|   |-- ggc_pre_update.sh
|   |-- ggc_post_update.sh
|   |-- ota_pre_update.sh
|   |-- ota_post_update.sh
|-- ota
```

### OTA Self\-Update with Managed Respawn<a name="managed-respawn"></a>

As the OTA Agent prepares to do a self\-update, if the `managedRespawn` flag is set to `true` then the OTA Agent will look in the `./greengrass/usr/scripts` directory for the `ota_pre_update.sh` script and run it\.

After the OTA Agent completes the update, it will attempt to run the `ota_post_update.sh` script from the `./greengrass/usr/scripts` directory\.

### AWS IoT Greengrass Core Update with Managed Respawn<a name="managed-respawn-ggc"></a>

As the OTA Agent prepares to do an AWS IoT Greengrass core update, if the `managedRespawn` flag is set to `true`, then the OTA Agent will look in the `./greengrass/usr/scripts` directory for the `ggc_pre_update.sh` script and run it\.

After the OTA Agent completes the update, it will attempt to run the `ggc_post_update.sh` script from the `./greengrass/usr/scripts` directory\.

Note:
+ The user\-defined scripts in `./greengrass/usr/scripts` should be owned by root and executable by root only\.
+ If `managedRespawn` is set to `true`, the scripts must exist and return a successful return code\.
+ If `managedRespawn` is set to `false`, the scripts will not be run even if present on the device\.
+ It is imperative that a device which is the target of an update not run two OTA agents for the same AWS IoT thing\. Doing so will cause the two OTA Agents to process the same jobs which will lead to conflicts\.

## OTA Agent Self\-Update<a name="agent-self-update"></a>

To perform an OTA Agent self\-update follow these steps:

1. Ensure that the AWS IoT Greengrass core is correctly provisioned with valid `config.json` file entries and the necessary certificates\.

1. If the OTA Agent is being managed by an init system, ensure that `managedRespawn = true` in the `config.json` file and the scripts `ota_pre_update.sh` and `ota_post_update.sh` are present in the `./greengrass/usr/scripts` directory\.

1. Start the ggc\-ota agent by running `./greengrass/ota/ota_agent/ggc-ota`\.

1. Create an OTA self update job in the cloud with the CreateSoftwareUpdateJob API \(`aws greengrass create-software-update-job`\), making sure the `--software-to-update` parameter is set to `ota_agent`\.

1. The OTA Agent will perform a self update\.

## Greengrass Core Software Update<a name="ggc-update"></a>

To perform an AWS IoT Greengrass core software update follow these steps:

1. Ensure that the AWS IoT Greengrass core is correctly provisioned with valid `config.json` file entries and the necessary certificates\.

1. If the AWS IoT Greengrass core software is being managed by an init system, ensure that `managedRespawn = true` in the `config.json` file and the scripts `ggc_pre_update.sh` and `ggc_post_update.sh` are present in the `./greengrass/usr/scripts` directory\.

1. Start the ggc\-ota agent by running `./greengrass/ota/ota_agent/ggc-ota`\.

1. Create an OTA self update job in the cloud with the CreateSoftwareUpdateJob API \(`aws greengrass create-software-update-job`\), making sure the `--software-to-update` parameter is set to `core`\.

1. The OTA Agent will perform an update of AWS IoT Greengrass core software\.