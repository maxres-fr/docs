---
title: JetBrains TeamCity

menu:
    userguides:
        parent: cont_delivery
        weight: 1

aliases:
- /docs/reference/cd-jetbrains-teamcity/
- /docs/console/continuous-delivery/jetbrains-teamcity/
---

This page details how to use [JetBrains TeamCity](https://www.jetbrains.com/teamcity/) to deploy some sample infrastructure
to AWS using Pulumi.

## Prerequisities

- A working installation of TeamCity
- An account on the [Pulumi Console](https://app.pulumi.com).
- The latest version of Pulumi. Installation instructions are [here]({{< relref "/docs/get-started/install" >}}).
- Setup a new project and [stack]({{< relref "/docs/intro/concepts/stack" >}}) using one of our 
[Get Started]({{< relref "/docs/get-started" >}}) guides or simply by running [`pulumi new`]({{< relref "/docs/reference/cli/pulumi_new.md" >}})
and choosing one of the many templates that are available.

## Sample Project

The example we are going to deploy is located [here](https://github.com/pulumi/examples/tree/master/aws-ts-hello-fargate). 
You may download the project and upload it to your own repo to avoid having to clone the entire Pulumi Examples repo onto 
your TeamCity server.

## Configuring the TeamCity Project

For the purposes of this guide, we are going to build out a TeamCity project via the UI. An alternative way to do this, would
be to use the [Kotlin DSL](https://www.jetbrains.com/help/teamcity/kotlin-dsl.html). When creating a new project via the UI,
we will see the following creation wizard:

![TeamCity Project Creation Wizard](/images/docs/reference/teamcity/new-project.png)

We are going to follow the TeamCity project setup `From a repository URL`. This means we can enter the URL to the 
[Pulumi Examples Repository](https://github.com/pulumi/examples). This specific respository is open source, so we do not
need to enter a `Username` or a `Password`. We can then `Proceed`.

TeamCity will tell us if it can successfully connect to the repository. If it can, then it will ask us to give this project
a name and to create a [build configuration])(https://www.jetbrains.com/help/teamcity/build-configuration.html) name.

![TeamCity Project Naming](/images/docs/reference/teamcity/project-name.png)

Let's call the project `Pulumi Example` and let's create a build configuration called `Development Environment`. We can 
`Proceed` to the next step. TeamCity will then try and find any build steps that it can find in the repository. We can click
the link that says `configure build steps manually` and then we can start creating our project.

A TeamCity configuration is made of [build steps](https://www.jetbrains.com/help/teamcity/configuring-build-steps.html).
We can break down the steps required for us to deploy this application into a number of build steps.

### Ensure NodeJS & NPM are installed

The first thing we need to do is to ensure that [NodeJS](https://nodejs.org/en/) and [NPM](https://www.npmjs.com/) are 
installed on our build agents. We would chose `Command Line` as the [build runner](https://www.jetbrains.com/help/teamcity/build-runner.html) 
type and TeamCity will present us with the build runner wizard

![NodeJS and NPM Install](/images/docs/reference/teamcity/nodejs-install.png)

We are going to name the step `Install NodeJS & NPM`. The working directory will be `aws-ts-hello-fargate` and the contents
of the custom scriptm will be:

```bash
    yum -y install nodejs
    npm version
``` 

We can `Save` the step. Let's add the next build step by clicking on `Add Build Step`. 

![Build Steps](/images/docs/reference/teamcity/build-steps-1.png)

### Installing Pulumi

Again, we are going to chose a `Command Line` build runner and we can create the build step as follows

![Pulumi Install](/images/docs/reference/teamcity/pulumi-install.png)

### Restoring NPM Dependencies

The next build step we are going to create is to restore the NPM dependencies required to deploy our application. Let's create
a new `Command Line` build runner and create the build step as follows

![Restore Dependencies](/images/docs/reference/teamcity/restore-dependencies.png)

### Pulumi Stack Creation

Let's create another `Command Line` build runner that we can configure to create our Pulumi [stack]({{< relref "/docs/intro/concepts/stack" >}})
as follows

![Stack creation](/images/docs/reference/teamcity/stack-creation.png)

### Pulumi Up...

The last build step we need to create is to instruct TeamCity to run the `pulumi up` command. We can create a `Command Line`
runner and configure it as follows

![Pulumi Up](/images/docs/reference/teamcity/pulumi-up.png)

### Configuring Build Parameters

The last thing we need to do before we `run` the build is to configure the parameters needed to allow Pulumi to be executed.
There are a number of things we need to set here:

* AWS_ACCESS_KEY_ID     - environment variable
* AWS_SECRET_ACCESS_KEY - environment variable
* PULUMI_ACCESS_TOKEN   - environment variable
* AWS_REGION            - configuration parameter
* STACK_NAME            - configuration parameter
* PATH                  - we need to update the PATH env var to include the Pulumi installation

The [TeamCity Documentation](https://www.jetbrains.com/help/teamcity/configuring-build-parameters.html) describes how to
create these parameters.  

![Build Parameters](/images/docs/reference/teamcity/build-parameters.png)

### Running the build

We can now run the build by clicking on the `Run` button. After a few minutes, we should have a successful build. On inspection
of the build log, we can see that the infrastructure has been successfully deployed:

```bash
[17:05:48][Step 5/5]                     exec: {
[17:05:48][Step 5/5]                         apiVersion: "client.authentication.k8s.io/v1alpha1"
[17:05:48][Step 5/5]                         args      : [
[17:05:48][Step 5/5]                             [0]: "token"
[17:05:48][Step 5/5]                             [1]: "-i"
[17:05:48][Step 5/5]                             [2]: "helloworld-eksCluster-c5bd220"
[17:05:48][Step 5/5]                         ]
[17:05:48][Step 5/5]                         command   : "aws-iam-authenticator"
[17:05:48][Step 5/5]                     }
[17:05:48][Step 5/5]                 }
[17:05:48][Step 5/5]             }
[17:05:48][Step 5/5]         ]
[17:05:48][Step 5/5]     }
[17:05:48][Step 5/5]     namespaceName  : "helloworld-eyay6eno"
[17:05:48][Step 5/5]     serviceHostname: "a2b21d9e5f4ee11e99d67026bafffdcc-603547860.us-east-2.elb.amazonaws.com"
[17:05:48][Step 5/5]     serviceName    : "helloworld-d4oadgeg"
[17:05:48][Step 5/5] 
[17:05:48][Step 5/5] Resources:
[17:05:48][Step 5/5]     + 40 created
[17:05:48][Step 5/5] 
[17:05:48][Step 5/5] Duration: 13m0s
[17:05:48][Step 5/5] 
[17:05:48][Step 5/5] Permalink: https://app.pulumi.com/stack72/aws-ts-eks-hello-world/development-example/updates/1
[17:05:48][Step 5/5] Process exited with code 0
```






