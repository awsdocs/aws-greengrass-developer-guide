# Perform Machine Learning Inference<a name="ml-inference"></a>

This feature is available for AWS Greengrass Core v1\.5\.0 only\.

With AWS Greengrass, you can perform machine learning \(ML\) inference at the edge on locally generated data using cloud\-trained models\. This lets you benefit from the low latency and cost savings of running local inference, yet still take advantage of cloud computing power for training models and complex processing\.

To get started performing local inference, see [How to Configure Machine Learning Inference Using the AWS Management Console](ml-console.md)\.

## How AWS Greengrass ML Inference Works<a name="how-ml-inference-works"></a>

You can train your inference models anywhere, deploy them locally as *machine learning resources* in a Greengrass group, and then access them from Greengrass Lambda functions\. For example, you can build and train deep\-learning models in [Amazon SageMaker](https://console.aws.amazon.com/sagemaker) and deploy them to your Greengrass core\. Then, your Lambda functions can use the local models to perform inference on connected devices and send new training data back to the cloud\.

The following diagram shows the AWS Greengrass ML inference workflow\.

![\[Components of the machine learning workflow and the information flow between the core device, AWS Greengrass service, and cloud-trained models.\]](http://docs.aws.amazon.com/greengrass/latest/developerguide/images/ml-inference/diagram-ml-overview.png)

AWS Greengrass ML inference simplifies each step of the ML workflow, including:
+ Building and deploying ML framework prototypes\.
+ Accessing cloud\-trained models and deploying them to Greengrass core devices\.
+ Creating inference apps that can access hardware accelerators \(such as GPUs and FPGAs\) as [local resources](access-local-resources.md)\.

## Machine Learning Resources<a name="ml-resources"></a>

Machine learning resources represent cloud\-trained inference models that are deployed to an AWS Greengrass core\. To deploy machine learning resources, first you add the resources to a Greengrass group, and then you define how Lambda functions in the group can access them\. During group deployment, AWS Greengrass retrieves the source model packages from the cloud and extracts them to directories inside the Lambda runtime namespace\. Then, Greengrass Lambda functions use the locally deployed models to perform inference\.

To update a locally deployed model, first update the source model \(in the cloud\) that corresponds to the machine learning resource, and then deploy the group\. During deployment, AWS Greengrass checks the source for changes\. If changes are detected, then AWS Greengrass updates the local model\.

### Supported Model Sources<a name="supported-model-sources"></a>

AWS Greengrass supports Amazon SageMaker and Amazon S3 model sources for machine learning resources\.

The following requirements apply to model sources:
+ S3 buckets that store your Amazon SageMaker and Amazon S3 model sources must not be encrypted using SSE\-C\. For buckets that use server\-side encryption, AWS Greengrass ML inference currently supports only SSE\-S3 or SSE\-KMS encryption options\. For more information about server\-side encryption options, see [ Protecting Data Using Server\-Side Encryption](http://docs.aws.amazon.com/AmazonS3/latest/dev/serv-side-encryption.html) in the * Amazon Simple Storage Service Developer Guide*\.
+ The names of S3 buckets that store your Amazon SageMaker and Amazon S3 model sources must not include periods \("`.`"\)\. For more information, see the rule about using virtual hosted–style buckets with SSL in [ Rules for Bucket Naming](http://docs.aws.amazon.com/AmazonS3/latest/dev/BucketRestrictions.html#bucketnamingrules) in the * Amazon Simple Storage Service Developer Guide*\.
+ Service\-level region support must be available, as follows:
  + Amazon SageMaker model sources are supported only in regions that have both [ AWS Greengrass support](http://docs.aws.amazon.com/general/latest/gr/rande.html#greengrass_region) and [ Amazon SageMaker support](http://docs.aws.amazon.com/general/latest/gr/rande.html#sagemaker_region)\.
  + Amazon S3 model sources are supported only in regions that have both [ AWS Greengrass support](http://docs.aws.amazon.com/general/latest/gr/rande.html#greengrass_region) and [ Amazon S3 support](http://docs.aws.amazon.com/general/latest/gr/rande.html#s3_region)\.
+ AWS Greengrass must have `read` permission to the model source, as described in the following sections\.

**Amazon SageMaker**  
AWS Greengrass supports models that are saved as Amazon SageMaker training jobs\.  
If you configured your Amazon SageMaker environment by [creating a bucket](http://docs.aws.amazon.com/sagemaker/latest/dg/gs-config-permissions.html) whose name contains `sagemaker`, then AWS Greengrass has sufficient permission to access your Amazon SageMaker training jobs\. The AWSGreengrassResourceAccessRolePolicy managed policy allows access to buckets whose name contains the string `sagemaker`\. This policy is attached to the Greengrass service role\.  
Otherwise, you must grant AWS Greengrass `read` permission to the bucket where your training job is stored\. To do this, embed the following inline policy in the Greengrass service role\. You can list multiple bucket ARNs\.  

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
Amazon SageMaker is a fully managed ML service that enables you to build and train models using built\-in or custom algorithms\. For more information, see [What Is Amazon SageMaker](http://docs.aws.amazon.com/sagemaker/latest/dg/whatis.html) in the *Amazon SageMaker Developer Guide*\.

**Amazon S3**  
AWS Greengrass supports models that are stored in Amazon S3 as `tar.gz` or `.zip` files\.  
To enable AWS Greengrass to access models that are stored in Amazon S3 buckets, you must grant AWS Greengrass `read` permission to access the buckets by doing **one** of the following:  
+ Store your model in a bucket whose name contains `greengrass`\.

  The AWSGreengrassResourceAccessRolePolicy managed policy allows access to buckets whose name contains the string `greengrass`\. This policy is attached to the Greengrass service role\.

   
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

  For more information, see [Embedding Inline Policies](http://docs.aws.amazon.com/IAM/latest/UserGuide/access_policies_manage-attach-detach.html#embed-inline-policy-console) in the *IAM User Guide*\.

## Requirements<a name="ml-requirements"></a>

The following requirements apply for creating and using machine learning resources:
+ You must be using AWS Greengrass Core Software v1\.5\.0\.
+ Access to the local destination directory where the resource is stored must not require root privileges\.
+ Lambda functions can't perform privileged operations on the resource\. Only `read` or `read and write` permissions are available\.
+ You must provide the full path of the resource on the operating system of the core device\.
+ A resource name or ID has a maximum length of 128 characters and must use the pattern `[a-zA-Z0-9:_-]+`\.

## Precompiled Libraries for ML Frameworks<a name="precompiled-ml-libraries"></a>

To help you quickly get started experimenting with ML inference, AWS Greengrass provides precompiled libraries for the following ML frameworks\.
+ [Apache MXNet](https://aws.amazon.com/mxnet/) \(Apache License 2\.0\)
+ [ TensorFlow](https://aws.amazon.com/tensorflow/) \(Apache License 2\.0\)
+ Chainer \(MIT License\)

The precompiled MXNet and TensorFlow libraries can be installed on NVIDIA Jetson TX2, Intel Atom, and Raspberry Pi platforms\. The libraries are available from the **Software** page of the AWS IoT console\. You can install them directly on your core or include them as part of the software in your Greengrass group\.

Be sure to read the following information about compatibility and limitations\.

### MXNet Versioning<a name="mxnet-version-compatibility"></a>

Apache MXNet doesn't currently ensure forward compatibility, so models that you train using later versions of the framework might not work properly in earlier versions of the framework\. To avoid conflicts between the model\-training and model\-serving stages, and to provide a consistent end\-to\-end experience, use the same MXNet framework version in both stages\.

**Note**  
We recommend using MXNet v0\.11 for AWS Greengrass ML inference\. To configure Amazon SageMaker to train models using the recommended version, see [How to Configure Amazon SageMaker to Use MXNet v0\.11](#ml-sagemaker-v011)\.

### TensorFlow Model\-Serving Limitations on Raspberry Pi<a name="w3ab1c15c17c13"></a>

TensorFlow officially only supports installation on 64\-bit laptop or desktop operating systems\. Therefore, the precompiled TensorFlow libraries that AWS Greengrass provides for 32\-bit ARM platforms \(such as Raspberry Pi\) have inherent limitations and are intended for experimentation purposes only\.

The following recommendations for improving inference results are based on our tests with the 32\-bit ARM precompiled libraries on the Raspberry Pi platform\. These recommendations are intended for advanced users for reference only, without guarantees of any kind\.
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

## How to Configure Amazon SageMaker to Use MXNet v0\.11<a name="ml-sagemaker-v011"></a>

This section describes how to configure Amazon SageMaker to train models using MXNet v0\.11\. This is the recommended version for AWS Greengrass ML inference\.

1. Create an Amazon S3 bucket by following the [Create a Bucket](http://docs.aws.amazon.com/AmazonS3/latest/user-guide/create-bucket.html) procedure in the *Amazon Simple Storage Service Console User Guide*\.

   Make sure that you include `sagemaker` in the name \(for example, **sagemaker\-*datetime***\)\.

1. Create an Amazon SageMaker notebook instance by following the [Create an Amazon SageMaker Notebook Instance](http://docs.aws.amazon.com/sagemaker/latest/dg/gs-setup-working-env.html) procedure in the *Amazon SageMaker Developer Guide*\.

1. When the status of the notebook instance is **InService**, choose the **Open** action for your notebook instance\.

1. Choose **New**, and then choose **conda\_mxnet\_p27** from the list of Jupyter kernels\. This opens an MXNet environment for Python 2\.7\.

1. Override the default **sagemaker\.mxnet\.MXNet\.train\_image** field in the code with an MXNet v0\.11 container image, as follows:

   1. In the following code, replace the *container\-image* placeholder value with the MXNet v0\.11 container image that you want to use:
      + For MXNet v0\.11 on Python2\.7 with CPU, specify **sagemaker\-mxnet\-py2\-cpu**\.
      + For MXNet v0\.11 on Python2\.7 with GPU, specify **sagemaker\-mxnet\-py2\-gpu**\.
      + For MXNet v0\.11 on Python 3\.6 with CPU, specify **sagemaker\-mxnet\-py3\-cpu**\.
      + For MXNet v0\.11 on Python 3\.6 with GPU, specify **sagemaker\-mxnet\-py3\-gpu**\.

      ```
      region = sagemaker_session.boto_session.region_name
      mnist_estimator.train_image = lambda:'780728360657.dkr.ecr.{}.amazonaws.com/container-image:1.0'.format(region)
      ```

   1. Insert the code before the call to `sagemaker.mxnet.MXNet.fit`, which sends a `CreateTrainingJob` request to Amazon SageMaker using the MXNet v0\.11 container image\. This overrides the default v0\.12 image with the v0\.11 image\.

      The following shows a sample MXNet training code snippet with the new code highlighted\.  
![\[Jupyter notebook code screen with the code change highlighted.\]](http://docs.aws.amazon.com/greengrass/latest/developerguide/images/ml-inference/sm-mxnet-version-override.png)

Now you can continue training the model as described in the Amazon SageMaker documentation\.