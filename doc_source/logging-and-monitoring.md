# Logging and monitoring in AWS IoT Greengrass<a name="logging-and-monitoring"></a>

Monitoring is an important part of maintaining the reliability, availability, and performance of AWS IoT Greengrass and your AWS solutions\. You should collect monitoring data from all parts of your AWS solution so that you can more easily debug a multi\-point failure, if one occurs\. Before you start monitoring AWS IoT Greengrass, you should create a monitoring plan that includes answers to the following questions:
+ What are your monitoring goals?
+ Which resources will you monitor?
+ How often will you monitor these resources?
+ Which monitoring tools will you use?
+ Who will perform the monitoring tasks?
+ Who should be notified when something goes wrong?

## Monitoring tools<a name="monitoring_automated_manual"></a>

AWS provides tools that you can use to monitor AWS IoT Greengrass\. You can configure some of these tools to do the monitoring for you\. Some of the tools require manual intervention\. We recommend that you automate monitoring tasks as much as possible\.

You can use the following automated monitoring tools to monitor AWS IoT Greengrass and report issues:
+ **Amazon CloudWatch Logs** – Monitor, store, and access your log files from AWS CloudTrail or other sources\. For more information, see [Monitoring log files](https://docs.aws.amazon.com/AmazonCloudWatch/latest/DeveloperGuide/WhatIsCloudWatchLogs.html) in the *Amazon CloudWatch User Guide*\.
+ **AWS CloudTrail Log Monitoring** – Share log files between accounts, monitor CloudTrail log files in real time by sending them to CloudWatch Logs, write log processing applications in Java, and validate that your log files have not changed after delivery by CloudTrail\. For more information, see [Working with CloudTrail log files](https://docs.aws.amazon.com/awscloudtrail/latest/userguide/cloudtrail-working-with-log-files.html) in the *AWS CloudTrail User Guide*\.
+ **Amazon EventBridge** – Use EventBridge event rules to get notifications about state changes for your Greengrass group deployments or API calls logged with CloudTrail\. For more information, see [Get deployment notifications](deployment-notifications.md) or [What is Amazon EventBridge?](https://docs.aws.amazon.com/eventbridge/latest/userguide/what-is-amazon-eventbridge.html) in the *Amazon EventBridge User Guide*\.
+ **Greengrass system health telemetry** – Subscribe to receive telemetry data sent from the Greengrass core\. For more information, see [Gathering system health telemetry data from AWS IoT Greengrass core devices](telemetry.md)\.
+ **Local health check** – Use the health APIs to get a snapshot of the state of local AWS IoT Greengrass processes on the core device\. For more information, see [Calling the local health check API](health-check.md)\.

## See also<a name="logging-and-monitoring-see-also"></a>
+ [Monitoring with AWS IoT Greengrass logs](greengrass-logs-overview.md)
+ [Logging AWS IoT Greengrass API calls with AWS CloudTrail](logging-using-cloudtrail.md)
+ [Get deployment notifications](deployment-notifications.md)