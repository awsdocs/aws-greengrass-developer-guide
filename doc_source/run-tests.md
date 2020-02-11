# Running Tests<a name="run-tests"></a>

After you set the required configuration, you can start the tests\. The run time of the full test suite depends on your hardware\. For reference, it takes approximately 30 minutes to complete the full test suite on a Raspberry Pi 3B\.

The following example command line shows you how to run the qualification tests for a device pool \(a set of identical devices\)\. You can find these commands in the `<devicetester-extract-location>/bin` directory\.

Use the following command to run all test groups in a specified suite:  

```
devicetester_[linux | mac | win_x86-64] run-suite --suite-id GGQ_1 --pool-id <pool-id>
```

Use the following command to run a specific test group:  

```
devicetester_[linux | mac | win_x86-64] run-suite --suite-id GGQ_1 --group-id <group-id> --pool-id <pool-id>
```
`suite-id` and `pool-id` are optional if you are running a single test suite on a single device pool \(that is, you have only one device pool defined in your `device.json` file\)\.  
We recommend that you to run the dependency checker test group to make sure all Greengrass dependencies are installed before you run related test groups\. For example:  
+ Run `ggcdependencies` before core qualification test groups\.
+ Run `containerdependencies` before container\-specific test groups\.
+ Run `dockerdependencies` before Docker\-specific test groups\.
+ Run `ggcstreammanagementdependencies` before stream manager\-specific test groups\.

## AWS IoT Device Tester for AWS IoT Greengrass Commands<a name="bk-cli"></a>

The IDT commands can be used for the following operations:

help  
Lists information about the specified command\.

list\-groups  
Lists the groups in a given test suite\.

list\-suites  
Lists the available test suites\.

run\-suite  
Runs a suite of tests on a pool of devices\.