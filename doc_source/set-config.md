# Setting Configuration to Run the AWS IoT Greengrass Qualification Suite<a name="set-config"></a>

Before you run tests, you must configure settings for AWS credentials and devices on your host computer\.

## Configure Your AWS Credentials<a name="cfg-aws-gg"></a>

You must configure your IAM user credentials in the `<device_tester_extract_location> /configs/config.json` file\. Use the credentials for the IDT for AWS IoT Greengrass user created in [Create and Configure an AWS Account](dev-tst-prereqs.md#config-aws-account-for-idt)\. You can specify your credentials in one of two ways:
+ Credentials file
+ Environment variables

### Configure AWS Credentials with a Credentials File<a name="config-cred-file"></a>

IDT uses the same credentials file as the AWS CLI\. For more information, see [Configuration and Credential Files](https://docs.aws.amazon.com/cli/latest/userguide/cli-config-files.html)\.

The location of the credentials file varies, depending on the operating system you are using:
+ macOS, Linux: `~/.aws/credentials`
+ Windows: `C:\Users\UserName\.aws\credentials`

Add your AWS credentials to the `credentials` file in the following format:

```
[default]
aws_access_key_id = <your_access_key_id>
aws_secret_access_key = <your_secret_access_key>
```

To configure IDT for AWS IoT Greengrass to use AWS credentials from your `credentials` file, edit your `config.json` file as follows:

```
{
	"awsRegion": "us-west-2",
	"auth": {
		"method": "file",
		"credentials": {
			"profile": "default"
		}
	}
}
```

**Note**  
If you do not use the `default` AWS profile, be sure to change the profile name in your `config.json` file\. For more information, see [Named Profiles](https://docs.aws.amazon.com/cli/latest/userguide/cli-configure-profiles.html)\.

### Configure AWS Credentials with Environment Variables<a name="config-env-vars"></a>

Environment variables are variables maintained by the operating system and used by system commands\. They are not saved if you close the SSH session\. IDT for AWS IoT Greengrass can use the `AWS_ACCESS_KEY_ID` and `AWS_SECRET_ACCESS_KEY` environment variables to store your AWS credentials\.

To set these variables on Linux, macOS, or Unix, use export:

```
export AWS_ACCESS_KEY_ID=<your_access_key_id>
export AWS_SECRET_ACCESS_KEY=<your_secret_access_key>
```

To set these variables on Windows, use set:

```
set AWS_ACCESS_KEY_ID=<your_access_key_id>
set AWS_SECRET_ACCESS_KEY=<your_secret_access_key>
```

To configure IDT to use the environment variables, edit the `auth` section in your `config.json` file\. Here is an example:

```
{
	"awsRegion": "us-west-2",
	"auth": {
		"method": "environment"
	}
}
```

## Device Configuration<a name="device-config"></a>

In addition to AWS credentials, IDT for AWS IoT Greengrass needs information about the devices that tests are run on \(for example, IP address, login information, operating system, and CPU architecture\)\.

You must provide this information using the `device.json` template located in ` <device_tester_extract_location>/configs/device.json`:

------
#### [ Physical device ]

```
[
  {
    "id": "<pool-id>",
    "sku": "<sku>",
    "features": [
      {
        "name": "os",
        "value": "linux | ubuntu | openwrt"
      },
      {
        "name": "arch",
        "value": "x86_64 | armv6l | armv7l | aarch64"
      }
    ],
    "hsm": {
      "p11Provider": "</path/to/pkcs11ProviderLibrary>",
      "slotLabel": "<slot-label>",
      "slotUserPin": "<pin>",
      "privateKeyLabel": "<key-label>",
      "openSSLEngine": "</path/to/openssl/engine>"
    },
    "kernelConfigLocation": "",
    "greengrassLocation": "",
    "devices": [
      {
        "id": "<device-id>",
        "connectivity": {
          "protocol": "ssh",
          "ip": "<ip-address>",
          "auth": {
            "method": "pki" | "password",
            "credentials": {
              "user": "<user>",
              "privKeyPath": "</path/to/private/key>",
              "password": "<your-password>"
            }
          }
        }
      }
    ]
  }
]
```

**Note**  
Specify `privKeyPath` only if `method` is set to `pki`\.  
Specify `password` only if `method` is set to `password`

------
#### [ Docker container ]

```
[
  {
    "id": "<pool-id>",
    "sku": "<sku>",
    "features": [
      {
        "name": "os",
        "value": "linux | ubuntu | openwrt"
      },
      {
        "name": "arch",
        "value": "x86_64"
      }
    ],
    "kernelConfigLocation": "",
    "greengrassLocation": "/greengrass",
    "devices": [
      {
        "id": "<device-id>",
        "connectivity": {
          "protocol": "docker",
          "containerId": "<container-name>" | "<container-id>"
        }
      }
    ]
  }
]
```

------

All fields that contain values are required as described here:

`id`  
A user\-defined alphanumeric ID that uniquely identifies a collection of devices called a *device pool*\. Devices that belong to a pool must have identical hardware\. When you run a suite of tests, devices in the pool are used to parallelize the workload\. Multiple devices are used to run different tests\.

`sku`  
An alphanumeric value that uniquely identifies the device under test\. The SKU is used to track qualified boards\.  
If you want to list your board in the AWS Partner Device Catalog, the SKU you specify here must match the SKU that you use in the listing process\.

`features`  
An array that contains the device's supported features\.  
+ Required features: `os`, `arch`\.
+ Supported OS/architecture combinations:
  + Linux, x86\_64
  + Linux, ARMv6l
  + Linux, ARMv7l
  + Linux, AArch64
  + Ubuntu, x86\_64
  + OpenWrt, ARMv7l
  + OpenWrt, AArch64
**Note**  
When you use IDT to test AWS IoT Greengrass running in a Docker container, the `os` field is your Docker operating system and `arch` is your Docker architecture\. Currently, only the x86\_64 Docker architecture is supported\.

`hsm (optional)`  
Contains configuration information for testing with an AWS IoT Greengrass Hardware Security Module \(HSM\)\. Otherwise, the `<hsm>` element should be omitted\. For more information, see [Hardware Security Integration](hardware-security.md)\.  
<a name="connectivity-protocol-ssh-only"></a>This property applies only if `connectivity.protocol` is set to `ssh`\.    
`hsm.p11Provider`  
The absolute path to the PKCS\#11 implementation's libdl\-loadable library\.  
`hsm.slotLabel`  
The slot label used to identify the hardware module\.  
`hsm.slotUserPin`  
The user PIN used to authenticate the AWS IoT Greengrass core to the module\.  
`hsm.privateKeyLabel`  
The label used to identify the key in the hardware module\.  
`hsm.openSSLEngine`  
The absolute path to the OpenSSL engine's `.so` file that enables PKCS\#11 support on OpenSSL\. Used by the AWS IoT Greengrass OTA update agent\.

`devices.id`  
A user\-defined unique identifier for the device being tested\.

`connectivity.protocol`  
The communication protocol used to communicate with this device\. Currently, the only supported values are `ssh` for physical devices and `docker` for Docker containers\.

`connectivity.ip`  
The IP address of the device being tested\.  
<a name="connectivity-protocol-ssh-only"></a>This property applies only if `connectivity.protocol` is set to `ssh`\.

`connectivity.containerId`  
The container ID or name of the Docker container being tested\.  
<a name="connectivity-protocol-docker-only"></a>This property applies only if `connectivity.protocol` is set to `docker`\.

`connectivity.auth`  
Authentication information for the connection\.  
<a name="connectivity-protocol-ssh-only"></a>This property applies only if `connectivity.protocol` is set to `ssh`\.    
`connectivity.auth.method`  
The authentication method used to access a device over the given connectivity protocol\.  
Supported values are:  
+ `pki`
+ `password`  
`connectivity.auth.credentials`  
The credentials used for authentication\.    
`connectivity.auth.credentials.password`  
The password used for signing in to the device being tested\.  
This value applies only if `connectivity.auth.method` is set to `password`\.  
`connectivity.auth.credentials.privKeyPath`  
The full path to the private key used to sign in to the device under test\.  
This value applies only if `connectivity.auth.method` is set to `pki`\.  
`connectivity.auth.credentials.user`  
The user name for signing in to the device being tested\.  
`connectivity.auth.credentials.privKeyPath`  
The full path to the private key used to sign in to the device being tested\.

`greengrassLocation`  
The location of AWS IoT Greengrass Core software on your devices\.  
For physical devices, this value is only used when you use an existing installation of AWS IoT Greengrass\. Use this attribute to tell IDT to use the version of the AWS IoT Greengrass Core software installed on your devices\.  
When running tests in a Docker container from Docker image or Dockerfile provided by AWS IoT Greengrass, set this value to `/greengrass`\.

`kernelConfigLocation`  
\(Optional\) The path to the kernel configuration file\. AWS IoT Device Tester uses this file to check if the devices have the required kernel features enabled\. If not specified, IDT uses the following paths to search for the kernel configuration file: `/proc/config.gz` and `/boot/config-<kernel-version>`\. AWS IoT Device Tester uses the first path it finds\.