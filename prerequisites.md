# Release Machine workshop prerequisites

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

I recommend you use [VS Code](https://code.visualstudio.com/) and the [AWS Toolkit for VS Code](https://aws.amazon.com/visualstudiocode/), but you can use whatever IDE you feel comfortable with.

> The AWS Toolkit for VS Code includes full support for state machine visualization, enabling you to visualize your state machine in real time as you build. The AWS Toolkit for VS Code includes a language server for Amazon States Language, which lints your state machine definition to highlight common errors, provides auto-complete support, and code snippets for each state, enabling you to build state machines faster.

### Mac users - install Homebrew

Please [use Homebrew](https://docs.brew.sh/Installation) to install Node.js and Python.

When you install HomeBrew, you may see an error like this in the console

```
fatal: cannot copy '/Library/Developer/CommandLineTools/usr/share/git-core/templates/hooks/fsmonitor-watchman.sample' to '/usr/local/Homebrew/.git/hooks/fsmonitor-watchman.sample': Permission denied
Failed during: git init -q
```

You have to change this directory permision, so

```
sudo chown -R $USER /usr/local
```

Then run the Homebrew install script again.

### Install Node.js

You will need to have access to Node.js 10.3.0 or later. To see if you have it installed already:

```
$ node -v
v13.12.0
```

If you instead see some old version of node, run

```
brew link node
```

to link instead to the new version installed by Homebrew. Note you may have to use `--overwwrite` if your old version was installed using sudo.

#### Mac users

If you are installing Node.js for the first time, please [use Homebrew](https://docs.brew.sh/Installation).

When you install HomeBrew, if you see an error like this in the console

fatal: cannot copy '/Library/Developer/CommandLineTools/usr/share/git-core/templates/hooks/fsmonitor-watchman.sample' to '/usr/local/Homebrew/.git/hooks/fsmonitor-watchman.sample': Permission denied
Failed during: git init -q

You have to give this directory permision, so

```
sudo chown -R $USER /usr/local
```

Then run the Homebrew install script again.

If you're on Mac OS or Linux and need to manage multiple Node.js versions, nvm (Node Version Manager) is a good way to do that: https://nodejs.org/en/download/package-manager/#nvm

#### Windows users

Download the current stable MSI from [the Node.js downloads page:](https://nodejs.org/en/download/)

#### Linux users

Download the current stable Linux binary for your architecture from [the Node.js downloads page:](https://nodejs.org/en/download/)

### Install the AWS CLI

The AWS CLI is needed for both CDK and SAM to work.

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

You must set up credentials and an AWS Region to use AWS CDK. SAM needs to run as an IAM user with the AdministratorAccess policy, so make sure the user you configure here has those permissions.

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

Python AWS CDK applications require Python 3.6 or later.

Mac users should follow this link that discusses how to [use Homebrew to install Python3](https://installpython3.com/mac/). Another tool you may consider is [pyenv](https://github.com/pyenv/pyenv), which lets you manage multiple python versions.

Windows users can download a compatible version at [python.org](https://python.org).

If you run Linux, your system may have come with a compatible version, or you may install it using your distro's package manager (yum, apt, etc.).

The Python package installer, pip, and a virtual environment manager (venv or virtualenv), are also required. Python versions 3.3 and above for Mac and Windows include venv. On Linux, pip and virtualenv may be provided as separate packages in your package manager. Alternatively, you may install them with the following commands:

```
python3 -m ensurepip --upgrade
python3 -m pip install --upgrade pip
python3 -m pip install --upgrade virtualenv
```

If you encounter a permission error, run the above commands using sudo (to install the modules system-wide) or add the --user flag to each command so that the modules are installed in your user directory.

#### If you get stuck installing the CDK for python

The AWS documentation is [here](https://docs.aws.amazon.com/cdk/latest/guide/work-with-cdk-python.html).

#### Note

It is common for Linux distros to use the executable name python3 for Python 3.x, and have python refer to a Python 2.x installation, and similarly for pip/pip3. You can adjust the command used to run your application by editing cdk.json in the project's main directory.

Make sure the pip executable (on Windows, pip.exe) is in a directory included on the system PATH. You should be able to type `pip --version` and see its version, not an error message.

### Creating a project

You create a new AWS CDK project by invoking cdk init in an empty userspace directory.

```
cd ~
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

If you don't see `cdk-hello-world`, make sure you named your app's directory cdk-hello-world. If you didn't, go back to Create the app and try again.

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

If everything is set up correctly, CDK should deploy the sample infrastructure to the AWS account you configured in the earlier steps. Once you have checked this worked, you can use

```
cdk destroy
```

to remove the bucket you just created.

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

## Next Step

[Part One](part-one.md) - create the demo infrastructure
