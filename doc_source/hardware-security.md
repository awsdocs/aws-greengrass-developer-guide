# Hardware Security Integration<a name="hardware-security"></a>

This feature is available for AWS IoT Greengrass Core v1\.7\.0 only\.

AWS IoT Greengrass supports the use of hardware security modules \(HSM\) through the [PKCS\#11 interface](#hardware-security-see-also) for secure storage and offloading of private keys\. This prevents keys from being exposed or duplicated in software\. Private keys can be securely stored on hardware modules, such as HSMs, Trusted Platform Modules \(TPM\), or other cryptographic elements\.

Search for devices that are qualified for this feature in the [AWS Partner Device Catalog](https://devices.amazonaws.com/search?kw=%22HSI%22&page=1)\.

The following diagram shows the hardware security architecture for an AWS IoT Greengrass core\.

![\[\]](http://docs.aws.amazon.com/greengrass/latest/developerguide/images/hardware-security-arch.png)

On a standard installation, AWS IoT Greengrass uses two private keys\. One key is used by the AWS IoT client component during the Transport Layer Security \(TLS\) handshake when a Greengrass core connects to AWS IoT\. \(This key is also referred to as the core private key\.\) The other key is used by the local MQTT server, which enables Greengrass devices to communicate with the Greengrass core\. If you want to use hardware security for both components, you can use a shared private key or separate private keys\. For more information, see [Provisioning Practices for AWS IoT Greengrass Hardware Security](#optional-provisioning)\.

## Requirements<a name="hardware-security-reqs"></a>

Before you can configure hardware security for a Greengrass core, you must have the following:
+ An HSM that supports an RSA\-2048 key size \(or larger\) and [PKCS\#1 v1\.5](#hardware-security-see-also) signature scheme\.

  AWS IoT Greengrass supports both RSA and ECC private keys for IoT client message encryption\. This applies to messages sent between the Greengrass core and AWS IoT cloud\. However, AWS IoT Greengrass doesn't currently support ECC private keys for local MQTT server message encryption, which applies to messages between the Greengrass core and other Greengrass devices in the group\. Therefore, if your secure element doesn't support RSA key storage, you won't be able to use the secure element key for local MQTT server message encryption\. For these messages, you need to store an RSA private key in the file system\.
**Note**  
Search for devices that are qualified for this feature in the [AWS Partner Device Catalog](https://devices.amazonaws.com/search?kw=%22HSI%22&page=1)\.
+ A PKCS\#11 provider library that is loadable at runtime \(using libdl\) and provides [PKCS\#11](#hardware-security-see-also) functions\.
+ The hardware module must be resolvable by slot label, as defined in the PKCS\#11 specification\.
+ The private key must be generated and loaded on the HSM by using the vendor\-provided provisioning tools\.
+ The private key must be resolvable by object label\.
+ The core device certificate\. This is an AWS IoT client certificate that corresponds to the private key\.
+ If you're using the Greengrass OTA update agent, the [OpenSSL libp11 PKCS\#11](https://github.com/OpenSC/libp11) wrapper library must be installed\. For more information, see [Configure Support for Over\-the\-Air Updates](#hardware-security-ota-updates)\.

In addition, make sure that the following conditions are met:
+ The IoT client certificates that are associated with the private key are registered in AWS IoT and activated\. You can verify this from the **Manage** page for the core thing in the AWS IoT console\.
+ The AWS IoT Greengrass core software \(v1\.7\.0\) is installed on the core device, as described in [Module 2](module2.md) of the Getting Started tutorial\.
+ The certificates are attached to the Greengrass core\. You can verify this from the **Manage** page for the core thing in the AWS IoT console\.

**Note**  
Currently, AWS IoT Greengrass doesn't support loading the CA certificate or AWS IoT client certificate directly from the HSM\. The certificates must be loaded as plain\-text files on the file system in a location that can be read by Greengrass\.

## Hardware Security Configuration for an AWS IoT Greengrass Core<a name="configure-hardware-security"></a>

Hardware security is configured in the Greengrass configuration file\. This is the [`config.json`](gg-core.md#config-json) file that's located in the `/greengrass-root/config` directory\.

**Note**  
To walk through the process of setting up an HSM configuration using a pure software implementation, see [Module 7: Simulating Hardware Security Integration](console-mod7.md)\.  
The simulated configuration in the example doesn't provide any security benefits\. It's intended to allow you to learn about the PKCS\#11 specification and do initial testing of your software if you plan to use a hardware\-based HSM in the future\.

To configure hardware security in AWS IoT Greengrass, you edit the `crypto` object in `config.json`\.

When using hardware security, the `crypto` object is used to specify paths to certificates, private keys, and assets for the PKCS\#11 provider library on the core, as shown in the following example\.

```
"crypto": {
  "PKCS11" : {
    "OpenSSLEngine" : "/path-to-p11-openssl-engine",
    "P11Provider" : "/path-to-pkcs11-provider-so",
    "slotLabel" : "crypto-token-name",
    "slotUserPin" : "crypto-token-user-pin"
  },
  "principals" : {
    "IoTCertificate" : {
      "privateKeyPath" : "pkcs11:object=core-private-key-label;type=private",
      "certificatePath" : "file:///path-to-core-device-certificate"
    },
    "MQTTServerCertificate" : {
      "privateKeyPath" : "pkcs11:object=server-private-key-label;type=private",
    },
    "SecretsManager" : {
      "privateKeyPath": "pkcs11:object=core-private-key-label;type=private"
    }
  },    
  "caPath" : "file:///path-to-root-ca"
```

The `crypto` object contains the following properties:


| Field | Description | Notes | 
| --- | --- | --- | 
| <a name="config-capath"></a>caPath |  The absolute path to the AWS IoT root CA\.  |  Must be a file URI of the form: `file:///absolute/path/to/file`\. Make sure that your [endpoints correspond to your certificate type](gg-core.md#certificate-endpoints)\.  | 
| PKCS11 | 
| OpenSSLEngine |  Optional\. The absolute path to the OpenSSL engine `.so` file to enable PKCS\#11 support on OpenSSL\. This is used by the Greengrass OTA update agent\.  |  Must be a path to a file on the file system\.  | 
| P11Provider |  The absolute path to the PKCS\#11 implementation's libdl\-loadable library\.  |  Must be a path to a file on the file system\.  | 
| slotLabel |  The slot label that's used to identify the hardware module\.  |  Must conform to PKCS\#11 label specifications\.  | 
| slotUserPin |  The user pin that's used to authenticate the Greengrass core to the module\.  |  Must have sufficient permissions to perform C\_Sign with the configured private keys\.  | 
| principals | 
| IoTCertificate | The certificate and private key that the core uses to make requests to AWS IoT\. | 
| IoTCertificate  \.privateKeyPath  |  The path to the core private key\.  |  For file system storage, must be a file URI of the form: `file:///absolute/path/to/file`\. For HSM storage, must be an [RFC 7512 PKCS\#11](https://tools.ietf.org/html/rfc7512) path that specifies the object label\.  | 
| IoTCertificate  \.certificatePath |  The absolute path to the core device certificate\.  |  Must be a file URI of the form: `file:///absolute/path/to/file`\.  | 
| MQTTServerCertificate | Optional\. The private key that the core uses in combination with the certificate to act as an MQTT server or gateway\. | 
| MQTTServerCertificate  \.privateKeyPath |  The path to the local MQTT server private key\.  |  Use this value to specify your own private key for the local MQTT server\. For file system storage, must be a file URI of the form: `file:///absolute/path/to/file`\. For HSM storage, must be an [RFC 7512 PKCS\#11](https://tools.ietf.org/html/rfc7512) path that specifies the object label\. If this property is omitted, AWS IoT Greengrass rotates the key based your rotation settings\. If specified, the customer is responsible for rotating the key\.  | 
| SecretsManager | The private key that secures the data key used for encryption\. For more information, see [Deploy Secrets to the AWS IoT Greengrass Core](secrets.md)\. | 
| SecretsManager  \.privateKeyPath |  The path to the local secrets manager private key\.  |  For file system storage, must be a file URI of the form: `file:///absolute/path/to/file`\. For HSM storage, must be an [RFC 7512 PKCS\#11](https://tools.ietf.org/html/rfc7512) path that specifies the object label\. The private key must be generated using the [PKCS\#1 v1\.5](https://tools.ietf.org/html/rfc2313) padding mechanism\.  | 

## Provisioning Practices for AWS IoT Greengrass Hardware Security<a name="optional-provisioning"></a>

The following are security and performance\-related provisioning practices\.

**Security**  
+ Generate private keys directly on the HSM by using the internal hardware random\-number generator\.
**Note**  
If you configure private keys to use with this feature \(by following the instructions provided by the hardware vendor\), be aware that AWS IoT Greengrass currently supports only the PKCS1 v1\.5 padding mechanism for encryption and decryption of [local secrets](secrets.md)\. AWS IoT Greengrass doesn't support Optimal Asymmetric Encryption Padding \(OAEP\)\.
+ Configure private keys to prohibit export\.
+ Use the provisioning tool that's provided by the hardware vendor to generate a certificate signing request \(CSR\) using the hardware\-protected private key, and then use the AWS IoT console to generate a client certificate\.
The practice of rotating keys doesn't apply when private keys are generated on an HSM\.

**Performance**  <a name="hsm-performance"></a>
The following diagram shows the AWS IoT client component and local MQTT server on the AWS IoT Greengrass core\. If you want to use an HSM configuration for both components, you can use the same private key or separate private keys\.  
AWS IoT Greengrass supports both RSA and ECC private keys for IoT client message encryption\. This applies to messages sent between the Greengrass core and AWS IoT cloud\. However, AWS IoT Greengrass doesn't currently support ECC private keys for local MQTT server message encryption, which applies to messages between the Greengrass core and other Greengrass devices in the group\. Therefore, if your secure element doesn't support RSA key storage, you won't be able to use the secure element key for local MQTT server message encryption\. For these messages, you need to store an RSA private key in the file system\.

![\[\]](http://docs.aws.amazon.com/greengrass/latest/developerguide/images/multi-key-diagram.png)
In general, the IoT client key is not used very frequently because the Greengrass core software maintains long\-lived connections to the cloud\. However, the MQTT server key is used every time that a Greengrass device connects to the core\. These interactions directly affect performance\.  
When the MQTT server key is stored on the HSM, the rate at which devices can connect depends on the number of RSA signature operations per second that the HSM can perform\. For example, if the HSM takes 300 milliseconds to perform an RSASSA\-PKCS1\-v1\.5 signature on an RSA\-2048 private key, then only three devices can connect to the Greengrass core per second\. After the connections are made, the HSM is no longer used and the standard [Greengrass limits](https://docs.aws.amazon.com/general/latest/gr/aws_service_limits.html#limits_greengrass) apply\.  
To mitigate performance bottlenecks, you can store the private key for the MQTT server on the file system instead of on the HSM\. With this configuration, the MQTT server behaves as if hardware security isn't enabled\.  
AWS IoT Greengrass supports the following key\-storage configurations so you can optimize for your security and performance requirements\.      
[\[See the AWS documentation website for more details\]](http://docs.aws.amazon.com/greengrass/latest/developerguide/hardware-security.html)
To configure the Greengrass core to use file system\-based keys for the MQTT server, omit the `principals.MQTTServerCertificate` section from `config.json` \(or specify a file\-based path to the key if you're not using the default key generated by AWS IoT Greengrass\)\. The resulting `crypto` object looks like this:  

```
"crypto": {
  "PKCS11": {
    "OpenSSLEngine": "...",
    "P11Provider": "...",
    "slotLabel": "...",
    "slotUserPin": "...",
  },
  "principals": {
    "IoTCertificate": {
      "privateKeyPath": "...",
      "certificatePath": "..."
    },      
    "SecretsManager": {
      "privateKeyPath": "..."
    }
  },    
  "caPath" : "..."
}
```

## Supported Cipher Suites for Hardware Security Integration<a name="cipher-suites-for-hsm"></a>

AWS IoT Greengrass supports the following cipher suites when using hardware security:

[\[See the AWS documentation website for more details\]](http://docs.aws.amazon.com/greengrass/latest/developerguide/hardware-security.html)

When connecting to the Greengrass core from Greengrass devices, be sure to use one of these cipher suites to make the TLS connection\.

**Note**  
This is a subset of the cipher suites that are supported when the core is configured to use file\-based security\. For the full list, see [AWS IoT Greengrass Cipher Suites](gg-sec.md#gg-cipher-suites)\.

## Configure Support for Over\-the\-Air Updates<a name="hardware-security-ota-updates"></a>

To enable over\-the\-air \(OTA\) updates of the AWS IoT Greengrass core core software when using hardware security, you must install the OpenSC libp11 [PKCS\#11 wrapper library](https://github.com/OpenSC/libp11) and edit the Greengrass configuration file\. For more information about OTA updates, see [OTA Updates of AWS IoT Greengrass Core Software](core-ota-update.md)\.

1. Stop the AWS Greengrass daemon\.

   ```
   cd /greengrass-root/ggc/core/
   sudo ./greengrassd stop
   ```
**Note**  
*greengrass\-root* represents the path where the AWS IoT Greengrass core software is installed on your device\. If you installed the software by following the [Getting Started](gg-gs.md) tutorial, then this is the `/greengrass` directory\.

1. Install the OpenSSL engine\.

   ```
   sudo apt-get install libengine-pkcs11-openssl
   ```

1. Find the path to the OpenSSL engine \(`libpkcs11.so`\) on your system:

   1. Get the list of installed packages for the library\.

      ```
      sudo dpkg -L libengine-pkcs11-openssl
      ```

      The `libpkcs11.so` file is located in the `engines` directory\.

   1. Copy the full path to the file \(for example, `/usr/lib/ssl/engines/libpkcs11.so`\)\.

1. Open the Greengrass configuration file\. This is the [`config.json`](gg-core.md#config-json) file in the `/greengrass-root/config` directory\.

1. For the `OpenSSLEngine` property, enter the path to the `libpkcs11.so` file\.

   ```
   {
    "crypto": {
      "caPath" : "file:///path-to-root-ca",
      "PKCS11" : {
        "OpenSSLEngine" : "/path-to-p11-openssl-engine",
        "P11Provider" : "/path-to-pkcs11-provider-so",
        "slotLabel" : "crypto-token-name",
        "slotUserPin" : "crypto-token-user-pin"
       },
       ...
     }
     ...
   }
   ```
**Note**  
If the `OpenSSLEngine` property doesn't exist in the `PKCS11` object, then add it\.

1. Start the AWS Greengrass daemon\.

   ```
   cd /greengrass-root/ggc/core/
   sudo ./greengrassd start
   ```

## Backward Compatibility with Earlier Versions of the AWS IoT Greengrass Core Software<a name="hardware-security-backward-compatibiity"></a>

The AWS IoT Greengrass core software with hardware security support is fully backward compatible with `config.json` files that are generated for v1\.6\.0 and earlier\. If the `crypto` object is not present in the `config.json` configuration file, then AWS IoT Greengrass uses the file\-based `coreThing.certPath`, `coreThing.keyPath`, and `coreThing.caPath` properties\. This backward compatibility applies to Greengrass OTA updates, which do not overwrite a file\-based configuration that's specified in `config.json`\.

## Hardware Without PKCS\#11 Support<a name="hardware-without-pkcs11"></a>

The PKCS\#11 library is typically provided by the hardware vendor or is open source\. For example, with standards\-compliant hardware \(such as TPM1\.2\), it might be possible to use existing open source software\. However, if your hardware doesn't have a corresponding PKCS\#11 library implementation, or if you want to write a custom PKCS\#11 provider, you should contact your AWS Enterprise Support representative with integration\-related questions\.

## See Also<a name="hardware-security-see-also"></a>
+ *PKCS \#11 Cryptographic Token Interface Usage Guide Version 2\.40*\. Edited by John Leiseboer and Robert Griffin\. 16 November 2014\. OASIS Committee Note 02\. [ http://docs\.oasis\-open\.org/pkcs11/pkcs11\-ug/v2\.40/cn02/pkcs11\-ug\-v2\.40\-cn02\.html](http://docs.oasis-open.org/pkcs11/pkcs11-ug/v2.40/cn02/pkcs11-ug-v2.40-cn02.html)\. Latest version: [ http://docs\.oasis\-open\.org/pkcs11/pkcs11\-ug/v2\.40/pkcs11\-ug\-v2\.40\.html](http://docs.oasis-open.org/pkcs11/pkcs11-ug/v2.40/pkcs11-ug-v2.40.html)\.
+ [RFC 7512](https://tools.ietf.org/html/rfc7512)
+ [PKCS \#1: RSA Encryption Version 1\.5](https://tools.ietf.org/html/rfc2313)