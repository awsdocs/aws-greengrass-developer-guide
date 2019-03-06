# AWS IoT Device Tester for AWS IoT Greengrass User Guide<a name="device-tester-for-greengrass-ug"></a>

AWS IoT Device Tester \(IDT\) allows you to test that AWS IoT Greengrass works locally on your device and can communicate with the AWS IoT cloud\. The components of AWS IoT Device Tester include a Test Manager and Test Cases\.

Test Manager runs on the host computer \(Windows, Mac, or Linux\) that is connected to the device to be tested\. The Test Manager executes test cases and aggregates results\. It also provides a command line interface to manage test execution\. Test Cases contain test logic and set up the resources required for tests\.

![\[Image NOT FOUND\]](http://docs.aws.amazon.com/greengrass/latest/developerguide/images/devicetester_gg.png)

AWS IoT Device Tester for AWS IoT Greengrass verifies that the combination your Linux device’s CPU architecture, kernel configuration and drivers work with AWS IoT Greengrass\. For more information about the tests AWS IoT Device Tester runs, see [Test Group Descriptions](dt-test-groups.md)\.

## AWS IoT Device Tester for AWS IoT Greengrass Versions<a name="dev-test-versions"></a>

The following tabs describe the versions of AWS IoT Device Tester for AWS IoT Greengrass\. By downloading the software you agree to the [AWS IoT Device Tester License agreement](https://d232ctwt5kahio.cloudfront.net/greengrass/AWS%20IoT%20Device%20Tester%20License%20Agreement.pdf)\. Each version of AWS IoT Greengrass has a corresponding version of AWS IoT Device Tester\. Newer versions of AWS IoT Device Tester will be added to this page with each new release of AWS IoT Greengrass\. Please refer to the [ AWS IoT Device Tester for AWS IoT Greengrass User Guide](https://docs.aws.amazon.com/greengrass/latest/developerguide/device-tester-for-greengrass-ug.html) for more details about AWS IoT Device Tester for AWS IoT Greengrass\.

------
#### [ IDT for AWS IoT Greengrass v1\.7 ]

**IDT for AWS IoT Greengrass v1\.7\.1 \- Current Version**  
+ IDT for AWS IoT Greengrass \- [ Linux](https://d232ctwt5kahio.cloudfront.net/greengrass/devicetester_greengrass_linux_1.0.190219211616.zip) \(MD5 1f89310b8c679bfeb5f7273338302545\)
+ IDT for AWS IoT Greengrass \- [ Mac](https://d232ctwt5kahio.cloudfront.net/greengrass/devicetester_greengrass_mac_1.0.190219211616.zip) \(MD5 0bc67bf3da5a00e023641a821aa22952\)
+ IDT for AWS IoT Greengrass \- [ Windows](https://d232ctwt5kahio.cloudfront.net/greengrass/devicetester_greengrass_win_1.0.190219211616.zip) \(MD5 3d470a17fb10bb996b89245a00847b64\)

------
#### [ IDT for AWS IoT Greengrass v1\.6 ]

**IDT for AWS IoT Greengrass v1\.6\.1**  
+ IDT for AWS IoT Greengrass \- [ Linux](https://d232ctwt5kahio.cloudfront.net/greengrass/devicetester_greengrass_linux_1.0.190219211017.zip) \(MD5 2781e183d62a8d4a06b1321d5d40b2dc\)
+ IDT for AWS IoT Greengrass \- [ Mac](https://d232ctwt5kahio.cloudfront.net/greengrass/devicetester_greengrass_mac_1.0.190219211017.zip) \(MD5 a997f89566048c124964ba26c4d7fde0\)
+ IDT for AWS IoT Greengrass \- [ Windows](https://d232ctwt5kahio.cloudfront.net/greengrass/devicetester_greengrass_win_1.0.190219211017.zip) \(MD5 56d412810045ed2b63f7fb9547531d60\)

------

## Prerequisites<a name="dev-tst-prereqs"></a>

This section describes the prerequistes for AWS IoT Device Tester for AWS IoT Greengrass

### Download the Latest Version of AWS IoT Device Tester for AWS IoT Greengrass<a name="install-dev-tst-gg"></a>

Download the latest version of AWS IoT Device Tester from [AWS IoT Device Tester for AWS IoT Greengrass Versions](#dev-test-versions)\. The software should be extracted into a location on the file system where you have read and write permissions\. Due to a path length limitation, on Microsoft Windows, extract AWS IoT Device Tester into a root directory like C:\\ or D:\\\.

### Create and Configure an AWS Account<a name="config-aws-account"></a>

If you don't have an AWS account, follow the instructions on the [AWS webpage](https://aws.amazon.com) to create one\. Choose **Create an AWS Account** and follow the prompts\.

#### Create an IAM User in Your AWS Account<a name="create-iam-user-gg"></a>

When you create an AWS account, a root user that has access to all resources in your account is created for you\. It is a best practice to create another user for everyday tasks\. To create an IAM user, follow the instructions in [Creating an IAM User in Your AWS Account](https://docs.aws.amazon.com/IAM/latest/UserGuide/id_users_create.html)\. For more information about the root user, see [The AWS Account Root User](https://docs.aws.amazon.com/IAM/latest/UserGuide/id_root-user.html)\.

#### Create and Attach an IAM Policy to Your AWS Account<a name="create-policy-bk-gg"></a>

IAM policies grant your IAM user access to AWS resources\. 

**To create an IAM policy**

1. Browse to the [IAM console](https://console.aws.amazon.com/iam/)\.

1. In the navigation pane, choose **Policies**, and then choose **Create Policy**\.

1. Choose the **JSON** tab and copy and paste the policy template located in the [Permissions Policy Template](policy-template.md) into the editor window\.

1. Choose **Review policy**\.

1. In **Name**, enter a name for your policy\. In **Description**, enter an optional description\. Choose **Create Policy**\.

After you create an IAM policy, you must attach it to your IAM user\.

**To attach an IAM policy to your IAM user**

1. Browse to the [IAM console](https://console.aws.amazon.com/iam/)\.

1. In the navigation pane, choose **Users**\. Find and select your IAM user\.

1. Choose **Add permissions**, and then choose **Attach existing policies directly**\. Find and select your IAM policy, choose **Next: Review**, and then choose **Add Permissions**\.

### Install the AWS Command Line Interface \(CLI\)<a name="install-cli"></a>

 You will need to use the CLI to perform some operations, if you don't have the CLI installed, follow the instructions in [Install the AWS CLI](https://docs.aws.amazon.com/cli/latest/userguide/installing.html)\.

### Configure the AWS IoT Greengrass Service Role<a name="config-gg-role"></a>

To successfully deploy an AWS IoT Greengrass group, AWS IoT Greengrass requires permissions to perform actions on your behalf\. Follow these instructions to create and associate the service role\.

**To create the AWS IoT Greengrass service role**

1. Use the following AWS CLI command on your host computer to create a service role\.

   ```
   aws iam create-role --role-name GreengrassServiceRole --assume-role-policy-document '{ 
   	"Version": "2012-10-17",
   	"Statement": [{
   		"Effect": "Allow",
   		"Principal": {
   			"Service": "greengrass.amazonaws.com"
   		},
   		"Action": "sts:AssumeRole"
   	}]
   }'
   ```

   This command generates a service role ARN that you use when you associate your AWS IoT Greengrass service role to your AWS account\.

1. Use the following AWS CLI command to attach the `AWSGreengrassResourceAccessRolePolicy` to your AWS IoT Greengrass service role\.

   ```
   aws iam attach-role-policy --policy-arn arn:aws:iam::aws:policy/service-role/AWSGreengrassResourceAccessRolePolicy --role-name GreengrassServiceRole
   ```

1. Use the following AWS CLI command to associate your AWS IoT Greengrass service role with your AWS account\.

   ```
   aws greengrass associate-service-role-to-account --role-arn <your-greengrass-service-role-arn>
   ```

### Verify AWS IoT Greengrass Dependencies on the Device Under Test<a name="install-gg-dependencies"></a>

Before AWS IoT Device Tester can test your devices, make sure that you have set up your device to run AWS IoT Greengrass as described in [Getting Started with AWS IoT Greengrass](https://docs.aws.amazon.com/greengrass/latest/developerguide/gg-gs.html)\. For information about supported platforms, see [Supported Platforms](https://docs.aws.amazon.com/greengrass/latest/developerguide/what-is-gg.html#gg-platforms)\.

### Configure the AWS IoT Greengrass Software<a name="config-gg"></a>

AWS IoT Device Tester tests your device for compatibility with a specific version of AWS IoT Greengrass\. AWS IoT Device Tester provides you two options to test AWS IoT Greengrass on your devices:
+ Use a version of the AWS IoT Greengrass Core software downloaded from the AWS IoT console\. AWS IoT Device Tester installs the software for you\.
+ Use a version of the AWS IoT Greengrass Core software already installed on your device\.

**Note**  
Each version of AWS IoT Greengrass has a corresponding AWS IoT Device Tester version\. You must download the version of AWS IoT Device Tester that corresponds to the version of AWS IoT Greengrass you are using\.

#### Download the AWS IoT Greengrass Core Software and Configure AWS IoT Device Tester to Use It<a name="download-gg"></a>

You can download the AWS IoT Greengrass Core software from the **Software** page in the AWS IoT Core console or from the [AWS IoT Greengrass Core Software](what-is-gg.md#gg-core-download-tab) downloads\. The following procedure shows you how to download the AWS IoT Greengrass Core software from the AWS IoT console\.

**To use a version of the AWS IoT Greengrass Core software downloaded from the AWS IoT console**

1. Browse to the [AWS IoT console](https://console.aws.amazon.com/iot/)\.

1. In the navigation pane, choose **Software**\.

1. In **AWS IoT Greengrass Core Software**, choose **Configure download**\.

1. From the available configurations, find the correct architecture and Linux distribution, and then choose **Download**\.

1. Copy the tar\.gz file to the `<device-tester-extract-location>/products/greengrass/ggc`\.

**Note**  
Do not change the name of the AWS IoT Greengrass tar\.gz file\.

#### Use an Existing Installation of AWS IoT Greengrass with AWS IoT Device Tester<a name="existing-gg"></a>

Configure AWS IoT Device Tester to test the AWS IoT Greengrass Core software installed on your device by adding the `greengrassLocation` attribute to the `device.json` file located in the `<device_tester_extract_location>/configs` folder\. For example:

```
"greengrassLocation" : "<path-to-greengrass-on-device>"
```

For more information about the `device.json` file, see [Device Configuration](#device-config)\.

On Linux devices, the default location of the AWS IoT Greengrass Core software is `/greengrass`\.

**Note**  
Your device should have a clean install of the AWS IoT Greengrass Core software where the AWS IoT Greengrass Core software was never started\.

### Configure Your Host Computer to Access Your Device Under Test<a name="configure-host"></a>

The AWS IoT Device Tester runs on your host computer and must be able to use SSH to connect to your device\. You must create an SSH key and authorize your key to sign in to your device under test\.

The following instructions for creating an SSH key are written with the assumption that you are using OpenSSL\. If you are using another SSL implementation, refer to the documentation\.

#### Create an SSH Key<a name="create-ssh-key"></a>

AWS IoT Device Tester uses an SSH key to authenticate with your device under test\. You can use the Open SSH ssh\-keygen command to create an SSH key pair\. If you already have an SSH key pair on your host computer, it is a best practice to create a new one to allow AWS IoT Device Tester to sign in to your device under test\.

**Note**  
Windows does not come with an installed SSH client\. For information about installing an SSH client on Windows, see [Download SSH Client Software](https://www.ssh.com/ssh/#sec-Download-client-software)\.

The ssh\-keygen command prompts you for a name and path to store the key pair\. By default, the key pair files are named id\_rsa \(private key\) and id\_rsa\.pub \(public key\)\. On macOS and and Linux, the default location of these files is `~/.ssh/`\. On Windows, the default location is `C:\Users\<user-name>\.ssh`\.

When you are prompted for a key phrase, type a key phrase to protect your SSH key\. For more information about creating your own SSH key and key phrases, see [Generate a New SSH Key](https://www.ssh.com/ssh/keygen/)\.

#### Adding Authorized SSH Keys to Your Device Under Test<a name="auth-keys"></a>

AWS IoT Device Tester must use your SSH private key to sign in to your device under test\. To authorize your SSH private key to sign in to your device under test, use the ssh\-copy\-id command from your host computer\. This command adds your public key into the `~/.ssh/authorized_keys` file on your device under test\. For example:

$ ssh\-copy\-id*<remote\-ssh\-user>*@*<remote\-device\-ip>*

Where *remote\-ssh\-user* is the user name used to sign in to your device under test and *remote\-device\-ip* is the IP address of the device under test to run tests against\. For example:

ssh\-copy\-id pi@192\.168\.1\.5

When prompted, enter the password for the user name you specified in the ssh\-copy\-id command\.

ssh\-copy\-id assumes the public key is named id\_rsa\.pub and is stored the default location \(on macOS and Linux, `~/.ssh/` and on Windows, `C:\Users\<user-name>\.ssh`\)\. If you gave the public key a different name or stored it in a different location, you must specify the fully qualified path to your SSH public key using the \-i option to ssh\-copy\-id \(for example, ssh\-copy\-id \-i \~/my/path/myKey\.pub\)\. For more information about creating SSH keys and copying public keys, see [SSH\-COPY\-ID](https://www.ssh.com/ssh/copy-id)\.

### Root Access<a name="root-access"></a>

AWS IoT Device Tester performs operations on various directories and files in a device under test \(DUT\)\. Some of these operations require root access\. To automate these operations, AWS IoT Device Tester must be able to run commands with sudo without being prompted for a password\. 

Follow these steps on the device under test to allow sudo access without being prompted for a password\. 

**Note**  
*user* and *user name* refer to the SSH user used by AWS IoT Device Tester to access the DUT\.

**To add the user to the sudo group**

1. Run `sudo usermod -aG sudo <username>`

1. Sign out and then sign back in for changes to take effect\.

## Setting Configuration to Run the AWS IoT Greengrass Qualification Suite<a name="set-config"></a>

Before you run tests, you must configure settings for logging, AWS credentials, and devices on your host computer\.

### Configure Your AWS Credentials<a name="cfg-aws-gg"></a>

You must configure your IAM user credentials in the `<device_tester_extract_location>/configs/config.json`\. You can specify your credentials in one of two ways:
+ Environment variables
+ Credentials file

#### Configuring AWS Credentials with Environment Variables<a name="config-env-vars"></a>

Environment variables are variables maintained by the operating system and used by system commands\. AWS IoT Device Tester can use the `AWS_ACCESS_KEY_ID` and `AWS_SECRET_ACCESS_KEY` environment variables to store your AWS credentials\. The way you set environment variables depends on the operating system you are running\.

**To set environment variables on Windows**

1. From the Windows 10 desktop, open the **Power User Task Menu**\. To open the menu, move your mouse cursor to the bottom\-left corner of the screen \(the **Start** menu icon\) and right\-click\.

1. From the **Power User Task Menu**, choose **System**, and then choose **Advanced System Settings**\.
**Note**  
In Windows 10, you might need to scroll to **Related settings** and choose **System info**\. In **System**, choose **Advanced system settings**\.

   In **System Properties**, choose the **Advanced** tab, and then choose the **Environment Variables** button\.

1. Under **User variables for <user\-name>**, choose **New** to create an environment variable\. Enter a name and a value for each environment variable\. 

**To set environment variables on macOS and Linux**
+ Open `~/.bash_profile` in any text editor and add the following lines: 

  ```
  export AWS_ACCESS_KEY_ID="<your-aws-access-key-id>"
  export AWS_SECRET_ACCESS_KEY="<your-aws-secret-key>"
  ```

  Replace the items in angle brackets \(< & >\) with your AWS access key ID and AWS secret key\.

After you set your environment variables, close your command line window or terminal and reopen it so the new values take effect\.

To configure your AWS credentials using environment variables, set the `AWS_ACCESS_KEY_ID` and `AWS_SECRET_ACCESS_KEY`\. In the `config.json` file, set the `method` attribute to `environment`:

```
{
	"awsRegion": "us-west-2",
	"auth": {
		"method": "environment"
	}
}
```

### Configuring AWS Credentials with a Credentials File<a name="config-cred-file"></a>

Create a credentials file that contains your credentials\. AWS IoT Device Tester uses the same credentials file as the AWS CLI\. For more information, see [Configuration and Credential Files](https://docs.aws.amazon.com/cli/latest/userguide/cli-config-files.html)\. You must also specify a named profile\. For more information, see [Named Profiles](https://docs.aws.amazon.com/cli/latest/userguide/cli-multiple-profiles.html)\. 

The following is an example JSON snippet that shows how to specify AWS credentials using a credentials file in the `config.json` file:

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

### Device Configuration<a name="device-config"></a>

In addition to AWS credentials, AWS IoT Device Tester needs information about the devices available to run tests on \(for example, IP address, login information, operating system, and CPU architecture\)\. 

You can provide this information using the `device.json` template located in `<device_tester_extract_location>/configs/device.json`:

```
[
  {
    "id": "<pool-id>",
    "sku": "<sku>",
    "features": [
      {
        "name": "os",
        "value": "linux | ubuntu"
      },
      {
        "name": "arch",
        "value": "x86_64 | armv7l | aarch64"
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
            "method": "pki",
            "credentials": {
              "user": "<user>",
              "privKeyPath": "</path/to/private/key>"
            }
          }
        }
      }
    ]
  }
]
```

All fields that contain values are required as described here:

`id`  
A user\-defined alphanumeric ID that uniquely identifies a pool of devices\. Devices that belong to a pool must be of the same type\. When you run a suite of tests, devices in the pool are used to parallelize the workload\.

`sku`  
An alphanumeric value that uniquely identifies the device under test\. The SKU is used to track qualified boards\.  
If you want to list your board in the AWS IoT Device Catalog, the SKU you specify here must match the SKU that you use in the listing process\.

`features`  
An array that contains the device's supported features\.  
+ Required features: `os`, `arch`\.
+ Supported OS/architecture combinations:
  + linux, x86\_64
  + linux, armv7l
  + linux, aarch64
  + ubuntu, x86\_64

`hsm`  
An array that contains configuration information for testing with AWS IoT Greengrass hardware security integration\. For more information, see [Hardware Security Integration](hardware-security.md)     
`hsm.p11Provider`  
The absolute path to the PKCS\#11 implementation's libdl\-loadable library\.  
`hsm.slotLabel`  
The slot label used to identify the hardware module\.  
`hsm.slotUserPin`  
The user pin used to authenticate the AWS IoT Greengrass core to the module\.  
`hsm.privateKeyLabel`  
The label used to identify the key within the hardware module\.  
`hsm.openSSLEngine`  
The absolute path to the OpenSSL engine's \.so file that enables PKCS\#11 support on OpenSSL\. This is used by the AWS IoT Greengrass OTA update agent\.

`devices.id`  
A user\-defined unique identifier for the device being tested\.

`connectivity.protocol`  
The communication protocol used to communicate with this device\. Supported values are:  
+ `ssh`
+ `serial` \(Amazon FreeRTOS only\)

`connectivity.ip`  
The IP address of the device being tested\.

`connectivity.auth.method`  
The authorization method used to access a device over the given connectivity protocol\. Supported values are:  
+ `pki`

`connectivity.auth.credentials.user`  
The user name for signing in to the device being tested\.

`connectivity.auth.credentials.privKeyPath`  
The full path to the private key used to sign in to the device being tested\.

`greengrassLocation`  
\(Optional\) The location of AWS IoT Greengrass Core software on your devices\. Use this attribute to tell &IoT; Device Tester to use the version of the AWS IoT Greengrass Core software installed on your devices\.

`kernelConfigLocation`  
\(Optional\) The path to the kernel configuration file\. AWS IoT Device Tester uses this file to check whether the devices have the required kernel features enabled\. If not specified, AWS IoT Device Tester uses one of the following paths: `/proc/config.gz` or `/boot/config-<kernel-version>`\.

### Running the Test Suite<a name="run-tests"></a>

After you set the required configuration, you can start the tests\.

The following example command line shows you how to run the qualification tests for a device pool \(set of identical devices\)\. These commands can be found in the `<devicetester-extract-location>/bin` directory\.

Use the following command to run all test groups in a specified suite:

devicetester\_*\[linux \| mac \| win\_x86\-64\]* run\-suite \-\-suite\-id GGQ\_1 \-\-pool\-id *<pool\-id>*

Use the following command to run a specific test group:

devicetester\_*\[linux \| mac \| win\_x86\-64\]* run\-suite \-\-suite\-id GGQ\_1 \-\-group\-id *<group\-id>* \-\-pool\-id *<pool\-id>*

`suite-id` and `pool-id` are optional if you are running a single test suite on a single device pool \(that is, you have only one device pool defined in your `device.json` file\)\.

#### AWS IoT Device Tester Commands<a name="bk-cli"></a>

The AWS IoT Device Tester command supports the following operations:

help  
Lists information about the specified command\.

list\-groups  
Lists the groups in given suite\.

list\-suites  
Lists the available suites\.

run\-suite  
Runs a suite of tests on a particular pool of devices\.

### Results and Logs<a name="results-logs"></a>

This section describes how to view and interpret AWS IoT Device Tester result reports and logs\.

#### Viewing Results<a name="view-results"></a>

After AWS IoT Device Tester executes the qualification test suite, it generates two reports\. These reports can be found in `<devicetester-extract-location>/results/<execution-id>/`\. Both reports capture the results from the qualification test suite execution\.

The `awsiotdevicetester_report.xml` is the qualification test report you submit to AWS for listing your device in the AWS IoT Device Catalog\. The report contains the following elements:
+ The AWS IoT Device Tester version\.
+ The AWS IoT Greengrass version that was tested\.
+ The SKU and the device name specified in the `device.json` file\.
+ The features of the device specified in the `device.json` file\.
+ The aggregate summary of test case results\.
+ A breakdown of test case results by libraries that were tested based on the device features \(for example, lra, shadow, mqtt, and so on\)\.

The `GGQ_Result.xml` report is in standard junit\.xml format that you can integrate into your existing CI/CD platforms like Jenkins, Bamboo, and so on\. The report contains the following elements:
+ Aggregate summary of test case results\.
+ Breakdown of test case results by AWS IoT Greengrass functionality that was tested \(for example, lra, shadow, mqtt, and so on\)\.

##### Interpreting AWS IoT Device Tester Results<a name="interpreting-results-gg"></a>

The report section in `awsiotdevicetester_report.xml` or `GGQ_Report.xml` lists the tests that were run and the results\.

The first XML tag `<testsuites>` contains the overall summary of the test execution\. For example:

```
<testsuites name="GGQ results" time="2299" tests="28" failures="0" errors="0" disabled="0">
```Attributes used in the `<testsuites>` tag

`name`  
The name of the test suite\.

`time`  
The time \(in seconds\) it took to run the qualification suite\.

`tests`  
The number of test cases executed\.

`failures`  
The number of test cases that were run but did not pass\.

`errors`  
The number of test cases that AWS IoT Device tester couldn't execute\.

`disabled`  
This attribute is not used and can be ignored\.

The `GGQ_Report.xml` file contains a `<awsproduct` tag that contains information about the product being tested and the features of the product that were validated after running a suite of tests\.Attributes used in the `<awsproduct>` tag

`name`  
The name of the product tested

`version`  
The version of the product tested\.

`features`  
The features validated\. Features marked as *required* are required to submit your board for qualification\. The following snippet shows how this information is written in the GGQ\_Report\.xml file:  

```
<feature name="aws-iot-greengrass-no-container" value="supported" type="required"><
								/feature>
```

If there are no test case failures or errors, your device meets the technical requirements to run AWS IoT Greengrass and can interoperate with AWS IoT services\. If you choose to list your device in the Partner Device Catalog, you can use this report as qualification evidence\.

In the case of test case failures or errors, you can identify the test case that failed by reviewing the `<testsuites>` XML tags\. The `<testsuite>` XML tags inside the `<testsuites>` tag shows the test case result summary for a test group\. For example:

```
<testsuite name="combination" package="" tests="1" failures="0" time="161" disabled="0" errors="0" skipped="0">
```

The format is similar to the `<testsuites>` tag, but with an additional attribute called `skipped` that is not used and can be ignored\. Inside each `<testsuite>` XML tag, there are `<testcase>` tags for each executed test case for a test group\. For example:

```
<testcase classname="Security Combination (IPD + DCM) Test Context" name="Security Combination IP Change Tests sec4_test_1: Should rotate server cert when IPD disabled and following changes are made:Add CIS conn info and Add another CIS conn info" attempts="1"></testcase>>
```Attributes used in the `<testcase>` tag

`name`  
The name of the test case\.

`attempts`  
The number of times AWS IoT Device Tester executed the test case\.

When a test case fails or an error occurs, `<failure>` or `<error>` tags are added to the `<testcase>` tag with additional information for troubleshooting\. For example:

```
<testcase classname="mcu.Full_MQTT" name="AFQP_MQTT_Connect_HappyCase" attempts="1">
	<failure type="Failure">Reason for the test case failure</failure>
	<error>Reason for the test case execution error</error>
</testcase>
```

##### Viewing Logs<a name="view-logs-gg"></a>

AWS IoT Device Tester generates logs from test execution in `<devicetester-extract-location>/results/<execution-id>/logs`\. Two sets of logs are generated:

`test_manager.log`  
Contains logs generated from the Test Manager component of AWS IoT Device Tester\. For example, logs related configuration, test sequencing and report generation are here\.

`<test_case_id>.log (for example, ota.log)`  
The logs of the test group, including logs from the device under test\. When a test case fails, a tar\.gz file that contains the logs of the device under test for the test case is created \(for example, `ota_prod_test_1_ggc_logs.tar.gz`\)\.

#### Viewing Logs<a name="view-logs"></a>

AWS IoT Device Tester generates logs and writes them to `<devicetester-extract-location>/results/<execution-id>/logs`\. Two sets of logs are generated:

`test_manager.log`  
Contains the logs generated from the AWS IoT Device Tester Test Manager\. These logs contain log\-related configuration, test sequencing, and test reports\.

`<test_case_id>`  
Contains the test group logs, including logs from the device under test\.

## Testing Infrastructure<a name="testing-infrastructure"></a>

In addition to the devices being tested, other resources \(for example, AWS IoT things, AWS IoT Greengrass groups, Lambda functions, and so on\) must be created to facilitate the qualification process\.

To create these resources, AWS IoT Device Tester uses the AWS credentials configured in the `config.json` to make API calls on your behalf\. These resources are provisioned at various times during a test\.

## Test Organization<a name="test-org"></a>

AWS IoT Device Tester organizes tests using the concepts of *test suites*, *test groups*, and individual* tests*\.

A test suite consists of groups of tests that are related to AWS IoT Greengrass or Amazon FreeRTOS\. You should run a whole suite of tests to ensure that a product works as expected on devices\.

A test group consists of individual tests that are related to the particular feature being tested\.

A test consists of individual executables and configurations used to run a test to ensure the functionality of a portion of a feature\.

## Error Codes<a name="bk-error-codes"></a>

The following table lists the error codes generated by AWS IoT Device Tester for AWS IoT Greengrass\.


| Error Code | Error Code Name | Possible Root Cause | Troubleshooting | 
| --- | --- | --- | --- | 
|  101  |  InternalError  |  An internal error occured  |   Please contact [AWS Developer Support](https://aws.amazon.com/premiumsupport/plans/developers/)\.  | 
|  102  |  TimeoutError  |  TimeoutError means that test case is not able to finish in a limited time range\. This can happen in several situations: [\[See the AWS documentation website for more details\]](http://docs.aws.amazon.com/greengrass/latest/developerguide/device-tester-for-greengrass-ug.html)  |  [\[See the AWS documentation website for more details\]](http://docs.aws.amazon.com/greengrass/latest/developerguide/device-tester-for-greengrass-ug.html)  | 
|  103  |  PlatformNotSupportError  |  Incorrect OS/architecture combination specified in device\.json\.  |  Change your configuration to one of the supported combinations: [\[See the AWS documentation website for more details\]](http://docs.aws.amazon.com/greengrass/latest/developerguide/device-tester-for-greengrass-ug.html) For more information about device configurate, please refer to [Device Configuration](#device-config)\.  | 
|  104  |  VersionNotSupportError  |  The AWS IoT Greengrass core software version is not supported for the version of AWS IoT device tester you are using\.  |  Find the supported version of the AWS IoT Greengrass core software by using the device\_tester\_bin version command\. For example, if you are using a Mac you would use the following command: \./devicetester\_mac\_x86\_64 version\. To find the version of AWS IoT Greengrass core software you are using: [\[See the AWS documentation website for more details\]](http://docs.aws.amazon.com/greengrass/latest/developerguide/device-tester-for-greengrass-ug.html) You can test a different version of the AWS IoT Greengrass core software\. For more information, see [Getting Started with AWS IoT Greengrass](gg-gs.md)\.  | 
|  105  |  LanguageNotSupportError  |  AWS IoT Device Tester only supports Python for AWS IoT Greengrass libraries and SDKs\.  |  Please make sure: [\[See the AWS documentation website for more details\]](http://docs.aws.amazon.com/greengrass/latest/developerguide/device-tester-for-greengrass-ug.html)  | 
|  106  |  ValidationError  |  Some fields in device\.json or config\.json file are invalid\.  |  Please check the error message at the right side of error code in report\. If the message is: [\[See the AWS documentation website for more details\]](http://docs.aws.amazon.com/greengrass/latest/developerguide/device-tester-for-greengrass-ug.html)  | 
|  107  |  SSHConnectionFailed  |  The test machine is not able to connect to the configured device\.  |  Verify the following fields from your device\.json file are correct: [\[See the AWS documentation website for more details\]](http://docs.aws.amazon.com/greengrass/latest/developerguide/device-tester-for-greengrass-ug.html) For more information about device configuration, see [Device Configuration](#device-config)\.  | 
|  108  |  RunCommandError  |  A test case failed to execute command on the device under test\.  |  Verify that root access is allowed for the configured user in device\.json\. A password is required by some devices when executing commands with root access\. Make sure root access is allowed without a password\. consult your device's documentation for more information\. Try running the failing command manually on your device and see if an error occurs\.  | 
|  109  |  PermissionDeniedError  |  No root access\.  |  Set root access for configured user on your device\.  | 
|  110  |  CreateFileError  |  Unable to create a file\.  |  Check your device's disk space and directory permissions\.  | 
|  111  |  CreateDirError  |  Unable to create a directory\.  |  Check your device's disk space and directory permissions\.  | 
|  112  |  InvalidPathError  |  The path to the AWS IoT Greengrass core software is incorrect  |  Verify that the path in the error message is valid\. Do not to edit any files under the `/test` directory\.  | 
|  113  |  InvalidFileError  |  A file is invalid\.  |  Verify the file mentioned in error message is valid\.  | 
|  114  |  ReadFileError  |  The specified file cannot be read\.  |  Verify the following: [\[See the AWS documentation website for more details\]](http://docs.aws.amazon.com/greengrass/latest/developerguide/device-tester-for-greengrass-ug.html) If you are testing on a Mac, increase the open files limit\. The default limit is 256 which is enough for testing\.  | 
|  115  |  FileNotFoundError  |  A needed file is not found\.  |  Verify the following: [\[See the AWS documentation website for more details\]](http://docs.aws.amazon.com/greengrass/latest/developerguide/device-tester-for-greengrass-ug.html)  | 
|  116  |  OpenFileFailed  |  Unable to open the specified file\.  |  Verify the following: [\[See the AWS documentation website for more details\]](http://docs.aws.amazon.com/greengrass/latest/developerguide/device-tester-for-greengrass-ug.html) If you are testing on a Mac, increase the open files limit\. The default limit is 256 which is enough for testing\.  | 
|  117  |  WriteFileFailed  |  Failed to write file \(can be both device under test or test machine\)\.  |  Try to manually create a file in the directory specified in the error message\.  | 
|  118  |  FileCleanUpError  |  A test case failed to remove the specified file, directory or to umount the specified file on the remote device\.  |  If the binary file is still running, the file may be locked\. Kill the process and delete the specified file\.  | 
|  119  |  InvalidInputError  |  Invalid configuration\.  |  Verify your suite\.json file is valid\.  | 
|  120  |  InvalidCredentialError  |  Invalid AWS credentials\.  |  Verify your AWS credentials\. This error can also be caused by network problems\. Check your network connection and rerun the test group\.  | 
|  121  |  AWSSessionError  |  Failed to create an AWS session\.  |  The error can occur because of invalid AWS credentials or an unstable internet connection\. Try to call an AWS API using AWS ClI to see it works\.  | 
|  123  |  AWSApiCallError  |  An AWS API error occured\.  |  This might happen due to network issue\. Please check your network before retry the test group\.  | 
|  124  |  IpNotExistError  |   IP address is not included in connectivity info\.  |  Check your internet connection\. Ensure you are not using "ctrl\+c" to interrupt the test\. You can go to the AWS Greengrass console to check the connectivity information for the AWS IoT Greengrass core thing that is being used by the test\. If there are 10 endpoints included in connectivity info, you can remove some or all of them and rerun the test\. For more about connectivity info see, [Connectivity Information](https://docs.aws.amazon.com/cli/latest/reference/greengrass/get-connectivity-info.html)\.  | 
|  125  |  OTAJobNotCompleteError  |  An OTA job did not complete\.  |  Check your internet connection and retry the OTA test group\.  | 
|  126  |  CreateGreengrassServiceRoleError  |  One of the following occured: [\[See the AWS documentation website for more details\]](http://docs.aws.amazon.com/greengrass/latest/developerguide/device-tester-for-greengrass-ug.html)  |  Configure the AWS IoT Greengrass service role, for more information, see [Configure the AWS IoT Greengrass Service Role](#config-gg-role)\.  | 

## Troubleshooting<a name="bk-gg-troublshooting"></a>

We recommend the following workflow for troubleshooting testing on an AWS IoT Greengrass device:

1. Read the console output\.

1. Look in the GGQ\_Result\.xml file\.

1. Investigate one of the following problem areas:
   + Device configuration
   + Device interface

### Troubleshooting Device Configuration<a name="troubleshoot-device-config"></a>

When you use AWS IoT Device Tester, you must get the correct configuration files in place before you execute the binary\. If you are getting parsing and configuration errors, your first step should be to locate and use a configuration template appropriate for your environment\.

If you are still having issues, see the following debugging process\.

#### Where Do I Look?<a name="where-to-look"></a>

Start by looking in the `GGQ_Report.xml` file in the `/results/<uuid>` directory\. This file contains all of the test cases that were run and error snippets for each failure\. To get all of the execution logs, look under `/results/<uuid>/<test-case-id>.log` for each test group\.

#### Parsing Errors<a name="parse-error"></a>

Occasionally, a typo in a JSON configuration can lead to parsing errors\. Most of the time, the issue is a result of omitting a bracket, comma, or quote from your JSON file\. AWS IoT Device Tester performs JSON validation and prints debugging information\. It prints the line where the error occurred, the line number, and the column number of the syntax error\. This information should be enough to help you fix the error, but if you are still having issues locating the error, you can perform validation manually in your IDE, a text editor such as Atom or Sublime, or through an online tool like JSONLint\.

#### Required Parameter Missing Error<a name="param-missing"></a>

Because new features are being added to AWS IoT Device Tester, changes to the configuration files might be introduced\. Using an old configuration file might break your configuration\. If this happens, the `<test_case_id>.log` file under `/results/<uuid>/logs` explicitly lists all missing parameters\. AWS IoT Device Tester also validates your JSON configuration file schemas to ensure that the latest supported version has been used\.

#### Could Not Start Test Error<a name="could-not-start-test"></a>

You might encounter errors that point to failures during test start\. Because there are several possible causes for this, make sure to check the following areas for correctness:
+ Make sure that the pool name you've included in your execution command actually exists\. This is referenced directly from your `device.json` file\.
+ Make sure that the devices in your pool have correct configuration parameters\.

#### Permission Denied Errors<a name="pwd-sudo"></a>

AWS IoT Device Tester performs operations on various directories and files in a device under test \(DUT\)\. Some of these operations require root access\. To automate these operations, AWS IoT Device Tester must be able to run commands with sudo without typing a password\. 

Follow these steps to allow sudo access without typing a password\.

**Note**  
`user` and `username` refer to the SSH user used by AWS IoT Device Tester to access the device under test\.

1. Use sudo usermod \-aG sudo *<ssh\-username>* to add your SSH user to the sudo group

1. Sign out and then sign in for changes to take effect\.

1. Open `/etc/sudoers` file and then add the following line to the end of the file: `<ssh-username> ALL=(ALL) NOPASSWD: ALL"`

#### SSH Connection Errors<a name="connect-errors"></a>

When AWS IoT Device Tester cannot connect to a device under test, connection failures are logged in `/results/<uuid>/logs/<test-case-id>.log`\. SSH failure messages appear at the top of this log file because connecting to a device under test is one of the first operations AWS IoT Device Tester performs\.

Most Windows setups use the PuTTy terminal application to connect to Linux hosts\. This application requires that standard PEM private key files are converted into a proprietary Windows format called PPK\. When AWS IoT Device Tester is configured in your `device.json` file, it is important to use PEM files only\. If a PPK file is used, AWS IoT Device Tester cannot create an SSH connection with the AWS IoT Greengrass device and cannot run tests\.

#### Command Not Found Errors While Testing OTA<a name="cmd-not-found"></a>

You need an older version of the OpenSSL library \(libssl1\.0\.0\) to run OTA tests on AWS IoT Greengrass devices\. Most current Linux distributions use libssl version 1\.0\.2 or later \(v1\.1\.0\)\. These versions are not compatible with OTA\. 

For example, on a Raspberry Pi, run the following commands to install the required version of libssl:

1. wget http://ftp\.us\.debian\.org/debian/pool/main/o/openssl/libssl1\.0\.0\_1\.0\.2l\-1\~bpo8\+1\_armhf\.deb

1. sudo dpkg \-i libssl1\.0\.0\_1\.0\.2l\-1\~bpo8\+1\_armhf\.deb

#### Logging<a name="bk-logging"></a>

AWS IoT Device Tester logs are placed in a single location\. From the root AWS IoT Device Tester directory, the available files are:
+ `./results/<uuid>`
+ `GGQ_Report.xml`
+ `awsiotdevicetester_report.xml`
+ `logs/<test_case_id>.log`

The `<test_case_id>.log` and `GGQ_Report.xml` are the most important logs to examine\. The `results.xml` file contains information about which tests failed with a specific error message\. You can then use the `<test_case_id>.log` to investigate the problem\.

##### Console Errors<a name="err-console"></a>

When AWS IoT Device Tester is run, failures are reported to console with brief messages\. For information about the error, look in `<test_case_id>.log`\.

##### Log Errors<a name="err-log"></a>

The `<test_case_id>.log` file is located in the `/results/<uuid>/logs` directory\. Each test execution has a unique test ID that is used to create the `<uuid>` directory\. Individual test group logs are under the `<uuid>` directory\. Use the console to look up the test group that failed and then open the log file for that test case in the `/results/<uuid>/logs` directory\. The information in this file includes the full build and flash command output, as well as test execution output and more verbose AWS IoT Device Tester console output\.