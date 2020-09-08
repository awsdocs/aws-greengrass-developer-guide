# Optional: Configuring your device for ML qualification<a name="idt-ml-qualification"></a>

IDT for AWS IoT Greengrass provides machine learning \(ML\) qualification tests to validate that your devices can perform ML inference locally using cloud\-trained models\.

To run ML qualification tests, you must first configure your devices as described in [Configuring your device](device-config-setup.md)\. Then, follow the steps in this topic to install dependencies for the ML frameworks that you want to run\.

IDT v3\.1\.0 or later is required to run tests for ML qualification\.

## Installing ML framework dependencies<a name="ml-qualification-framework-dependencies"></a>

All ML framework dependencies must be installed under the `/usr/local/lib/python3.x/site-packages` directory\. To make sure they are installed under the correct directory, we recommend that you use `sudo` root permissions when installing the dependencies\. Virtual environments are not supported for qualification tests\.

**Note**  
If you're testing Lambda functions that run with [containerization](lambda-group-config.md#lambda-containerization-considerations) \(in **Greengrass container** mode\), creating symlinks for Python libraries under `/usr/local/lib/python3.x` isn't supported\. To avoid errors, you must install the dependencies under the correct directory\.

Follow the steps to install the dependencies for your target framework:
+ [Install MXNet dependencies](#ml-qualification-mxnet-dependencies)
+ [Install TensorFlow dependencies](#ml-qualification-tensorflow-dependencies)
+ [Install DLR dependencies](#ml-qualification-dlr-dependencies)

 

## Install Apache MXNet dependencies<a name="ml-qualification-mxnet-dependencies"></a>

<a name="test-framework-dependencies"></a>IDT qualification tests for this framework have the following dependencies:
+ <a name="ml-qualification-python-req"></a>Python 3\.6 or Python 3\.7\.
**Note**  <a name="python-symlink-command"></a>
If you're using Python 3\.6, you must create a symbolic link from Python 3\.7 to Python 3\.6 binaries\. This configures your device to meet the Python requirement for AWS IoT Greengrass\. For example:  

  ```
  sudo ln -s path-to-python-3.6/python3.6 path-to-python-3.7/python3.7
  ```
+ Apache MXNet v1\.2\.1 or later\.
+ NumPy\. The version must be compatible with your MXNet version\.

### Installing MXNet<a name="ml-qualification-mxnet-install"></a>

Follow the instructions in the MXNet documentation to [install MXNet](https://mxnet.apache.org/get_started/?platform=linux&language=python&processor=cpu&environ=pip&)\.

**Note**  
<a name="run-python3-commands"></a>If Python 2\.x and Python 3\.x are both installed on your device, use Python 3\.x in the commands that you run to install the dependencies\.

### Validating the MXNet installation<a name="ml-qualification-mxnet-validate"></a>

Choose one of the following options to validate the MXNet installation\.

#### Option 1: SSH into your device and run scripts<a name="ml-qualification-validate-mxnet-option-1"></a>

1. <a name="ssh-validate-framework-install-ssh"></a>SSH into your device\.

1. <a name="ssh-validate-framework-install-run-scripts"></a>Run the following scripts to verify that the dependencies are correctly installed\.

   ```
   sudo python3.7 -c "import mxnet; print(mxnet.__version__)"
   ```

   ```
   sudo python3.7 -c "import numpy; print(numpy.__version__)"
   ```

   <a name="ssh-passed-mldependencies"></a>The output prints the version number and the script should exit without error\.

#### Option 2: Run the IDT dependency test<a name="ml-qualification-validate-mxnet-option-2"></a>

1. <a name="idt-validate-framework-install-check-config"></a>Make sure that `device.json` is configured for ML qualification\. For more information, see [Configure device\.json for ML qualification](set-config.md#device-json-ml-qualification)\.

1. <a name="idt-validate-framework-install-run-test"></a>Run the dependencies test for the framework\.

   ```
   devicetester_[linux | mac | win_x86-64] run-suite --group-id mldependencies --test-id mxnet_dependency_check
   ```

   <a name="idt-passed-mldependencies"></a>The test summary displays a `PASSED` result for `mldependencies`\.

 

## Install TensorFlow dependencies<a name="ml-qualification-tensorflow-dependencies"></a>

<a name="test-framework-dependencies"></a>IDT qualification tests for this framework have the following dependencies:
+ <a name="ml-qualification-python-req"></a>Python 3\.6 or Python 3\.7\.
**Note**  <a name="python-symlink-command"></a>
If you're using Python 3\.6, you must create a symbolic link from Python 3\.7 to Python 3\.6 binaries\. This configures your device to meet the Python requirement for AWS IoT Greengrass\. For example:  

  ```
  sudo ln -s path-to-python-3.6/python3.6 path-to-python-3.7/python3.7
  ```
+ TensorFlow 1\.x\.

### Installing TensorFlow<a name="ml-qualification-tensorflow-install"></a>

Follow the instructions in the TensorFlow documentation to install TensorFlow 1\.x [with pip](https://www.tensorflow.org/install/pip) or [from source](https://www.tensorflow.org/install/source)\.

**Note**  
<a name="run-python3-commands"></a>If Python 2\.x and Python 3\.x are both installed on your device, use Python 3\.x in the commands that you run to install the dependencies\.

### Validating the TensorFlow installation<a name="ml-qualification-tensorflow-validate"></a>

Choose one of the following options to validate the TensorFlow installation\.

#### Option 1: SSH into your device and run a script<a name="ml-qualification-validate-tensorflow-option-1"></a>

1. <a name="ssh-validate-framework-install-ssh"></a>SSH into your device\.

1. Run the following script to verify that the dependency is correctly installed\.

   ```
   sudo python3.7 -c "import tensorflow; print(tensorflow.__version__)"
   ```

   <a name="ssh-passed-mldependencies"></a>The output prints the version number and the script should exit without error\.

#### Option 2: Run the IDT dependency test<a name="ml-qualification-validate-tensorflow-option-2"></a>

1. <a name="idt-validate-framework-install-check-config"></a>Make sure that `device.json` is configured for ML qualification\. For more information, see [Configure device\.json for ML qualification](set-config.md#device-json-ml-qualification)\.

1. <a name="idt-validate-framework-install-run-test"></a>Run the dependencies test for the framework\.

   ```
   devicetester_[linux | mac | win_x86-64] run-suite --group-id mldependencies --test-id tensorflow_dependency_check
   ```

   <a name="idt-passed-mldependencies"></a>The test summary displays a `PASSED` result for `mldependencies`\.

 

## Install Amazon SageMaker Neo Deep Learning Runtime \(DLR\) dependencies<a name="ml-qualification-dlr-dependencies"></a>

<a name="test-framework-dependencies"></a>IDT qualification tests for this framework have the following dependencies:
+ <a name="ml-qualification-python-req"></a>Python 3\.6 or Python 3\.7\.
**Note**  <a name="python-symlink-command"></a>
If you're using Python 3\.6, you must create a symbolic link from Python 3\.7 to Python 3\.6 binaries\. This configures your device to meet the Python requirement for AWS IoT Greengrass\. For example:  

  ```
  sudo ln -s path-to-python-3.6/python3.6 path-to-python-3.7/python3.7
  ```
+ SageMaker Neo DLR\.
+ numpy\.

After you install the DLR test dependencies, you must [compile the model](#ml-qualification-dlr-compile-model)\.

### Installing DLR<a name="ml-qualification-dlr-install"></a>

Follow the instructions in the DLR documentation to [install the Neo DLR](https://neo-ai-dlr.readthedocs.io/en/latest/install.html#building-on-linux)\.

**Note**  
<a name="run-python3-commands"></a>If Python 2\.x and Python 3\.x are both installed on your device, use Python 3\.x in the commands that you run to install the dependencies\.

### Validating the DLR installation<a name="ml-qualification-dlr-validate"></a>

Choose one of the following options to validate the DLR installation\.

#### Option 1: SSH into your device and run scripts<a name="ml-qualification-validate-dlr-option-1"></a>

1. <a name="ssh-validate-framework-install-ssh"></a>SSH into your device\.

1. <a name="ssh-validate-framework-install-run-scripts"></a>Run the following scripts to verify that the dependencies are correctly installed\.

   ```
   sudo python3.7 -c "import dlr; print(dlr.__version__)"
   ```

   ```
   sudo python3.7 -c "import numpy; print(numpy.__version__)"
   ```

   <a name="ssh-passed-mldependencies"></a>The output prints the version number and the script should exit without error\.

#### Option 2: Run the IDT dependency test<a name="ml-qualification-validate-dlr-option-2"></a>

1. <a name="idt-validate-framework-install-check-config"></a>Make sure that `device.json` is configured for ML qualification\. For more information, see [Configure device\.json for ML qualification](set-config.md#device-json-ml-qualification)\.

1. <a name="idt-validate-framework-install-run-test"></a>Run the dependencies test for the framework\.

   ```
   devicetester_[linux | mac | win_x86-64] run-suite --group-id mldependencies --test-id dlr_dependency_check
   ```

   <a name="idt-passed-mldependencies"></a>The test summary displays a `PASSED` result for `mldependencies`\.

## Compile the DLR model<a name="ml-qualification-dlr-compile-model"></a>

You must compile the DLR model before you can use it for ML qualification tests\. For steps, choose one of the following options\.

### Option 1: Use Amazon SageMaker to compile the model<a name="ml-qualification-compile-dlr-option-1"></a>

Follow these steps to use SageMaker to compile the ML model provided by IDT\. This model is pretrained with Apache MXNet\.

1. Verify that your device type is supported by SageMaker\. For more information, see the [target device options](https://docs.aws.amazon.com/sagemaker/latest/APIReference/API_OutputConfig.html#sagemaker-Type-OutputConfig-TargetDevice) the *Amazon SageMaker API Reference*\. If your device type is not currently supported by SageMaker, follow the steps in [Option 2: Use TVM to compile the DLR model](#ml-qualification-compile-dlr-option-2)\.
**Note**  
Running the DLR test with a model compiled by SageMaker might take 4 or 5 minutes\. Don’t stop IDT during this time\.

1. <a name="compile-dlr-download-uncompiled-model"></a>Download the tarball file that contains the uncompiled, pretrained MXNet model for DLR:
   + [dlr\-noncompiled\-model\-1\.0\.tar\.gz](https://d232ctwt5kahio.cloudfront.net/ml/dlr/noncompiled1.0/dlr-noncompiled-model-1.0.tar.gz)

1. <a name="compile-dlr-decompress-uncompiled-model"></a>Decompress the tarball\. This command generates the following directory structure\.  
![\[The resnet18 directory contains three files.\]](http://docs.aws.amazon.com/greengrass/latest/developerguide/images/idt/idt-ml-qualification-dlr-uncompiled.png)

1. Move `synset.txt` out of the `resnet18` directory\. Make a note of the new location\. You copy this file to compiled model directory later\.

1. Compress the contents of the `resnet18` directory\.

   ```
   tar cvfz model.tar.gz resnet18v1-symbol.json resnet18v1-0000.params
   ```

1. Upload the compressed file to an Amazon S3 bucket in your AWS account, and then follow the steps in [Compile a Model \(Console\)](https://docs.aws.amazon.com/sagemaker/latest/dg/neo-job-compilation-console.html) to create a compilation job\.

   1. For **Input configuration**, use the following values:
      + For **Data input configuration**, enter `{"data": [1, 3, 224, 224]}`\.
      + For **Machine learning framework**, choose `MXNet`\.

   1. For **Output configuration**, use the following values:
      + For **S3 Output location**, enter the path to the Amazon S3 bucket or folder where you want to store the compiled model\.
      + For **Target device**, choose your device type\.

1. Download the compiled model from the output location you specified, and then unzip the file\.

1. Copy `synset.txt` into the compiled model directory\.

1. Change the name of the compiled model directory to `resnet18`\.

   Your compiled model directory must have the following directory structure\.  
![\[The resnet18 compiled model directory contains four files.\]](http://docs.aws.amazon.com/greengrass/latest/developerguide/images/idt/idt-ml-qualification-dlr-compiled-sm.png)

### Option 2: Use TVM to compile the DLR model<a name="ml-qualification-compile-dlr-option-2"></a>

Follow these steps to use TVM to compile the ML model provided by IDT\. This model is pretrained with Apache MXNet, so you must install MXNet on the computer or device where you compile the model\. To install MXNet, follow the instructions in the [ MXNet documentation](https://mxnet.apache.org/get_started/?platform=linux&language=python&processor=cpu&environ=pip&)\.

**Note**  
We recommend that you compile the model on your target device\. This practice is optional, but it can help ensure compatibility and mitigate potential issues\.

 

1. <a name="compile-dlr-download-uncompiled-model"></a>Download the tarball file that contains the uncompiled, pretrained MXNet model for DLR:
   + [dlr\-noncompiled\-model\-1\.0\.tar\.gz](https://d232ctwt5kahio.cloudfront.net/ml/dlr/noncompiled1.0/dlr-noncompiled-model-1.0.tar.gz)

1. <a name="compile-dlr-decompress-uncompiled-model"></a>Decompress the tarball\. This command generates the following directory structure\.  
![\[The resnet18 directory contains three files.\]](http://docs.aws.amazon.com/greengrass/latest/developerguide/images/idt/idt-ml-qualification-dlr-uncompiled.png)

1. Follow the instructions in the TVM documentation to [build and install TVM from source for your platform](https://docs.tvm.ai/install/from_source.html)\.

1. After TVM is built, run the TVM compilation for the resnet18 model\. The following steps are based on [ Quick Start Tutorial for Compiling Deep Learning Models](https://tvm.apache.org/docs/tutorials/get_started/relay_quick_start.html#sphx-glr-tutorials-get-started-relay-quick-start-py) in the TVM documentation\.

   1. Open the `relay_quick_start.py` file from the cloned TVM repository\.

   1. Update the code that [defines a neural network in relay](https://tvm.apache.org/docs/tutorials/get_started/relay_quick_start.html#define-neural-network-in-relay)\. You can use one of following options:
      + Option 1: Use `mxnet.gluon.model_zoo.vision.get_model` to get the relay module and parameters:

        ```
        from mxnet.gluon.model_zoo.vision import get_model
        block = get_model('resnet18_v1', pretrained=True)
        mod, params = relay.frontend.from_mxnet(block, {"data": data_shape})
        ```
      + Option 2: From the uncompiled model that you downloaded in step 1, copy the following files to the same directory as the `relay_quick_start.py` file\. These files contain the relay module and parameters\.
        + `resnet18v1-symbol.json`
        + `resnet18v1-0000.params`

   1. Update the code that [saves and loads the compiled module](https://tvm.apache.org/docs/tutorials/get_started/relay_quick_start.html#save-and-load-compiled-module) to use the following code\.

      ```
      from tvm.contrib import util
      path_lib = "deploy_lib.so"
      #  Export the model library based on your device architecture
      lib.export_library("deploy_lib.so", cc="aarch64-linux-gnu-g++")
      with open("deploy_graph.json", "w") as fo:
          fo.write(graph)
      with open("deploy_param.params", "wb") as fo:
          fo.write(relay.save_param_dict(params))
      ```

   1. Build the model:

      ```
      python3 tutorials/relay_quick_start.py --build-dir ./model
      ```

      This command generates the following files\.
      + `deploy_graph.json`
      + `deploy_lib.so`
      + `deploy_param.params`

1. Copy the generated model files into a directory named `resnet18`\. This is your compiled model directory\.

1. Copy the compiled model directory to your host computer\. Then copy `synset.txt` from the uncompiled model that you downloaded in step 1 into the compiled model directory\.

   Your compiled model directory must have the following directory structure\.  
![\[The resnet18 compiled model directory contains four files.\]](http://docs.aws.amazon.com/greengrass/latest/developerguide/images/idt/idt-ml-qualification-dlr-compiled-tvm.png)

Next, [configure your AWS credentials and `device.json` file](set-config.md)\.