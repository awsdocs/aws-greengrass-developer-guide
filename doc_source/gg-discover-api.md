# Greengrass Discovery RESTful API<a name="gg-discover-api"></a>

All devices that communicate with an AWS IoT Greengrass core must be a member of a Greengrass group\. Each group must have a Greengrass core\. The Discovery API enables devices to retrieve information required to connect to a Greengrass core that is in the same Greengrass group as the device\. When a device first comes online, it can connect to the AWS IoT Greengrass service and use the Discovery API to find:
+ The group to which it belongs\. A device can be a member of up to 10 groups\.
+ The IP address and port for the Greengrass core in the group\.
+ The group CA certificate, which can be used to authenticate the Greengrass core device\.

**Note**  
Devices can also use the AWS IoT Device SDKs to discover connectivity information for a Greengrass core\. For more information, see [AWS IoT Device SDK](what-is-gg.md#iot-device-sdk)\.

To use this API, send HTTP requests to the Discovery API endpoint\. For example:

```
https://greengrass-ats.iot.region.amazonaws.com:port/greengrass/discover/thing/thing-name
```

For a list of supported AWS Regions and endpoints for the AWS IoT Greengrass Discovery API, see [AWS IoT Greengrass endpoints and quotas](https://docs.aws.amazon.com/general/latest/gr/greengrass.html) in the *AWS General Reference*\. This is a data plane only API\. The endpoints for group management and AWS IoT Core operations are different from the Discovery API endpoints\.

## Request<a name="gg-discover-request"></a>

The request contains the standard HTTP headers and is sent to the Greengrass Discovery endpoint, as shown in the following examples\.

The port number depends on whether the core is configured to send HTTPS traffic over port 8443 or port 443\. For more information, see [Connect on port 443 or through a network proxy](gg-core.md#alpn-network-proxy)\.

Port 8443  

```
HTTP GET https://greengrass-ats.iot.region.amazonaws.com:8443/greengrass/discover/thing/thing-name
```

Port 443  

```
HTTP GET https://greengrass-ats.iot.region.amazonaws.com:443/greengrass/discover/thing/thing-name
```
Clients that connect on port 443 must implement the [ Application Layer Protocol Negotiation \(ALPN\)](https://tools.ietf.org/html/rfc7301) TLS extension and pass `x-amzn-http-ca` as the `ProtocolName` in the `ProtocolNameList`\. For more information, see [Protocols](https://docs.aws.amazon.com/iot/latest/developerguide/protocols.html) in the *AWS IoT Developer Guide*\.  
These examples use the Amazon Trust Services \(ATS\) endpoint, which is used with ATS root CA certificates \(recommended\)\. Endpoints must match the root CA certificate type\. For more information, see [Service endpoints must match the root CA certificate type](gg-core.md#certificate-endpoints)\.

## Response<a name="gg-discover-response"></a>

Upon success, the response includes the standard HTTP headers plus the following code and body:

```
HTTP 200
BODY: response document
```

For more information, see [Example discover response documents](#gg-discover-response-doc)\.

## Discovery authorization<a name="gg-discover-auth"></a>

Retrieving the connectivity information requires a policy that allows the caller to perform the `greengrass:Discover` action\. TLS mutual authentication with a client certificate is the only accepted form of authentication\. The following is an example policy that allows a caller to perform this action:

```
{
    "Version": "2012-10-17",
    "Statement": [{
        "Effect": "Allow",
        "Action": "greengrass:Discover",
        "Resource": ["arn:aws:iot:us-west-2:123456789012:thing/MyThingName"]
     }]
}
```

## Example discover response documents<a name="gg-discover-response-doc"></a>

The following document shows the response for a device that is a member of a group with one Greengrass core, one endpoint, and one group CA certificate:

```
{
  "GGGroups": [
    {
      "GGGroupId": "gg-group-01-id",
      "Cores": [
        {
          "thingArn": "core-01-thing-arn",
          "Connectivity": [
            {
              "id": "core-01-connection-id",
              "hostAddress": "core-01-address",
              "portNumber": core-01-port,
              "metadata": "core-01-description"
            }
          ]
        }
      ],
      "CAs": [
        "-----BEGIN CERTIFICATE-----cert-contents-----END CERTIFICATE-----"
      ]
    }
  ]
}
```

The following document shows the response for a device that is a member of two groups with one Greengrass core, multiple endpoints, and multiple group CA certificates:

```
{
  "GGGroups": [
    {
      "GGGroupId": "gg-group-01-id",
      "Cores": [
        {
          "thingArn": "core-01-thing-arn",
          "Connectivity": [
            {
              "id": "core-01-connection-id",
              "hostAddress": "core-01-address",
              "portNumber": core-01-port,
              "metadata": "core-01-connection-1-description"
            },
            {
              "id": "core-01-connection-id-2",
              "hostAddress": "core-01-address-2",
              "portNumber": core-01-port-2,
              "metadata": "core-01-connection-2-description"
            }
          ]
        }
      ],
      "CAs": [
        "-----BEGIN CERTIFICATE-----cert-contents-----END CERTIFICATE-----",
        "-----BEGIN CERTIFICATE-----cert-contents-----END CERTIFICATE-----",
        "-----BEGIN CERTIFICATE-----cert-contents-----END CERTIFICATE-----"
      ]
    },
    {
      "GGGroupId": "gg-group-02-id",
      "Cores": [
         {
            "thingArn":"core-02-thing-arn",
            "Connectivity" : [
            {
              "id": "core-02-connection-id",
              "hostAddress": "core-02-address",
              "portNumber": core-02-port,
              "metadata": "core-02-connection-1-description"
            }
            ],
            "CAs": [
                "-----BEGIN CERTIFICATE-----cert-contents-----END CERTIFICATE-----",
                "-----BEGIN CERTIFICATE-----cert-contents-----END CERTIFICATE-----",
                "-----BEGIN CERTIFICATE-----cert-contents-----END CERTIFICATE-----"
            ]
        }
    ]
    }
}
```

**Note**  
A Greengrass group must define exactly one Greengrass core\. Any response from the AWS IoT Greengrass service that contains a list of Greengrass cores contains only one Greengrass core\. 

If you have `cURL` installed, you can test the discovery request\. For example:

```
$ curl --cert 1a23bc4d56.cert.pem --key 1a23bc4d56.private.key https://greengrass-ats.iot.us-west-2.amazonaws.com:8443/greengrass/discover/thing/MyDevice
{"GGGroups":[{"GGGroupId":"1234a5b6-78cd-901e-2fgh-3i45j6k1789","Cores":[{"thingArn":"arn:aws:iot:us-west-2:1234567
89012:thing/MyFirstGroup_Core","Connectivity":[{"Id":"AUTOIP_192.168.1.4_1","HostAddress":"192.168.1.5","PortNumber
":8883,"Metadata":""}]}],"CAs":["-----BEGIN CERTIFICATE-----\ncert-contents\n-----END CERTIFICATE-----\n"]}]}
```