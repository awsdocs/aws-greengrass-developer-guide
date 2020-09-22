# Perform machine learning inference<a name="ml-inference"></a>

This feature is available for AWS IoT Greengrass Core v1\.6 or later\.

With AWS IoT Greengrass, you can perform machine learning \(ML\) inference at the edge on locally generated data using cloud\-trained models\. You benefit from the low latency and cost savings of running local inference, yet still take advantage of cloud computing power for training models and complex processing\.

To get started performing local inference, see [How to configure machine learning inference using the AWS Management Console](ml-console.md)\.

## How AWS IoT Greengrass ML inference works<a name="how-ml-inference-works"></a>

You can train your inference models anywhere, deploy them locally as *machine learning resources* in a Greengrass group, and then access them from Greengrass Lambda functions\. For example, you can build and train deep\-learning models in [SageMaker](https://console.aws.amazon.com/sagemaker) and deploy them to your Greengrass core\. Then, your Lambda functions can use the local models to perform inference on connected devices and send new training data back to the cloud\.

The following diagram shows the AWS IoT Greengrass ML inference workflow\.

![\[Components of the machine learning workflow and the information flow between the core device, AWS IoT Greengrass service, and cloud-trained models.\]](http://docs.aws.amazon.com/greengrass/latest/developerguide/images/ml-inference/diagram-ml-overview.png)

AWS IoT Greengrass ML inference simplifies each step of the ML workflow, including:
+ Building and deploying ML framework prototypes\.
+ Accessing cloud\-trained models and deploying them to Greengrass core devices\.
+ Creating inference apps that can access hardware accelerators \(such as GPUs and FPGAs\) as [local resources](access-local-resources.md)\.

## Machine learning resources<a name="ml-resources"></a>

Machine learning resources represent cloud\-trained inference models that are deployed to an AWS IoT Greengrass core\. To deploy machine learning resources, first you add the resources to a Greengrass group, and then you define how Lambda functions in the group can access them\. During group deployment, AWS IoT Greengrass retrieves the source model packages from the cloud and extracts them to directories inside the Lambda runtime namespace\. Then, Greengrass Lambda functions use the locally deployed models to perform inference\.

To update a locally deployed model, first update the source model \(in the cloud\) that corresponds to the machine learning resource, and then deploy the group\. During deployment, AWS IoT Greengrass checks the source for changes\. If changes are detected, then AWS IoT Greengrass updates the local model\.

### Supported model sources<a name="supported-model-sources"></a>

AWS IoT Greengrass supports SageMaker and Amazon S3 model sources for machine learning resources\.

The following requirements apply to model sources:
+ S3 buckets that store your SageMaker and Amazon S3 model sources must not be encrypted using SSE\-C\. For buckets that use server\-side encryption, AWS IoT Greengrass ML inference currently supports the SSE\-S3 or SSE\-KMS encryption options only\. For more information about server\-side encryption options, see [Protecting data using server\-side encryption](https://docs.aws.amazon.com/AmazonS3/latest/dev/serv-side-encryption.html) in the *Amazon Simple Storage Service Developer Guide*\.
+ The names of S3 buckets that store your SageMaker and Amazon S3 model sources must not include periods \(`.`\)\. For more information, see the rule about using virtual hosted\-style buckets with SSL in [Rules for bucket naming](https://docs.aws.amazon.com/AmazonS3/latest/dev/BucketRestrictions.html#bucketnamingrules) in the *Amazon Simple Storage Service Developer Guide*\.
+ Service\-level AWS Region support must be available for both [AWS IoT Greengrass](https://docs.aws.amazon.com/general/latest/gr/greengrass.html) and [SageMaker](https://docs.aws.amazon.com/general/latest/gr/sagemaker.html)\. Currently, AWS IoT Greengrass supports SageMaker models in the following Regions:
  + US East \(Ohio\)
  + US East \(N\. Virginia\)
  + US West \(Oregon\)
  + Asia Pacific \(Mumbai\)
  + Asia Pacific \(Seoul\)
  + Asia Pacific \(Singapore\)
  + Asia Pacific \(Sydney\)
  + Asia Pacific \(Tokyo\)
  + Europe \(Frankfurt\)
  + Europe \(Ireland\)
  + Europe \(London\)
+ AWS IoT Greengrass must have `read` permission to the model source, as described in the following sections\.

**SageMaker**  
AWS IoT Greengrass supports models that are saved as SageMaker training jobs\. SageMaker is a fully managed ML service that you can use to build and train models using built\-in or custom algorithms\. For more information, see [What is SageMaker?](https://docs.aws.amazon.com/sagemaker/latest/dg/whatis.html) in the *SageMaker Developer Guide*\.  
If you configured your SageMaker environment by [creating a bucket](https://docs.aws.amazon.com/sagemaker/latest/dg/gs-config-permissions.html) whose name contains `sagemaker`, then AWS IoT Greengrass has sufficient permission to access your SageMaker training jobs\. The `AWSGreengrassResourceAccessRolePolicy` managed policy allows access to buckets whose name contains the string `sagemaker`\. This policy is attached to the [Greengrass service role](service-role.md)\.  
Otherwise, you must grant AWS IoT Greengrass `read` permission to the bucket where your training job is stored\. To do this, embed the following inline policy in the service role\. You can list multiple bucket ARNs\.  

```
{
    "Version": "2012-10-17",
    "Statement": [
        {
            "Effect": "Allow",
            "Action": [
                "s3:GetObject"
            ],
            "Resource": [
                "arn:aws:s3:::my-bucket-name"
            ]
        }
    ]
}
```

**Amazon S3**  
AWS IoT Greengrass supports models that are stored in Amazon S3 as `tar.gz` or `.zip` files\.  
To enable AWS IoT Greengrass to access models that are stored in Amazon S3 buckets, you must grant AWS IoT Greengrass `read` permission to access the buckets by doing **one** of the following:  
+ Store your model in a bucket whose name contains `greengrass`\.

  The `AWSGreengrassResourceAccessRolePolicy` managed policy allows access to buckets whose name contains the string `greengrass`\. This policy is attached to the [Greengrass service role](service-role.md)\.

   
+ Embed an inline policy in the Greengrass service role\.

  If your bucket name doesn't contain `greengrass`, add the following inline policy to the service role\. You can list multiple bucket ARNs\.

  ```
  {
      "Version": "2012-10-17",
      "Statement": [
          {
              "Effect": "Allow",
              "Action": [
                  "s3:GetObject"
              ],
              "Resource": [
                  "arn:aws:s3:::my-bucket-name"
              ]
          }
      ]
  }
  ```

  For more information, see [Embedding inline policies](https://docs.aws.amazon.com/IAM/latest/UserGuide/access_policies_manage-attach-detach.html#embed-inline-policy-console) in the *IAM User Guide*\.

## Requirements<a name="ml-requirements"></a>

The following requirements apply for creating and using machine learning resources:
+ You must be using AWS IoT Greengrass Core v1\.6 or later\.
+ User\-defined Lambda functions can perform `read` or `read and write` operations on the resource\. Permissions for other operations are not available\.  The containerization mode of affiliated Lambda functions determines how you set access permissions\. For more information, see [Access machine learning resources from Lambda functions](access-ml-resources.md)\.
+ You must provide the full path of the resource on the operating system of the core device\.
+ A resource name or ID has a maximum length of 128 characters and must use the pattern `[a-zA-Z0-9:_-]+`\.

## Runtimes and libraries for ML inference<a name="ml-libraries"></a>

You can use the following ML runtimes and libraries with AWS IoT Greengrass\.
+  [Amazon SageMaker Neo deep learning runtime](#dlc-optimize-info) 
+ Apache MXNet
+ TensorFlow

These runtimes and libraries can be installed on NVIDIA Jetson TX2, Intel Atom, and Raspberry Pi platforms\. For download information, see [Supported machine learning runtimes and libraries](what-is-gg.md#ml-runtimes-libs)\. You can install them directly on your core device\.

Be sure to read the following information about compatibility and limitations\.

### SageMaker Neo deep learning runtime<a name="dlc-optimize-info"></a>

 You can use the SageMaker Neo deep learning runtime to perform inference with optimized machine learning models on your AWS IoT Greengrass devices\. These models are optimized using the SageMaker Neo deep learning compiler to improve machine learning inference prediction speeds\. For more information about model optimization in SageMaker, see the [SageMaker Neo documentation](https://docs.aws.amazon.com/sagemaker/latest/dg/neo.html)\. 

**Note**  
 Currently, you can optimize machine learning models using the Neo deep learning compiler in specific AWS Regions only\. However, you can use the Neo deep learning runtime with optimized models in all AWS Regions where AWS IoT Greengrass core is supported\. For information, see [How to Configure Optimized Machine Learning Inference](ml-dlc-console.md)\. 

### MXNet versioning<a name="mxnet-version-compatibility"></a>

Apache MXNet doesn't currently ensure forward compatibility, so models that you train using later versions of the framework might not work properly in earlier versions of the framework\. To avoid conflicts between the model\-training and model\-serving stages, and to provide a consistent end\-to\-end experience, use the same MXNet framework version in both stages\.

### MXNet on Raspberry Pi<a name="mxnet-engine-rpi"></a>

Greengrass Lambda functions that access local MXNet models must set the following environment variable:

```
MXNET_ENGINE_TYPE=NaiveEngine
```

You can set the environment variable in the function code or add it to the function's group\-specific configuration\. For an example that adds it as a configuration setting, see this [step](ml-console.md#ml-console-config-lambda)\.

**Note**  
For general use of the MXNet framework, such as running a third\-party code example, the environment variable must be configured on the Raspberry Pi\.

### TensorFlow model\-serving limitations on Raspberry Pi<a name="w64aac22c15c17"></a>

The following recommendations for improving inference results are based on our tests with the TensorFlow 32\-bit Arm libraries on the Raspberry Pi platform\. These recommendations are intended for advanced users for reference only, without guarantees of any kind\.
+ Models that are trained using the [Checkpoint](https://www.tensorflow.org/get_started/checkpoints) format should be "frozen" to the protocol buffer format before serving\. For an example, see the [TensorFlow\-Slim image classification model library](https://github.com/tensorflow/models/tree/master/research/slim)\.
+ Don't use the TF\-Estimator and TF\-Slim libraries in either training or inference code\. Instead, use the `.pb` file model\-loading pattern that's shown in the following example\.

  ```
  graph = tf.Graph() 
  graph_def = tf.GraphDef()
  graph_def.ParseFromString(pb_file.read()) 
  with graph.as_default():
    tf.import_graph_def(graph_def)
  ```

**Note**  
For more information about supported platforms for TensorFlow, see [Installing TensorFlow](https://www.tensorflow.org/install/#installing_from_sources) in the TensorFlow documentation\.