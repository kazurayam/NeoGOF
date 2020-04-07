Neo GOF
=====

Author: kazurayam, japan

Date: 7th April 2020

Table of contents:
- [Overview](#overview)
- [Problem to solve](#problem)
- [Solution](#solution)
- [Prerequisites](#prerequisites)
- [Project structure](#project_structure)
- [If you want to try yourself](#if_you_want)
  - [S3 Bucket names need to be globally unique](#unique_bucket_name)
  - [One-liner to generate your identity](#identity)
  - [*.sh files must be executable](#sh_mode)
- [Description1: Neo GOF toolset](#NeoGOF)
  - [Head-first demonstration](#demo)
  - [Internal of the :subprojectD:createStack task](#createStack)gi
- [Conclusion](#conclusion)

<a name="overview" id="overview"></a>
## Overview

Build and Delivery by the toolset of
**Gradle + Shell + AWS CLI + CloudFormation** (new Gang of Four)
makes life easy for Java/Groovy/Kotlin developers.

![Neo GOF Overview](./docs/images/overview.png)

<a name="proble" id="problem"></a>
## Problem to solve

1. I want to use **[Gradle](https://gradle.org/) Build Tool** to achieve
[Continuous Delivery](https://martinfowler.com/bliki/ContinuousDelivery.html).
By one command `$ gradle deploy`, I want to achieve all tasks for developing applications 
in Java/Kotlin language which should run as 
[AWS Lambda](https://aws.amazon.com/lambda/) Funtions.
2. I want to use Gradle to do compiling, testing and archiving for my Java applications as usual.
3. I want to automate provisioning a [AWS S3](https://aws.amazon.com/s3/) Bucket where I locate the jar file containing
my Lambda functions.
4. I want to automate uploading the jar file up to the designated S3 Bucket
every after I modify the application source and rebuild it.
5. I want to automate provisioning other various AWS resources: 
AWS Lambda Functions, CloudWatch Event Rules, 
[IAM Roles](https://docs.aws.amazon.com/IAM/latest/UserGuide/id_roles.html), Simple Queue Service, 
 Simple Notification Service, etc. In order to accomplish these complex tasks,
 I want to use 
**AWS Provisioning Tool [Cloud Formation](https://aws.amazon.com/jp/cloudformation/)**.

A question came up to me. **How can a Gradle `build.gradle` script make full use of AWS CloudFormation?** 

<a name="solution" id="solution"></a>
## Solution

I found 2 possible approaches.

1. use combination of Gradle + Shell + AWS CLI + CloudFormation:  
The `build.gradle` scripts calls built-in `Exec` task which executes 
an external shell script file [`awscli-cooked.sh`](./awscli-cooked.sh) which executes
[AWS CLI](https://aws.amazon.com/cli/) to drive CloudFormation.
2. use a Gradle AWS plugin [jp.classmethod.aws](https://github.com/classmethod/gradle-aws-plugin):
the plugin adds some Gradle tasks for managing various AWS resources including CloudFormation.

I did a research for a few days and got a conclusion that the Gang of Four toolset 
is better than a single Gradle plugin.

I will explain how the Neo GOF toolset works first.
Later I will also explain what I found about the plugin.

<a name="prerequisites" id="prerequisites"></a>
## Prerequisites

- Java 8 or higher
- Bash shell. On Windows I installed 
[Git for Windows](https://gitforwindows.org/) and got bundled "Git Bash".
- AWS Account for my use
- IAM User with enough privileges.
- [AWS CLI](https://aws.amazon.com/cli/) installed and 
[configured](https://docs.aws.amazon.com/cli/latest/userguide/cli-chap-configure.html) on my Mac/PC.
The `default` profile in `~/.aws/credentials` file is configured to point my priviledged IAM User.
- I used Mac, though this project should work on Windows and Linux as well.

<a name="project_structure" id="project_structure"></a>
## Project structure

The NeoGOF project is a Gradle Multi-project, which comprises with 5 sub-projects.
```
$NeoGOF
├─app
├─subprojectA
├─subprojectB
├─subprojectC
└─subprojectD
```

On Commandline UI, I would `cd` to the rootProject, and execute `./gradlew` command.
For example, the `hello` task of the rootProject will call each `hello` tasks defined
each sub-projects.

```
$ cd $NeoGOF
$ ./gradlew -q hello
Hello, app
Hello, subprojectA
Hello, subprojectB
Hello, subprojectB
Hello, subprojectD
Hello, rootProject
```

Or, you can execute a specific task of a subproject by typing 
`:<subPorjectName>:<taskName>`.
For example;

```
$ cd $NeoGOF
$ ./gradlew -q :subprojectA:hello
Hello, subprojectA
```

<a name="if_you_want" id="if_you_want"></a>
## If you want to try yourself

<a name="unique_bucket_name" id="unique_bucket_name"></a>
### S3 Bucket names need to be globally unique

You can download an zip archive of the project from the
[Releases](https://github.com/kazurayam/NeoGOF/releases) page.
Provided that you have seasoned experience of using Gradle
you should be able to play on this project.

If you are going to try running this project on your PC, there is one thing you need to edit.

In `[gradle.properties](./gradle.properties) file, you will find such line:

```
S3BucketNameA=bb4b24b08c-20200406-neogof-a
```

This line specifies the name of a S3 Bucket to be provisioned.
As you know, a S3 Bucket name must be globally unique. 
The Bucket name starting with `bb4b24b08c` is mine, not for your use.
So you need to edit the `gradle.properties` file so that you give 
alternative names for your own use.
I would recommend you to replace the leading `bb4b24b08c` part 
with some other string.

<a name="identity" id="identity"></a>
### One-liner to generate your identity

You want to generate your identity to make your S3 Bucket names globally unique.
Ok, you can generate a mystified (possibly globally unique) 
10 characters based on your AWS Account ID (12 digits) 
by the following shell command. 
Here I assume that you have AWS CLI installed:

```
$ aws sts get-caller-identity --query Account | md5sum | cut -c 1-10
```

<a name="sh_mode" id="sh_mode"></a>
### *.sh files need to be executable

Another thing you need to be aware. Once cloned out, it is likely that
you need to change mode of *.sh files to make them executable. I mean,
you may want to do:

```
$ cd $NeoGOF
$ chmod +x ./awscli-cooked.sh
$ chmod +x ./subprojectD/awscli-cooked.sh
```

<a name="NeoGOF" id="NeoGOF"></a>
## Description1: Neo GOF toolset

You can locate the project anywhere on your PC. 
For example I have cloned out this project to a local directory 
`/Users/myname/github/NeoGOF`.
In the following description, I will use a symbol *`$NeoGOF`* 
for short of this local path.

<a name="demo" id="demo"></a>
### Head-first demonstration

In Bash commandline, type
```
$ cd $NeoGOF
$ ./gradlew -q deploy
```

Then the following output will follow if successful:

```
neogof-0.1.0.jar has been built
created /Users/myname/github/NeoGOF/subprojectD/build/parameters.json
{
    "StackId": "arn:aws:cloudformation:ap-northeast-1:84**********:stack/StackD/99bd96c0-78c9-11ea-b8e1-060319ee749a"
}
Neo GOF project has been deployed

```

Here I wrote `84**********`. This portion is the 12 digits of my AWS Account ID.
You will see different 12 digits of your AWS Account ID when you tried yourself.

Many things will be performed behind the sean. Let me follow the code
and explain the detail.

When you type `gradlew deploy`, the `deploy` task defined in the 
[NeoGOF/build.gradle](./build.gradle) is executed.

```
task deploy(dependsOn: [
        ":app:build",
        ":subprojectD:createStack"
]) {
    doLast {
        println "Neo GOF project has been deployed"
    }
}
```

The `deploy` task calls 2 tasks: `:app:build` and `:projectD:createStack`;
and when they finished the `deploy` task shows a farewell message. Of course 
you can execte this task independentlyas:

```
$ cd $NeoGOF
$ ./gradlew :app:build
...
$ ./gradlew :subprojectD/createStack
...
```

The `app` sub-project is a small Gradle project with `java` plugin applied.

The `app` sub-project contains a Java class `example.HelloPojo`. Theclass implements 
`com.amazonaws.services.lambda.runtime.RequestHandler`. 
Therefore the `example.Hello` class can run as a AWS Lambda Function.

The `build` task of the `app` project compiles the Java source and 
build a jar file. 
The `app` project is a typical Gradle project and has nothing new.

----

The `subprojectD` sub-project indirectly activates AWS CloudFormation to
 provision S3 Bucket and IAM role. 
 
Please note, **the `deploy` task combines a Gradle built-in feature 
(building Java application) and a extended feature 
(driving AWS CloudFormation) just seamlessly**.

<a name="createStack" id="createStack"></a>
### Internal of the `:subprojectD:createStack` task
 
The `createStack` task in 
 [`subprojectD/build.gradle`](./subprojectD/build.gradle) 
 file is like this:
 
```
task createStack {
    doFirst {
        copy {
            from "$projectDir/src/cloudformation"
            into "$buildDir"
            include 'parameters.json.template'

            rename { file -> 'parameters.json' }
            expand (
                    bucketName: "${S3BucketNameD}"
            )
        }
        println "created ${buildDir}/parameters.json"
    }
    doLast {
        exec {
            workingDir "${projectDir}"
            commandLine './awscli-cooked.sh', 'createStack'
        }
    }
}
```

The `createStack` task does 2 things.

First it executes a `copy` task.
The `copy` task prepares a set of parameters to be passed to CloudFormation Template.
It copies a template file into `build/parameters.json` while interpolating
a `$bucketName` symbol in the template to the value specified in the `rootProject/gradle.properties` file.

Secondly it executes a `exec` task which executes an external bash script file 
[`awscli-cooked.sh`](./awscli-cooked.sh) with sub-command `createStack`. 
Let's have a quick look at the code fragment:

```
sub_createStack() {
    aws cloudformation create-stack --template-body $Template --parameters $Parameters --stack-name $StackName --capabilities CAPABILITY_NAMED_IAM
}
```

Any AWS developer will easily see what this shell function does.
The shell function 
`sub_createStack` invokes AWS CLI to activate CloudFormation 
for creating a Stack with several options/parameters specified as appropriate.

The shell script [`awscli-cooked.sh`](./awscli-cooked.sh) implements a few 
other subcommands: `deleteStack`, `describeStacks`, `validateTemplate`. 
All of them are one-liners which invokes AWS CLI to active CloudFormation.

Easy to understand, isn't it?


## Description2: Gradle AWS Plugin --- what we can and can not


<a name="conclusion" id="conclusion"></a>
## Conclusion

I want to express my appreciations and respects to the developers of
the Gradle AWS Plugin 
[jp.classmethod.aws](https://github.com/classmethod/gradle-aws-plugin).
However I would note that small Gradle plugin projects 
for managing AWS resources may fail keeping pace with 
the rapid and continuous development of AWS services.

On the contrary, AWS CLI and AWS CloudFormation --- these are the primary
products which AWS fully supports to make their services available to users.
Users can safely rely on the CLI and CF.

Therefore a Gradle `build.gradle` which executes indirectly CloudFormation 
via Shell+CLI is assured that it can utilize full features of AWS services
together with Gradle's built-in features such as building a AWS Lambda 
Function in Java. 

The combination of Gradle + Shell + AWS CLI + CloudFormation (Neo GOF) 
is a powerful toolset. It will remain easy to maintain in future.








 