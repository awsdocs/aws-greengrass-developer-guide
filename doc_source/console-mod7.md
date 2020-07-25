# Module 7: Simulating hardware security integration<a name="console-mod7"></a>

This feature is available for AWS IoT Greengrass Core v1\.7 and later\.

This advanced module shows you how to configure a simulated hardware security module \(HSM\) for use with a Greengrass core\. The configuration uses SoftHSM, which is a pure software implementation that uses the [PKCS\#11](#console-mod7-see-also) application programming interface \(API\)\. The purpose of this module is to allow you to set up an environment where you can learn and do initial testing against a software\-only implementation of the PKCS\#11 API\. It is provided only for learning and initial testing, not for production use of any kind\.

You can use this configuration to experiment with using a PKCS\#11\-compatible service to store your private keys\. For more information about the software\-only implementation, see [SoftHSM](https://www.opendnssec.org/softhsm/)\. For more information about integrating hardware security on an AWS IoT Greengrass core, including general requirements, see [Hardware security integration](hardware-security.md)\.

**Important**  
This module is intended for experimentation purposes only\. We strongly discourage the use of SoftHSM in a production environment because it might provide a false sense of additional security\. The resulting configuration doesn't provide any actual security benefits\. The keys stored in SoftHSM are not stored more securely than any other means of secrets storage in the Greengrass environment\.  
The purpose of this module is to allow you to learn about the PKCS\#11 specification and do initial testing of your software if you plan to use a real hardware\-based HSM in the future\.  
You must test your future hardware implementation separately and completely before any production usage because there might be differences between the PKCS\#11 implementation provided in SoftHSM and a hardware\-based implementation\.

If you need assistance with the onboarding of a [supported hardware security module](hardware-security.md#hardware-security-reqs), contact your AWS Enterprise Support representative\.

Before you begin, run the [Greengrass Device Setup](quick-start.md) script, or make sure that you completed [Module 1](module1.md) and [Module 2](module2.md) of the Getting Started tutorial\. In this module, we assume that your core is already provisioned and communicating with AWS\. This module should take about 30 minutes to complete\.

## Install the SoftHSM software<a name="softhsm-install"></a>

In this step, you install SoftHSM and the pkcs11 tools, which are used to manage your SoftHSM instance\.
+ In a terminal on your AWS IoT Greengrass core device, run the following command:

  ```
  sudo apt-get install softhsm2 libsofthsm2-dev pkcs11-dump
  ```

  For more information about these packages, see [Install softhsm2](https://www.howtoinstall.co/en/ubuntu/xenial/softhsm2), [Install libsofthsm2\-dev](https://www.howtoinstall.co/en/ubuntu/xenial/libsofthsm2-dev), and [Install pkcs11\-dump](https://www.howtoinstall.co/en/ubuntu/xenial/pkcs11-dump)\.
**Note**  
If you encounter issues when using this command on your system, see [SoftHSM version 2](https://github.com/opendnssec/SoftHSMv2) on GitHub\. This site provides more installation information, including how to build from source\.

## Configure SoftHSM<a name="softhsm-config"></a>

In this step, you [configure SoftHSM](https://github.com/opendnssec/SoftHSMv2#configure-1)\.

1. Switch to the root user\.

   ```
   sudo su
   ```

1. Use the manual page to find the system\-wide `softhsm2.conf` location\. A common location is `/etc/softhsm/softhsm2.conf`, but the location might be different on some systems\.

   ```
   man softhsm2.conf
   ```

1. Create the directory for the softhsm2 configuration file in the system\-wide location\. In this example, we assume the location is `/etc/softhsm/softhsm2.conf`\.

   ```
   mkdir -p /etc/softhsm
   ```

1. Create the token directory in the `/greengrass` directory\.
**Note**  
If this step is skipped, softhsm2\-util reports `ERROR: Could not initialize the library`\.

   ```
   mkdir -p /greengrass/softhsm2/tokens
   ```

1. Configure the token directory\.

   ```
   echo "directories.tokendir = /greengrass/softhsm2/tokens" > /etc/softhsm/softhsm2.conf
   ```

1. Configure a file\-based backend\.

   ```
   echo "objectstore.backend = file" >> /etc/softhsm/softhsm2.conf
   ```

**Note**  
These configuration settings are intended for experimentation purposes only\. To see all configuration options, read the manual page for the configuration file\.  

```
man softhsm2.conf
```

## Import the private key into SoftHSM<a name="softhsm-import-key"></a>

In this step, you initialize the SoftHSM token, convert the private key format, and then import the private key\.

1. Initialize the SoftHSM token\.

   ```
   softhsm2-util --init-token --slot 0 --label greengrass --so-pin 12345 --pin 1234
   ```
**Note**  
If prompted, enter an SO pin of `12345` and a user pin of `1234`\. AWS IoT Greengrass doesn't use the SO \(supervisor\) pin, so you can use any value\.  
If you receive the error `CKR_SLOT_ID_INVALID: Slot 0 does not exist`, try the following command instead:  

   ```
   softhsm2-util --init-token --free --label greengrass --so-pin 12345 --pin 1234
   ```

1. Convert the private key to a format that can be used by the SoftHSM import tool\. For this tutorial, you convert the private key that you obtained from the **Default Group creation** option in [Module 2](module2.md) of the Getting Started tutorial\.

   ```
   openssl pkcs8 -topk8 -inform PEM -outform PEM -nocrypt -in hash.private.key -out hash.private.pem
   ```

1. Import the private key into SoftHSM\. Run only one of the following commands, depending on your version of softhsm2\-util\.  
**Raspbian softhsm2\-util v2\.2\.0** syntax  

   ```
   softhsm2-util --import hash.private.pem --token greengrass --label iotkey --id 0000 --pin 12340
   ```  
**Ubuntu softhsm2\-util v2\.0\.0** syntax  

   ```
   softhsm2-util --import hash.private.pem --slot 0 --label iotkey --id 0000 --pin 1234
   ```

   This command identifies the slot as `0` and defines the key label as `iotkey`\. You use these values in the next section\.

After the private key is imported, you can optionally remove it from the `/greengrass/certs` directory\. Make sure to keep the root CA and device certificates in the directory\.

## Configure the Greengrass core to use SoftHSM<a name="softhsm-config-core"></a>

In this step, you modify the Greengrass core configuration file to use SoftHSM\.

1. Find the path to the SoftHSM provider library \(`libsofthsm2.so`\) on your system:

   1. Get the list of installed packages for the library\.

      ```
      sudo dpkg -L libsofthsm2
      ```

      The `libsofthsm2.so` file is located in the `softhsm` directory\.

   1. Copy the full path to the file \(for example, `/usr/lib/x86_64-linux-gnu/softhsm/libsofthsm2.so`\)\. You use this value later\.

1. Stop the Greengrass daemon\.

   ```
   cd /greengrass/ggc/core/
   sudo ./greengrassd stop
   ```

1. Open the Greengrass configuration file\. This is the [`config.json`](gg-core.md#config-json) file in the `/greengrass/config` directory\.
**Note**  
The examples in this procedure are written with the assumption that the `config.json` file uses the format that's generated from the **Default Group creation** option in [Module 2](module2.md) of the Getting Started tutorial\.

1. In the `crypto.principals` object, insert the following MQTT server certificate object\. Add a comma where needed to create a valid JSON file\.

   ```
     "MQTTServerCertificate": {
       "privateKeyPath": "path-to-private-key"
     }
   ```

1. In the `crypto` object, insert the following `PKCS11` object\. Add a comma where needed to create a valid JSON file\.

   ```
     "PKCS11": {
       "P11Provider": "/path-to-pkcs11-provider-so",
       "slotLabel": "crypto-token-name",
       "slotUserPin": "crypto-token-user-pin"
     }
   ```

   Your file should look similar to the following:

   ```
   {
     "coreThing" : {
       "caPath" : "root.ca.pem",
       "certPath" : "hash.cert.pem",
       "keyPath" : "hash.private.key",
       "thingArn" : "arn:partition:iot:region:account-id:thing/core-thing-name",
       "iotHost" : "host-prefix.iot.region.amazonaws.com",
       "ggHost" : "greengrass.iot.region.amazonaws.com",
       "keepAlive" : 600
     },
     "runtime" : {
       "cgroup" : {
         "useSystemd" : "yes"
       }
     },
     "managedRespawn" : false,
     "crypto": {
       "PKCS11": {
         "P11Provider": "/path-to-pkcs11-provider-so",
         "slotLabel": "crypto-token-name",
         "slotUserPin": "crypto-token-user-pin"
       },
       "principals" : {
         "MQTTServerCertificate": {
           "privateKeyPath": "path-to-private-key"
         },
         "IoTCertificate" : {
           "privateKeyPath" : "file:///greengrass/certs/hash.private.key",
           "certificatePath" : "file:///greengrass/certs/hash.cert.pem"
         },
         "SecretsManager" : {
           "privateKeyPath" : "file:///greengrass/certs/hash.private.key"
         }
       },    
       "caPath" : "file:///greengrass/certs/root.ca.pem"
     }
   }
   ```
**Note**  
To use over\-the\-air \(OTA\) updates with hardware security, the `PKCS11` object must also contain the `OpenSSLEngine` property\. For more information, see [Configure support for over\-the\-air updates](hardware-security.md#hardware-security-ota-updates)\.

1. Edit the `crypto` object:

   1. Configure the `PKCS11` object\.
      + For `P11Provider`, enter the full path to `libsofthsm2.so`\.
      + For `slotLabel`, enter `greengrass`\.
      + For `slotUserPin`, enter `1234`\.

   1. Configure the private key paths in the `principals` object\. Do not edit the `certificatePath` property\.
      + For the `privateKeyPath` properties, enter the following RFC 7512 PKCS\#11 path \(which specifies the key's label\)\. Do this for the `IoTCertificate`, `SecretsManager`, and `MQTTServerCertificate` principals\.

        ```
        pkcs11:object=iotkey;type=private
        ```

   1. Check the `crypto` object\. It should look similar to the following:

      ```
        "crypto": {
          "PKCS11": {
            "P11Provider": "/usr/lib/x86_64-linux-gnu/softhsm/libsofthsm2.so",
            "slotLabel": "greengrass",
            "slotUserPin": "1234"
          },
          "principals": {
            "MQTTServerCertificate": {
              "privateKeyPath": "pkcs11:object=iotkey;type=private"
            },
            "SecretsManager": {
              "privateKeyPath": "pkcs11:object=iotkey;type=private"
            },
            "IoTCertificate": {
              "certificatePath": "file://certs/core.crt",
              "privateKeyPath": "pkcs11:object=iotkey;type=private"
            }
          },
          "caPath": "file://certs/root.ca.pem"
        }
      ```

1. Remove the `caPath`, `certPath`, and `keyPath` values from the `coreThing` object\. It should look similar to the following:

   ```
   "coreThing" : {
     "thingArn" : "arn:partition:iot:region:account-id:thing/core-thing-name",
     "iotHost" : "host-prefix-ats.iot.region.amazonaws.com",
     "ggHost" : "greengrass-ats.iot.region.amazonaws.com",
     "keepAlive" : 600
   }
   ```

**Note**  
For this tutorial, you specify the same private key for all principals\. For more information about choosing the private key for the local MQTT server, see [Performance](hardware-security.md#hsm-performance)\. For more information about the local secrets manager, see [Deploy secrets to the AWS IoT Greengrass core](secrets.md)\.

## Test the configuration<a name="softhsm-test"></a>
+ Start the AWS Greengrass daemon\.

  ```
  cd /greengrass/ggc/core/
  sudo ./greengrassd start
  ```

  If the daemon starts successfully, then your core is configured correctly\.

  You are now ready to learn about the PKCS\#11 specification and do initial testing with the PKCS\#11 API that's provided by the SoftHSM implementation\.
**Important**  
Again, it's extremely important to be aware that this module is intended for learning and testing only\. It doesn't actually increase the security posture of your Greengrass environment\.  
Instead, the purpose of the module is to enable you to start learning and testing in preparation for using a true hardware\-based HSM in the future\. At that time, you must separately and completely test your software against the hardware\-based HSM prior to any production usage, because there might be differences between the PKCS\#11 implementation provided in SoftHSM and a hardware\-based implementation\.

## See also<a name="console-mod7-see-also"></a>
+ *PKCS \#11 Cryptographic Token Interface Usage Guide Version 2\.40*\. Edited by John Leiseboer and Robert Griffin\. 16 November 2014\. OASIS Committee Note 02\. [ http://docs\.oasis\-open\.org/pkcs11/pkcs11\-ug/v2\.40/cn02/pkcs11\-ug\-v2\.40\-cn02\.html](http://docs.oasis-open.org/pkcs11/pkcs11-ug/v2.40/cn02/pkcs11-ug-v2.40-cn02.html)\. Latest version: [ http://docs\.oasis\-open\.org/pkcs11/pkcs11\-ug/v2\.40/pkcs11\-ug\-v2\.40\.html](http://docs.oasis-open.org/pkcs11/pkcs11-ug/v2.40/pkcs11-ug-v2.40.html)\.
+ [RFC 7512](https://tools.ietf.org/html/rfc7512)