# Release Manager Demo Part One

The first part of the demo requires us to set up some repos with CD pipelines that we can use in part two. This part uses the CDK and python (and a couple of shell scripts) to create the infrastructure.

This CDK project will do the following:

- Create one or more CodeCommit repositories, and push this code to each
- Create matching CodePipelines for each repository that will deploy this code

Each pipeline includes a build stage which runs CodeBuild to build the Lambda function and uses the CDK to build the Lambda/API Gateway CloudFormation stack to deploy. The CloudFormation deploy stage will then create

- The Lambda function
- An API Gateway endpoint in front of the Lambda function

The pipeline will be run whenever changes to the linked repository are committed.

## Prerequisites

You've already [installed the CDK for python](https://docs.aws.amazon.com/cdk/latest/guide/getting_started.html), right? Right.

## Getting Started

Create/change to whichever directory you generally like to clone repos eg

```
mkdir ~/workshop && cd ~/workshop
```

and clone the repo then cd into it

```
git clone https://github.com/guysqr/cdk-serverless-stack.git
cd cdk-serverless-stack
```

## Running the CDK project

This project is set up like a standard Python project. The initialization process also creates a virtualenv within this project, stored under the .env directory. To create the virtualenv it assumes that there is a `python3` (or `python` for Windows) executable in your path with access to the `venv` package. If for any reason the automatic creation of the virtualenv fails, you can create the virtualenv manually.

To manually create a virtualenv on MacOS and Linux:

```
python3 -m venv .env
```

After the init process completes and the virtualenv is created, you can use the following
step to activate your virtualenv.

```
source .env/bin/activate
```

If you are a Windows platform, you would activate the virtualenv like this:

```
.env\Scripts\activate.bat
```

Once the virtualenv is activated, you can install the required dependencies.

```
pip3 install -r requirements.txt
```

At this point you can now synthesize the CloudFormation template for this code.

```
cdk synth CdkServerlessStack
```

## Configuration

If you were able to synth the CdkServerlessStack then it's time to configure the project and create some AWS resources!

In the root of this directory you will see a file `demo-config.ini`. It contains the following items:

```
project=demoproject
name=demorepo
count=5
```

These are picked up and used in python and shell scripts to make getting set up easier. Note that we need these to be compatible with AWS naming rules so please stick to alphanumerics for the first two and an integer for the last, as per the defaults. Also, don't include spaces before or after the =.

`name` will be used to name your repositories and `count` will determine how many repos and pipelines you will create. If you're unsure you can just leave the default values, they should be fine.

Once you've done that, test the synth again

```
cdk synth CdkServerlessStack
```

And if everything works as expected, let's do the next step.

## Create the CodeCommit repositories

This project creates multiple CodePipeline pipelines, each of which will be connected to a CodeCommit repo, so the first step we need to take is to create the CodeCommit repos. To do that, run

```
cdk synth RepoStack
```

This will use `name` configured in the `demo-config.ini` file and make `count` repos for use by the CodePipeline. If you get no errors, you're good to go.

Because this template will be large, you will need to run `cdk bootstrap` to create a bucket for CDK to use to store the template on AWS.

```
cdk bootstrap
```

then

```
cdk deploy RepoStack
```

#### Set up Git Remote CodeCommit

Once you have the repos created, [go into the console](https://ap-southeast-2.console.aws.amazon.com/codesuite/codecommit/repositories?region=ap-southeast-2) and you will see a link to instructions on how to get connected to CodeCommit via HTTPS (GRC):

![alt text](https://github.com/guysqr/cdk-serverless-stack/raw/master/doc/repo-list.png 'Repo List View')

You will need to be using an IAM user who has CodeCommit permissions, and you will need to:

```
pip install git-remote-codecommit
```

#### Set the CodeCommit repos as remotes for this repo

We need the code in the current working tree to also be pushed to these new CodeCommit repos. To do that I created two helper scripts you can run that will add the new CodeCommit repos as remotes to this repo.

```
chmod 700 repo-add-remotes.sh
./repo-add-remotes.sh
```

or on Windows

```
bash -c ./repo-add-remotes.sh
```

> **Note**
> Ensure that all text files that you need to run via bash on Windows have been set to Unix line feeds. you can do this in Notepad. This includes the config file `demo-config.ini`.

#### Push the code from this repo to the CodeCommit remotes

This script will push the code in this repo to each of the CodeCommit repos.

```
chmod 700 repo-push.sh
./repo-push.sh
```

or on Windows

```
bash -c ./repo-push.sh
```

> Note this assumes you're connected via GRC, and using ap-southeast-2, so if you're not you will need to edit the script accordingly.

## Deploying the Pipeline stack

Once you have configured your CodeCommit repos you can run the CDK deploy command for the CodePipeline stack.

```
cdk deploy CdkServerlessStack
```

Assuming the stack creates correctly, you should now have `count` pipelines in CodePipeline, all running their source, build and deploy steps. Once completed you should end up with the following resources in your account

- `count` CodeCommit repositories called "`name`-1", from 1 to `count`
- `count` CodePipeline pipelines called "pipeline-for-`name`-1", from 1 to `count`
- `count` Lambda functions called "`project`-lambda-" followed by 8 alphanumeric random characters
- `count` API Gateway endpoints called "`project`-api-" followed by 8 alphanumeric random characters

![alt text](https://github.com/guysqr/cdk-serverless-stack/raw/master/doc/pipeline-view.png 'Pipeline View')

## Hacking this project

Feel free to play around with the stacks in this project to add additional infrastructure or change the configuration of what's defined here. To add additional dependencies, for example other CDK libraries for other AWS services, just add them to your `setup.py` file and rerun the `pip install -r requirements.txt` command.

### Useful commands

- `cdk ls` list all stacks in the app
- `cdk synth` emits the synthesized CloudFormation template
- `cdk deploy` deploy this stack to your default AWS account/region
- `cdk diff` compare deployed stack with current state
- `cdk docs` open CDK documentation

## Next Step

[Part Two](part-two.md) - create and test the Step Functions state machine
