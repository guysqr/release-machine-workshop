# Release Manager workshop pre-requisites

This workshop uses two Git repos to set up the different infrastructure used in the demo. This guide will walk you through the steps to get set up and use them. Depending on your machine, OS and how many of these you already have installed, this might take anywhere from 10 minutes to a couple of hours to complete.

We're going to

1. Install Node.js (need v10.3.x or later)
2. Install the AWS CLI
3. Configure your AWS IAM credentials
4. Install the CDK
5. Install Python 3
6. Install the SAM CLI

Of course, you should skip any steps for things you already have running on your machine.

## Get set up

We'll be using the AWS CDK for Python and AWS SAM (Serverless Application Model) CLI to build the resources for this demo.

I recommend you use [VS Code](https://code.visualstudio.com/) and the [AWS Toolkit for VS Code](https://aws.amazon.com/visualstudiocode/) because it has some nice features for working with Step Functions, but you can use whatever IDE you feel comfortable with.

### Install Node.js

All CDK developers need to install Node.js 10.3.0 or later. It's available for all platforms here: https://nodejs.org/en/download/

#### Mac users

If you're on Mac OS or Linux, nvm (Node Version Manager) is a good way to install and manage Node.js: https://nodejs.org/en/download/package-manager/#nvm

If that seems too much like hard work, https://nodejs.org/dist/v12.18.1/node-v12.18.1.pkg

#### Windows users

https://nodejs.org/dist/v12.18.1/node-v12.18.1-x86.msi

#### Linux users

https://nodejs.org/dist/v12.18.1/node-v12.18.1-linux-x64.tar.xz

### Install the AWS CLI

#### Mac users

Download and run the macOS pkg file: https://awscli.amazonaws.com/AWSCLIV2.pkg.

To verify:

```
$ which aws
/usr/local/bin/aws
$ aws --version
aws-cli/2.0.23 Python/3.7.4 Darwin/18.7.0 botocore/2.0.0
```

#### Windows users

Download and run the AWS CLI MSI installer for Windows (64-bit) at https://awscli.amazonaws.com/AWSCLIV2.msi.

To verify:

```
C:\> aws --version
aws-cli/2.0.23 Python/3.7.4 Windows/10 botocore/2.0.0
```

#### Linux users

```
curl "https://awscli.amazonaws.com/awscli-exe-linux-x86_64.zip" -o "awscliv2.zip"
unzip awscliv2.zip
sudo ./aws/install
```

To verify:

```
$ aws --version
aws-cli/2.0.23 Python/3.7.4 Darwin/18.7.0 botocore/2.0.0
```

https://docs.aws.amazon.com/cli/latest/userguide/install-cliv2.html

### Configure your AWS account credentials

You must provide your credentials and an AWS Region to use AWS CDK, if you have not already done so. SAM needs to run as an IAM user with the AdministratorAccess policy, so make sure the user you configure here has those permissions.

#### Important

We strongly recommend against using your AWS root account for day-to-day tasks. Instead, create a user in IAM and use its credentials with the CDK.

If you have the AWS CLI installed, the easiest way to satisfy this requirement is to use the AWS CLI and issue the following command:

```
aws configure
```

Provide your AWS access key ID, secret access key, and default region when prompted.

You may also manually create or edit the `~/.aws/config` and `~/.aws/credentials` (Linux or Mac) or `%USERPROFILE%\.aws\config and %USERPROFILE%\.aws\credentials` (Windows) files to contain credentials and a default region, in the following format.

In `~/.aws/config or %USERPROFILE%\.aws\config`

```
[default]
region=us-west-2
```

In `~/.aws/credentials or %USERPROFILE%\.aws\credentials`

```
[default]
aws_access_key_id=AKIAI44QH8DHBEXAMPLE
aws_secret_access_key=je7MtGbClwBF/2Zp9Utk/h3yCo8nvbEXAMPLEKEY
```

If you have multiple AWS account profiles, then you need to export the AWS_PROFILE environment variable to tell the CLI tools which profile to use.

### Install the CDK

To install the CDK as a global package use npm (the Node Package Manager).

```
npm install -g aws-cdk
```

Run the following command to verify correct installation and print the version number of the AWS CDK.

```
cdk --version
```

Further instructions [here](https://docs.aws.amazon.com/cdk/latest/guide/getting_started.html), should you need them.

### Install Python

Python AWS CDK applications require Python 3.6 or later. If you don't already have it installed, download a compatible version for your platform at [python.org](python.org). If you run Linux, your system may have come with a compatible version, or you may install it using your distro's package manager (yum, apt, etc.). Mac users may be interested in Homebrew, a Linux-style package manager for Mac OS X.

The Python package installer, pip, and virtual environment manager, virtualenv, are also required. Windows installations of compatible Python versions include these tools. On Linux, pip and virtualenv may be provided as separate packages in your package manager. Alternatively, you may install them with the following commands:

```
python -m ensurepip --upgrade
python -m pip install --upgrade pip
python -m pip install --upgrade virtualenv
```

If you encounter a permission error, run the above commands using sudo (to install the modules system-wide) or add the --user flag to each command so that the modules are installed in your user directory.

#### Note

It is common for Linux distros to use the executable name python3 for Python 3.x, and have python refer to a Python 2.x installation, and similarly for pip/pip3. You can adjust the command used to run your application by editing cdk.json in the project's main directory.

Make sure the pip executable (on Windows, pip.exe) is in a directory included on the system PATH. You should be able to type `pip --version` and see its version, not an error message.

#### Creating a project

You create a new AWS CDK project by invoking cdk init in an empty directory.

```
mkdir cdk-hello-world
cd cdk-hello-world
cdk init app --language python
```

`cdk init` uses the name of the project folder to name various elements of the project, including classes, subfolders, and files.

After initializing the project, activate the project's virtual environment. This allows the project's dependencies to be installed locally in the project folder, instead of globally.

```
source .env/bin/activate
```

> **Important**
>
> Activate the project's virtual environment whenever you start working on it. If you don't, you won't have access to the modules installed there, and modules you install will go in Python's global module directory (or will result in a permission error).

Then install the app's standard dependencies:

```
pip install -r requirements.txt
```

Just to verify everything is working correctly, list the stacks in your app.

```
cdk ls
```

If you don't see CdkHelloWorld, make sure you named your app's directory cdk-hello-world. If you didn't, go back to Create the app and try again.

If that worked, let's make a really simple demo. Start by installing the S3 package from the AWS Construct Library:

```
pip install aws-cdk.aws-s3
```

Replace the first import statement in hello_cdk_stack.py in the hello_cdk directory with the following code.

```
from aws_cdk import (
    aws_s3 as s3,
    core
)
```

Replace the comment with the following code.

```
bucket = s3.Bucket(self,
    "MyFirstBucket",
    versioned=True,)
```

Once you have done that you should be able to

```
cdk synth
```

and the CDK will produce a CloudFormation template as output. If you get an error instead, [consult the documentation](https://docs.aws.amazon.com/cdk/latest/guide/work-with-cdk-python.html). You should see something like:

```
Resources:
  MyFirstBucketB8884501:
    Type: AWS::S3::Bucket
    Properties:
      VersioningConfiguration:
        Status: Enabled
    UpdateReplacePolicy: Retain
    DeletionPolicy: Retain
    Metadata:
      aws:cdk:path: HelloCdkStack/MyFirstBucket/Resource
  CDKMetadata:
    Type: AWS::CDK::Metadata
    Properties:
      Modules: aws-cdk=1.XX.X,@aws-cdk/aws-events=1.XX.X,@aws-cdk/aws-iam=1.XX.X,@aws-cdk/aws-kms=1.XX.X,@aws-cdk/aws-s3=1.XX.X,@aws-cdk/cdk-assets-schema=1.XX.X,@aws-cdk/cloud-assembly-schema=1.XX.X,@aws-cdk/core=1.XX.X,@aws-cdk/cx-api=1.XX.X,@aws-cdk/region-info=1.XX.X,jsii-runtime=node.js/vXX.XX.X
```

If that worked, try

```
cdk deploy
```

If everything is set up correctly, CDK should deploy the sample infrastructure to the AWS account you configured in the earlier steps.

#### If you get stuck installing the CDK

The AWS documentation is [here](https://docs.aws.amazon.com/cdk/latest/guide/work-with-cdk-python.html).

### Install SAM

#### Mac/Linux users

If you need to install Homebrew, [do that first](https://docs.brew.sh/Installation).

Once you have Homebrew installed, follow these steps to install the AWS SAM CLI:

```
brew tap aws/tap
brew install aws-sam-cli
```

##### Verify the installation

```
sam --version
```

You should see output like the following after successful installation of the AWS SAM CLI:

```
SAM CLI, version 0.52.0
```

##### If you already had SAM CLI installed, check for updates

```
brew upgrade aws-sam-cli
```

#### Windows users

Install the [AWS SAM CLI 64-bit via the MSI](https://github.com/awslabs/aws-sam-cli/releases/latest/download/AWS_SAM_CLI_64_PY3.msi).

##### Verify the installation.

After completing the installation, verify it by opening a new command prompt or PowerShell prompt. You should be able to invoke sam from the command line.

```
sam --version
```

You should see output like the following after successful installation of the AWS SAM CLI:

```
SAM CLI, version 0.52.0
```

You're now ready to start development.

Please look at the [AWS documentation relevant to your OS](https://docs.aws.amazon.com/serverless-application-model/latest/developerguide/serverless-sam-cli-install.html).

### Enable bash on Windows

There is a bash script in the next section, so if you're on Windows, make sure you can run bash scripts. [Here's a guide](https://www.howtogeek.com/249966/how-to-install-and-use-the-linux-bash-shell-on-windows-10/) on how to do that.
