# Release Machine workshop

Have you ever needed to coordinate deployments from multiple Git repositories at once?

In this workshop, I'll demonstrate a way to use StepFunctions (via SAM) to orchestrate a multi-component, multi-pipeline software release. Instead of triggering the pipelines individually we’ll get StepFunctions to orchestrate it for us, by reading a release manifest file that will specify the components to deploy.

![alt text](https://github.com/guysqr/release-machine/raw/master/release-machine.svg?sanitize=true 'Successful Execution')

The workshop will allow you to quickly set up the multiple CodeCommit repos and CodePipeline pipelines we’ll need using python and the AWS CDK, so we recommend installing and getting those working beforehand. We will also be using SAM to build our StepFunctions and the new StepFunctions extension for VS Code. All the code needed will be provided.

As all the services used are serverless, the cost of running this workshop will be minimal, but it’ll also be easy to destroy via the CDK afterwards.

## Steps

1. Install the [prerequisites](prerequisites.md) _(do/check this prior to the Workshop if possible)_
2. Do [Part One](part-one.md) - create the demo infrastructure
3. Do [Part Two](part-two.md) - create and test the Step Functions state machine
4. [Tear it all down](teardown.md)
