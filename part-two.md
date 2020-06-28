# Release Manager Demo Part Two

The second part of the demo is the fun part. Here we're going to use AWS SAM to set up a Step Functions state machine to run our pipelines.

This SAM project will do the following:

- Create the Step Functions state machine
- Create the Lambda functions, S3 bucket & API gateway endpoint used by the state machine
- Create and configure the additional services, roles and other stuff required by the State Machine

## Prerequisites

You've already [installed the SAM CLI](https://docs.aws.amazon.com/serverless-application-model/latest/developerguide/serverless-sam-cli-install.html), right? 

## Getting Started

Clone the repo and cd into it.

```
git clone https://github.com/guysqr/release-machine
cd release-machine
```

Then you will need to run 

```
sam build
```

If that worked correctly it will prompt you to do a guided deployment. You should use guided mode the first time to set various parameters, but after that you can drop the `--guided` bit off. 

I suggest you use the same defaults I have set here.

```
sam deploy --guided

Configuring SAM deploy
======================

        Looking for samconfig.toml :  Found
        Reading default arguments  :  Success

        Setting default arguments for 'sam deploy'
        =========================================
        Stack Name []: release-manager
        AWS Region []: ap-southeast-2
        #Shows you resources changes to be deployed and require a 'Y' to initiate deploy
        Confirm changes before deploy [Y/n]: Y
        #SAM needs permission to be able to create roles to connect to the resources in your template
        Allow SAM CLI IAM role creation [Y/n]: Y
        Save arguments to samconfig.toml [Y/n]: Y

```

Once you hit enter you should see

```

        Looking for resources needed for deployment: Found!

                Managed S3 bucket: aws-sam-cli-managed-default-samclisourcebucket-178onu6euafev
                A different default S3 bucket can be set in samconfig.toml

        Saved arguments to config file
        Running 'sam deploy' for future deployments will use the parameters saved above.
        The above parameters can be changed by modifying samconfig.toml
        Learn more about samconfig.toml syntax at 
        https://docs.aws.amazon.com/serverless-application-model/latest/developerguide/serverless-sam-cli-config.html

        Deploying with following values
        ===============================
        Stack name                 : release-manager
        Region                     : ap-southeast-2
        Confirm changeset          : True
        Deployment s3 bucket       : aws-sam-cli-managed-default-samclisourcebucket-178onu6euafev
        Capabilities               : ["CAPABILITY_IAM"]
        Parameter overrides        : {}

Initiating deployment
=====================

Waiting for changeset to be created..    
```

If a changeset is created you will then get prompted to deploy it. Enter `y` to begin deployment.

## About the Release Machine

The Release Machine is a software release workflow for deploying from multiple pipelines simultaneously, triggered by a `release-manifest.json` file dropped in an S3 bucket or posted to an API Gateway endpoint. 

![alt text](https://github.com/guysqr/release-machine/raw/master/stepfunctions_graph.svg?sanitize=true 'Successful Execution')

The release process contains steps that

1. Verify the manifest is valid and has not already been run
2. If valid, records the release request and timestamp
3. Triggers an execution for each of the requested pipelines
4. Checks each pipeline execution's state every 30 seconds and records any changes
5. Records the release as complete if all pipelines complete successfully
6. Fails the release if any fail to complete successfully

> Note that this version does not attempt any rollbacks or pipeline execution cancellations when something fails, but these could definitely be added as steps, depending on how the deployment is done. Using CodeDeploy, for instance, stores previous releases to which Release Manager could roll back.

### Triggering the Release Machine

The Release Machine has been set up to be triggered by either writing the `release-manifest.json` file to the release manager S3 bucket, or by posting the `release-manifest.json` file's contents to the release manager API Gateway endpoint. 

To see the magic happen via the first method, go to [the S3 console](https://s3.console.aws.amazon.com/s3) and upload the `release-manifest.json` file from this directory to the release-manager S3 bucket that SAM has created.

To see the magic happen via the second method, go to [the API Gateway console](https://ap-southeast-2.console.aws.amazon.com/apigateway/main/apis?region=ap-southeast-2) and click on the API called `release-manager`. Click on the word POST in the Resources list, then click TEST.

![API Gateway test](img/api-gateway.png 'API Gateway test')

Copy the contents of the `release-manifest.json` file from this directory to your clipboard and paste it into the Request Body box then hit "Test". You should see a response body pop up that looks similar to this: 

```
{
  "executionArn": "arn:aws:states:ap-southeast-2:76332347855:execution:ReleaseStateMachine-l0NkeqrA8Z0X:bdec481c-5fac-44e9-9a26-ea234a438063",
  "startDate": 1593322928.575
}
```

### Configuring a release

Note you will need to change the releaseId value in the `release-manifest.json` file to trigger a second and subsequent deployment. This is also how you change which pipelines you want to trigger.

The idea is that this file is generated as part of a process that determines which of your application components needs to be deployed as part of that release. So, you might have a release that needs the component built by pipelines 1 and 3, and next time you do a release it requires pipelines 2, 4 and 5. These would result in manifest files that look like:

```json
{
    "releaseId": 5.15,
    "pipelines": [
        "pipeline-for-stepfunctions-pipeline-repo-1",
        "pipeline-for-stepfunctions-pipeline-repo-3"
    ]
}
```

and 

```json
{
    "releaseId": 5.16,
    "pipelines": [
        "pipeline-for-stepfunctions-pipeline-repo-2",
        "pipeline-for-stepfunctions-pipeline-repo-4",
        "pipeline-for-stepfunctions-pipeline-repo-5"
    ]
}
```

### Reviewing progress

Go to [the Step Functions console](https://ap-southeast-2.console.aws.amazon.com/states/home) and click on the state machine that begins with `ReleaseStateMachine-`. In the 'Executions' list you should see an execution In Progress. If you click on the current execution you will see the visual workflow showing each state in the execution and any errors that are occurring.

Go to [the CodePipeline pipelines list](https://ap-southeast-2.console.aws.amazon.com/codesuite/codepipeline/pipelines) to see which pipelines are being executed.

Go to [the DynamoDB console](https://ap-southeast-2.console.aws.amazon.com/dynamodb/home?region=ap-southeast-2) to see what is being recorded in the Releases and Executions tables.

#### How to log data events in CloudTrail

Navigate to the Trails page of the CloudTrail console and choose Create trail.

>**Note**
>While you can edit an existing trail to add logging data events, as a best practice, consider creating a separate trail specifically for logging data events.

For Data events, choose the pencil icon to enable editing then click on the S3 tab. Choose Add S3 bucket. Type the name of the release manager bucket created by SAM and specify Write events.

You can edit the bucket name, prefix, Read/Write option, or remove the resource by choosing the x icon.

Choose Save.

Release Machine can be triggered by an API Gateway endpoint as well. To use that, test the API via the console, making sure to send `release-manifest.json` as the POST payload.

## Project structure

Release Machine follows the standard SAM project structure, and includes the following files and folders:

- functions - Code for the application's Lambda functions
- statemachine - Definition for the state machine that orchestrates the release workflow
- template.yaml - A template that defines the application's AWS resources

## How it works

The `template.yaml` file declares the AWS resources, including Step Functions state machines, Lambda functions and DynamoDB tables. It also sets up permissions

The DynamoDB tables are used to log releases and pipeline execution state.

Resources are defined in the `template.yaml` file in this project. You can update the template to add AWS resources through the same deployment process that updates your application code.

If you prefer to use an integrated development environment (IDE) to build and test the Lambda functions within your application, you can use the AWS Toolkit. The AWS Toolkit is an open source plug-in for popular IDEs that uses the SAM CLI to build and deploy serverless applications on AWS. The AWS Toolkit also adds a simplified step-through debugging experience for Lambda function code. See the following links to get started:

- [PyCharm](https://docs.aws.amazon.com/toolkit-for-jetbrains/latest/userguide/welcome.html)
- [IntelliJ](https://docs.aws.amazon.com/toolkit-for-jetbrains/latest/userguide/welcome.html)
- [VS Code](https://docs.aws.amazon.com/toolkit-for-vscode/latest/userguide/welcome.html)
- [Visual Studio](https://docs.aws.amazon.com/toolkit-for-visual-studio/latest/user-guide/welcome.html)

The AWS Toolkit for VS Code includes full support for state machine visualization, enabling you to visualize your state machine in real time as you build. The AWS Toolkit for VS Code includes a language server for Amazon States Language, which lints your state machine definition to highlight common errors, provides auto-complete support, and code snippets for each state, enabling you to build state machines faster.

## Deploy the application

The Serverless Application Model Command Line Interface (SAM CLI) is an extension of the AWS CLI that adds functionality for building and testing Lambda applications and Step Functions state machines.

To use the SAM CLI, you need the following tools:

- SAM CLI - [Install the SAM CLI](https://docs.aws.amazon.com/serverless-application-model/latest/developerguide/serverless-sam-cli-install.html)
- Node.js - [Install Node.js 12](https://nodejs.org/en/), including the NPM package management tool.
- Docker - [Install Docker community edition](https://hub.docker.com/search/?type=edition&offering=community)

To build and deploy your application for the first time, run the following in your shell:

```bash
$ sam build
$ sam deploy --guided
```