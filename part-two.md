# Release Machine Demo Part Two

The second part of the demo is the fun part. Here we're going to use AWS SAM to set up a Step Functions state machine to run our pipelines.

SAM is great because it does a bunch of undifferentiated heavy lifting for us. When we declare AWS::Serverless::\* resources, we are taking advantage of AWS-managed CloudFormation macros that expand those resources to [add helpful things like roles and policies](https://docs.aws.amazon.com/AWSCloudFormation/latest/UserGuide/transform-aws-serverless.html) that are required to make them work.

This SAM project will do the following:

- Create the Step Functions state machine
- Create the Lambda functions, S3 bucket & API gateway endpoint used by the state machine
- Create and configure the additional services, roles and other stuff required by the State Machine

## Prerequisites

You've already [installed the SAM CLI](https://docs.aws.amazon.com/serverless-application-model/latest/developerguide/serverless-sam-cli-install.html), right?

## Getting Started

Check your current directory (you may want to `cd ~` first) then clone the repo and cd into it.

```
git clone https://github.com/guysqr/release-machine
cd release-machine
```

### Project structure

Release Machine follows the standard SAM project structure, and includes the following files and folders:

- functions - Code for the application's Lambda functions
- statemachine - Definition for the state machine that orchestrates the release workflow
- template.yaml - A template that defines the application's AWS resources

### How it works

The SAM template `template.yaml` declares the AWS resources we'll be using, including Step Functions state machines, Lambda functions and DynamoDB tables (used to log releases and pipeline execution state).

[SAM templates](https://docs.aws.amazon.com/serverless-application-model/latest/developerguide/sam-specification.html) are essentially a special kind of CloudFormation template that allows you to use shorthand constructs that get processed into "normal" CloudFormation during deployment.

The state machine JSON file is written in [Amazon States Language](https://docs.aws.amazon.com/step-functions/latest/dg/concepts-amazon-states-language.html). The Amazon States Language is a JSON-based, structured language used to define your state machine, a collection of states, that can do work (Task states), determine which states to transition to next (Choice states), stop an execution with an error (Fail states), and so on.

## Let's Build!

Let's get this party started. Time to run

```
sam build
```

If that worked correctly it will prompt you to do a _guided deployment_. You should use guided mode the first time to set various parameters, but after that you can drop the `--guided` bit off.

I suggest you use the same defaults I have set here.

```
sam deploy --guided

Configuring SAM deploy
======================

        Looking for samconfig.toml :  Found
        Reading default arguments  :  Success

        Setting default arguments for 'sam deploy'
        =========================================
        Stack Name []: release-machine
        AWS Region []: ap-southeast-2
        #Shows you resources changes to be deployed and require a 'Y' to initiate deploy
        Confirm changes before deploy [Y/n]: Y
        #SAM needs permission to be able to create roles to connect to the resources in your template
        Allow SAM CLI IAM role creation [Y/n]: Y
        Save arguments to samconfig.toml [Y/n]: Y

```

Once you hit enter you should see

```

        Looking for resources needed for deployment: Not found.
        Creating the required resources...
        Successfully created!

                Managed S3 bucket: aws-sam-cli-managed-default-samclisourcebucket-178onu6euafev
                A different default S3 bucket can be set in samconfig.toml

        Saved arguments to config file
        Running 'sam deploy' for future deployments will use the parameters saved above.
        The above parameters can be changed by modifying samconfig.toml
        Learn more about samconfig.toml syntax at
        https://docs.aws.amazon.com/serverless-application-model/latest/developerguide/serverless-sam-cli-config.html

        Deploying with following values
        ===============================
        Stack name                 : release-machine
        Region                     : ap-southeast-2
        Confirm changeset          : True
        Deployment s3 bucket       : aws-sam-cli-managed-default-samclisourcebucket-178jsadhfkjhasl
        Capabilities               : ["CAPABILITY_IAM"]
        Parameter overrides        : {}

Initiating deployment
=====================

Waiting for changeset to be created..
```

If a changeset is created you will then get prompted to deploy it. Enter `y` to begin deployment.

While you're waiting for that to complete, let's recap what we're doing here and why:

## About the Release Machine

The Release Machine is a software release workflow for deploying from multiple pipelines simultaneously, triggered by a `release-manifest.json` file dropped in an S3 bucket or posted to an API Gateway endpoint.

![alt text](https://github.com/guysqr/release-machine/raw/master/release-machine.svg?sanitize=true 'Successful Execution')

The release process contains steps that

1. Verify the manifest is valid and has not already been run
2. If valid, records the release request and timestamp
3. Triggers an execution for each of the requested pipelines
4. Checks each pipeline execution's state every 30 seconds and records any changes
5. Records the release as complete if all pipelines complete successfully
6. Fails the release if any fail to complete successfully

> Note that this version does not attempt any rollbacks or pipeline execution cancellations when something fails, but these could definitely be added as steps, depending on how the deployment is done. Using CodeDeploy, for instance, stores previous releases to which Release Machine could roll back.

Hopefully your infrastructure all built without issues, in which case it's now time to run it and see what happens!

### Triggering the Release Machine

If your SAM deployment went as planned, you should be able to go to [the Step Functions console](https://ap-southeast-2.console.aws.amazon.com/states/home?region=ap-southeast-2#/statemachines) and find your state macnine "ReleaseStateMachine".

#### Configuring your first release

Edit `release-manifest.json` so that the names of the pipelines match the names of [your pipelines](https://ap-southeast-2.console.aws.amazon.com/codesuite/codepipeline/pipelines). If you didn't change the config in step one, these will be called "pipeline-for-workshoprepo-1" and so on.

```json
{
  "releaseId": 5.15,
  "pipelines": ["pipeline-for-workshoprepo-1", "pipeline-for-workshoprepo-2"]
}
```

The Release Machine has been set up to be triggered by either writing the `release-manifest.json` file to the release machine S3 bucket, or by posting the `release-manifest.json` file's contents to the release machine API Gateway endpoint.

To see the magic happen via the first method, go to [the S3 console](https://s3.console.aws.amazon.com/s3) and upload the `release-manifest.json` file from this directory to the release-machine S3 bucket that SAM has created.

To see the magic happen via the second method, go to [the API Gateway console](https://ap-southeast-2.console.aws.amazon.com/apigateway/main/apis?region=ap-southeast-2) and click on the API called `release-machine`. Click on the word POST in the Resources list, then click TEST.

![API Gateway test](img/api-gateway.png 'API Gateway test')

Copy the contents of the `release-manifest.json` file from this directory to your clipboard and paste it into the Request Body box then hit "Test". You should see a response body pop up that looks similar to this:

```json
{
  "executionArn": "arn:aws:states:ap-southeast-2:76332347855:execution:ReleaseStateMachine-l0NkeqrA8Z0X:bdec481c-5fac-44e9-9a26-ea234a438063",
  "startDate": 1593322928.575
}
```

### Reviewing progress

Go to [the Step Functions console](https://ap-southeast-2.console.aws.amazon.com/states/home) and click on the state machine that begins with `ReleaseStateMachine-`. In the 'Executions' list you should see an execution In Progress. If you click on the current execution you will see the visual workflow showing each state in the execution and any errors that are occurring.

Go to [the CodePipeline pipelines list](https://ap-southeast-2.console.aws.amazon.com/codesuite/codepipeline/pipelines) to see which pipelines are being executed.

Go to [the DynamoDB console](https://ap-southeast-2.console.aws.amazon.com/dynamodb/home?region=ap-southeast-2) to see what is being recorded in the Releases and Executions tables.

### Configuring a second release

Note you will need to change the releaseId value in the `release-manifest.json` file to trigger a second and subsequent deployment. This is also how you change which pipelines you want to trigger.

The idea is that this file is generated as part of a process that determines which of your application components needs to be deployed as part of that release. So, you might have a release that needs the component built by pipelines 1 and 3, and next time you do a release it requires pipelines 2, 4 and 5. These would result in manifest files that look like:

```json
{
  "releaseId": 5.15,
  "pipelines": ["pipeline-for-workshoprepo-1", "pipeline-for-workshoprepo-3"]
}
```

and

```json
{
  "releaseId": 5.16,
  "pipelines": ["pipeline-for-workshoprepo-2", "pipeline-for-workshoprepo-4", "pipeline-for-workshoprepo-5"]
}
```

## Experimentation Zone

Once you have the above working, try the following:

Find the API Gateway endpoints for the Lambda functions and load them up in your browser. Try modifying the code via the CodeCommit console. The related pipelines are set up to run automatically on commit. See if you can disable the automatic deploy on commit so deployments only happen via the release machine.

Open up the X-Ray console and see what you can see.

Try introducing some errors - eg bad data in the `release-manifest.json`. Try repeating the same release.

Take a look at the `template.yaml` file and see if you can figure out what all the parts do. Try adding some more resources and see if you can deploy them as part of the stack.

Take a look at the `release_machine.asl.json` file (inside the statemachine folder) and see if you can modify it and how the linter and visualiser in VS Code works.

## Next Step

[Tear it all down](teardown.md)
