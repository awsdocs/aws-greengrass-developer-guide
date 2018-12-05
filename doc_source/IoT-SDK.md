# Install the AWS IoT Device SDK for Python<a name="IoT-SDK"></a>

The AWS IoT Device SDK for Python can be used by all AWS IoT devices to communicate with the AWS IoT cloud and AWS IoT Greengrass core devices \(using the Python programming language\)\. The SDK requires Python 2, version 2\.7\+ or Python 3, version 3\.3\+\. The SDK requires OpenSSL version 1\.0\.1\+ \(TLS version 1\.2\) compiled with the Python executable\. 

To install the SDK onto your computer, with all required components, choose the appropriate tab:

------
#### [ Windows ]

1. Open an [elevated command prompt](https://technet.microsoft.com/en-us/library/cc947813(v=ws.10).aspx) and run the following command:

   ```
   python --version
   ```

   If no version information is returned or if the version number is less than 2\.7 for Python 2 or less than 3\.3 for Python 3, follow the instructions in [Downloading Python](https://wiki.python.org/moin/BeginnersGuide/Download) to install Python 2\.7\+ or Python 3\.3\+\. For more information, see [Using Python on Windows](https://docs.python.org/3.6/using/windows.html)\.

1. Using a web browser, [download](https://s3.amazonaws.com/aws-iot-device-sdk-python/aws-iot-device-sdk-python-latest.zip) the AWS IoT Device SDK for Python `zip` file and and save it as `aws-iot-device-sdk-python-latest.zip` \(this should be the default name\)\. The `zip` file is typically be saved to your `Downloads` folder\. Decompress `aws-iot-device-sdk-python-latest.zip` to an appropriate location, such as your home directory \(for example, `cd %HOME%`\)\. Make a note of the file path to the decompressed `aws-iot-device-sdk-python-latest` folder\. In the next step, this file path is indicated by *path\-to\-SDK\-folder*\.

1. From the elevated command prompt, run the following:

   ```
   cd path-to-SDK-folder
   python setup.py install
   ```

------
#### [ macOS ]

1. Open a Terminal window and run the following command:

   ```
   python --version
   ```

   If no version information is returned or if the version number is less that 2\.7 for Python 2 or less than 3\.3 for Python 3, follow the instructions in [Downloading Python](https://wiki.python.org/moin/BeginnersGuide/Download) to install Python 2\.7\+ or Python 3\.3\+ \. For more information, see [Using Python on a Macintosh](https://docs.python.org/3/using/mac.html)\.

1. In the Terminal window, run the following commands to determine the OpenSSL version:

   ```
   python
   >>> import ssl
   >>> print ssl.OPENSSL_VERSION
   ```

   Make a note of the OpenSSL version value\. 
**Note**  
If you're running Python 3, use print\(ssl\.OPENSSL\_VERSION\)\.

   To close the Python shell, run the following command:

   ```
   >>> exit()
   ```

   If the OpenSSL version is 1\.0\.1 or later, skip to step 3\. Otherwise, follow these steps:

   1. From the Terminal window, run the following command to determine if the computer is using Simple Python Version Management:

     ```
     which pyenv
     ```

   If a file path is returned, then choose the **Using `pyenv`** tab\. If nothing is returned, choose the **Not using `pyenv`** tab\.

------
#### [ Using pyenv ]

   1. See [Python Releases for Mac OS X](https://www.python.org/downloads/mac-osx/) \(or similar\) to determine the latest stable Python version\. In the following example, this value is indicated by *latest\-Python\-version*\.

   1. From the Terminal window, run the following commands:

      ```
      pyenv install latest-Python-version
      pyenv global latest-Python-version
      ```

      For example, if the latest version for Python 2 is 2\.7\.14, then these commands are:

      ```
      pyenv install 2.7.14
      pyenv global 2.7.14
      ```

   1. Close and then reopen the Terminal window and then run the following commands:

      ```
      python
      >>> import ssl
      >>> print ssl.OPENSSL_VERSION
      ```

      The OpenSSL version should be at least 1\.0\.1\. If the version is less than 1\.0\.1, then the update failed\. Check the Python version value used in the pyenv install and pyenv global commands and try again\.

   1. Run the following command to exit the Python shell:

      ```
      >>> exit()
      ```

------
#### [ Not using pyenv ]

   1. From a Terminal window, run the following command to determine if [brew](https://brew.sh/) is installed:

      ```
      which brew
      ```

      If a file path is not returned, install `brew` as follows:

      ```
      /usr/bin/ruby -e "$(curl -fsSL https://raw.githubusercontent.com/Homebrew/install/master/install)"
      ```
**Note**  
Follow the installation prompts\. The download for the Xcode command line tools can take some time\.

   1. Run the following commands:

      ```
      brew update
      brew install openssl
      brew install python@2
      ```

      The AWS IoT Device SDK for Python requires OpenSSL version 1\.0\.1 \(or later\) compiled with the Python executable\. The brew install python command installs a `python2` executable that meets this requirement\. The `python2` executable is installed in the `/usr/local/bin` directory, which should be part of the `PATH` environment variable\. To confirm, run the following command:

      ```
      python2 --version
      ```

      If `python2` version information is provided, skip to the next step\. Otherwise, permanently add the `/usr/local/bin` path to your `PATH` environment variable by appending the following line to your shell profile:

      ```
      export PATH="/usr/local/bin:$PATH"
      ```

      For example, if you're using `.bash_profile` or do not yet have a shell profile, run the following command from a Terminal window:

      ```
      echo 'export PATH="/usr/local/bin:$PATH"' >> ~/.bash_profile
      ```

      Next, [source](https://en.wikipedia.org/wiki/Source_(command)) your shell profile and confirm that `python2 --version` provides version information\. For example, if you're using `.bash_profile`, run the following commands:

      ```
      source ~/.bash_profile
      python2 --version
      ```

      `python2` version information should be returned\.

   1. Append the following line to your shell profile:

      ```
      alias python="python2"
      ```

      For example, if you're using `.bash_profile` or do not yet have a shell profile, run the following command:

      ```
      echo 'alias python="python2"' >> ~/.bash_profile
      ```

   1. Next, [source](https://en.wikipedia.org/wiki/Source_(command)) your shell profile\. For example, if you're using `.bash_profile`, run the following command:

      ```
      source ~/.bash_profile
      ```

      Invoking the python command runs the Python executable that contains the required OpenSSL version \(`python2`\) \.

   1. Run the following commands:

      ```
      python
      >>> import ssl
      >>> print ssl.OPENSSL_VERSION
      ```

      The OpenSSL version should be 1\.0\.1 or later\.

   1. To exit the Python shell, run the following command:

      ```
      >>> exit()
      ```

------

1. Run the following commands to install the AWS IoT Device SDK for Python:

   ```
   cd ~
   git clone https://github.com/aws/aws-iot-device-sdk-python.git
   cd aws-iot-device-sdk-python
   python setup.py install
   ```

------
#### [ UNIX\-like system ]

1. From a terminal window, run the following command:

   ```
   python --version
   ```

   If no version information is returned or if the version number is less than 2\.7 for Python 2 or less than 3\.3 for Python 3, follow the instructions in [Downloading Python](https://wiki.python.org/moin/BeginnersGuide/Download) to install Python 2\.7\+ or Python 3\.3\+\. For more information, see [Using Python on Unix platforms](https://docs.python.org/3.6/using/unix.html)\.

1. In the terminal, run the following commands to determine the OpenSSL version:

   ```
   python
   >>> import ssl
   >>> print ssl.OPENSSL_VERSION
   ```

   Make a note of the OpenSSL version value\. 

   To close the Python shell, run the following command:

   ```
   >>> exit()
   ```

   If the OpenSSL version is 1\.0\.1 or later, skip to the next step\. Otherwise, run the command\(s\) to update OpenSSL for your distribution \(for example, `sudo yum update openssl`, `sudo apt-get update`, and so on\)\.

   Confirm that the OpenSSL version is 1\.0\.1 or later by running the following commands:

   ```
   python
   >>> import ssl
   >>> print ssl.OPENSSL_VERSION
   >>> exit()
   ```

1. Run the following commands to install the AWS IoT Device SDK for Python:

   ```
   cd ~
   git clone https://github.com/aws/aws-iot-device-sdk-python.git
   cd aws-iot-device-sdk-python
   sudo python setup.py install
   ```

------

After the AWS IoT Device SDK for Python is installed, navigate to the `samples` folder, open the `greengrass` folder, and then copy the `basicDiscovery.py` file to the folder that contains the HelloWorld\_Publisher and HelloWorld\_Subscriber device certificates files, as shown in the following example\. \(The [GUID](https://en.wikipedia.org/wiki/Universally_unique_identifier)\-like filename components are different\.\)

![\[Screenshot of files with basicDiscovery.py indicated.\]](http://docs.aws.amazon.com/greengrass/latest/developerguide/images/gg-get-started-075.png)