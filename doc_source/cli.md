# AWS CDK Toolkit \(`cdk` command\)<a name="cli"></a>

The AWS CDK Toolkit, the CLI command `cdk`, is the primary tool for interacting with your AWS CDK app\. It executes your AWS CDK app, interrogates the application model you defined, and produces and deploys the AWS CloudFormation templates generated by the AWS CDK\. It also provides other features that are useful for creating and working with AWS CDK projects\. This topic contains information on common use cases of the CDK Toolkit\.

## Toolkit commands<a name="cli-commands"></a>

All CDK Toolkit commands start with `cdk`, which is followed by a subcommand \(`list`, `synthesize`, `deploy`, etc\.\)\. Some subcommands have a shorter version \(`ls`, `synth`, etc\.\) that is equivalent\. Options and arguments follow the subcommand in any order\. The available commands are summarized here\.


| Command | Function | 
| --- | --- | 
| `cdk list` \(`ls`\) | Lists the stacks in the app | 
| `cdk synthesize` \(`synth`\) | Synthesizes and prints the CloudFormation template for the specified stack\(s\) | 
| `cdk bootstrap` | Deploys the CDK Toolkit stack, required to deploy stacks containing assets | 
| `cdk deploy` | Deploys the specified stack\(s\) | 
| `cdk destroy` | Destroys the specified stack\(s\) | 
| `cdk diff` | Compares the specified stack with the deployed stack or a local CloudFormation template | 
| `cdk metadata` | Displays metadata about the specified stack | 
| `cdk init` | Creates a new CDK project in the current directory from a specified template | 
| `cdk context` | Manages cached context values | 
| `cdk docs` \(`doc`\) | Opens the CDK API reference in your browser | 
| `cdk doctor` | Checks your CDK project for potential problems | 

## Built\-in help<a name="cli-help"></a>

The AWS CDK Toolkit has integrated help\. You can see general help about the utility and a list of the provided subcommands by issuing:

```
cdk --help
```

To see help for a particular subcommand, for example `deploy`, specify it before the `--help` flag\.

```
cdk deploy --help
```

Issue `cdk version` to display the version of the AWS CDK Toolkit\. Provide this information when requesting support\.

## Version reporting<a name="version_reporting"></a>

To gain insight into how the AWS CDK is used, the versions of libraries used by AWS CDK applications are collected and reported by using a resource identified as `AWS::CDK::Metadata`\. This resource is added to AWS CloudFormation templates, and can easily be reviewed\. This information can also be used to identify stacks using a package with known serious security or reliability issues, and to contact their users with important information\.

By default, the AWS CDK reports the name and version of the following NPM modules that are loaded at synthesis time:
+ AWS CDK core module
+ AWS Construct Library modules
+ AWS Solutions Constructs module

The `AWS::CDK::Metadata` resource looks something like the following\.

```
CDKMetadata:
  Type: "AWS::CDK::Metadata"
  Properties:
    Modules: "@aws-cdk/core=X.YY.Z,@aws-cdk/s3=X.YY.Z,@aws-solutions-consturcts/aws-apigateway-lambda=X.YY.Z"
```

To opt out of version reporting, use one of the following methods:
+ Use the cdk command with the \-\-no\-version\-reporting argument to opt out for a single command\.

  ```
  cdk --no-version-reporting synth
  ```

  Remember, the AWS CDK Toolkit synthesizes fresh templates before deploying, so you should also add `--no-version-reporting` to `cdk deploy` commands\.
+ Set versionReporting to **false** in `./cdk.json` or `~/.cdk.json`\. This opts out unless you opt in by specifying `--version-reporting` on an individual command\.

  ```
  {
    "app": "...",
    "versionReporting": false
  }
  ```

## Specifying the environment<a name="cli-environment"></a>

 In AWS CDK terms, the [environment](environments.md) consists of a region and AWS credentials valid in that region\. The CDK Toolkit needs credentials in order to query your AWS account and to deploy CloudFormation templates\. 

**Important**  
We strongly recommend against using your AWS root account for day\-to\-day tasks\. Instead, create a user in IAM and use its credentials with the CDK\.

If you have the AWS CLI installed, the easiest way to satisfy this requirement is to install the AWS CLI and issue the following command:

```
aws configure
```

Provide your AWS access key ID, secret access key, and default region when prompted\.

You may also manually create or edit the `~/.aws/config` and `~/.aws/credentials` \(Linux or Mac\) or `%USERPROFILE%\.aws\config` and `%USERPROFILE%\.aws\credentials` \(Windows\) files to contain credentials and a default region, in the following format\.
+ In `~/.aws/config` or `%USERPROFILE%\.aws\config`

  ```
  [default]
  region=us-west-2
  ```
+ In `~/.aws/credentials` or `%USERPROFILE%\.aws\credentials`

  ```
  [default]
  aws_access_key_id=AKIAI44QH8DHBEXAMPLE
  aws_secret_access_key=je7MtGbClwBF/2Zp9Utk/h3yCo8nvbEXAMPLEKEY
  ```

Besides specifying AWS credentials and a region under the `[default]` section, you can also put them in a `[profile NAME]` section, where *NAME* is the name of the profile\. You can add any number of named profiles, with or without a `[default]` section\. Be sure to add the same profile sections to both the configuration and credentials files\. 

**Tip**  
Don't name a profile `default`\. That's just confusing\.

Use the `--profile` flag to choose a set of credentials and default region from these configuration files for a given command\.

```
cdk deploy --profile test PipelineStack
```

Instead of the configuration file, you can set the environment variables `AWS_ACCESS_KEY_ID`, `AWS_SECRET_ACCESS_KEY`, and `AWS_DEFAULT_REGION` to appropriate values\. 

You may optionally use the `--role-arn` \(or `-r`\) option to specify the ARN of an IAM role that should be used for deployment\. This role must be assumable by the AWS account being used\.

## Specifying the app command<a name="cli-app-command"></a>

Many features of the CDK Toolkit require one or more AWS CloudFormation templates be synthesized, which in turn requires running your application\. Since the AWS CDK supports programs written in a variety of languages, it uses a configuration option to specify the exact command necessary to run your app\. This option can be specified in two ways\.

First, and most commonly, it can be specified using the `app` key inside the file `cdk.json`, which is in the main directory of your AWS CDK project\. The CDK Toolkit provides an appropriate command when creating a new project with `cdk init`\. Here is the `cdk.json` from a fresh TypeScript project, for instance\.

```
{
  "app": "npx ts-node bin/hello-cdk.ts"
}
```

The CDK Toolkit looks for `cdk.json` in the current working directory when attempting to run your app, so you might keep a shell open in your project's main directory for issuing CDK Toolkit commands\.

The CDK Toolkit also looks for the app key in `~/.cdk.json` \(that is, in your home directory\) if it can't find it in `./cdk.json`\. Adding the app command here can be useful if you usually work with CDK code in the same language, as it does not require you to be in the app's main directory when you run a `cdk` command\.

If you are in some other directory, or if you want to run your app via a command other than the one in `cdk.json`, you can use the `--app` \(or `-a`\) option to specify it\.

```
cdk --app "npx ts-node bin/hello-cdk.ts" ls
```

## Specifying stacks<a name="cli-stacks"></a>

Many CDK Toolkit commands \(for example, `cdk deploy`\) work on stacks defined in your app\. If your app contains only one stack, the CDK Toolkit assumes you mean that one if you don't specify a stack explicitly\.

Otherwise, you must specify the stack or stacks you want to work with\. You can do this by specifying the desired stacks by name individually on the command line\.

```
cdk synth PipelineStack LambdaStack
```

You may also use wildcards to specify names that match a pattern\.
+ ? matches any single character
+ \* matches any number of characters

When using wildcards, enclose the pattern in quotes\. If you don't, your shell may try to expand the pattern to the names of files in the current directory\. At best, this won't do what you expect; at worst, you could deploy stacks you didn't intend to\.

```
cdk synth "*Stack"    # PipelineStack, LambdaStack, etc.
cdk synth "Stack?"    # StackA, StackB, Stack1, etc.
cdk synth "*"         # All stacks in the app
```

**Note**  
The CDK Toolkit does not guarantee that stacks are processed in the specified order\. If for some reason the order of the stacks is important, use multiple `cdk` commands\.

## Bootstrapping your AWS environment<a name="cli-bootstrap"></a>

Stacks that contain [assets](assets.md) or large AWS Lambda functions require special dedicated AWS CDK resources to be provisioned\. Currently, this is only an Amazon S3 bucket\. The `cdk bootstrap` command creates the necessary resources for you\. You only need to bootstrap if you are deploying a stack that requires these dedicated resources\.

**Important**  
Each region to which you deploy such a stack must be bootstrapped separately\.

You may incur charges for what the AWS CDK stores in the bucket\. Because the AWS CDK does not remove any objects from the bucket, the bucket can accumulate objects as you use the AWS CDK\. From time to time, then, you might want to clear out the bucket from the Amazon S3 console\.

You can use the `--bootstrap-bucket-name` option of `cdk bootstrap` to specify the name of the bootstrap bucket, if the default \(`StagingBucket`\) is not suitable for some reason\. You can use the `--toolkit-stack-name` option if the standard name of the stack itself \(`CDKToolkit`\) is not suitable\.

## Creating a new app<a name="cli-init"></a>

To create a new app, create a directory for it, then, inside the directory, issue `cdk init`\.

```
mkdir my-cdk-app
cd my-cdk-app
cdk init TEMPLATE --language LANGUAGE
```

The supported languages \(*LANGUAGE*\) are:


| Code | Language | 
| --- | --- | 
| `typescript` | TypeScript | 
| `javascript` | JavaScript | 
| `python` | Python | 
| `java` | Java | 
| `csharp` | C\# | 

*TEMPLATE* is an optional template\. If the desired template is *app*, the default, you may omit it\. The available templates are:


| Template | Description | 
| --- | --- | 
| `app` \(default\) |  Creates an empty AWS CDK app\.  | 
| `sample-app` |  Creates an AWS CDK app with a stack containing an Amazon SQS queue and an Amazon SNS topic\.  | 

The templates use the name of the project folder to generate names for files and classes inside your new app\.

## Synthesizing stacks<a name="cli-synth"></a>

The `cdk synthesize` command \(almost always abbreviated `synth`\) synthesizes a stack defined in your app into a CloudFormation template\.

```
cdk synth         # if app contains only one stack
cdk synth MyStack
cdk synth Stack1 Stack2
cdk synth "*"     # all stacks in app
```

**Note**  
The CDK Toolkit actually runs your app and synthesizes fresh templates before most operations \(e\.g\. when deploying or comparing stacks\)\. These templates are stored by default in the cdk\.out directory\. The `cdk synth` command simply prints the generated templates for the specified stack\(s\)\.

See `cdk synth --help` for all available options\. A few of the most\-frequently\-used options are covered below\.

### Specifying context values<a name="w4aac23b7c21c11"></a>

Use the `--context` or `-c` option to pass [runtime context](context.md) values to your CDK app\.

```
# specify a single context value
cdk synth –context key=value MyStack

# specify multiple context values (any number)
cdk synth --context key1=value1 --context key2=value2 MyStack
```

When deploying multiple stacks, the specified context values are passed to all of them\. If you wish, you may specify different values for each stack by prefixing the stack name to the context value\.

```
# different context values for each stack
cdk synth --context Stack1:key=value Stack2:key=value Stack1 Stack2
```

### Specifying display format<a name="w4aac23b7c21c13"></a>

By default, the synthesized template is displayed in YAML format\. Add the `--json` flag to display it in JSON format instead\.

```
cdk synth –json MyStack
```

### Specifying output directory<a name="w4aac23b7c21c15"></a>

Add the \-\-output \(\-o\) option to write the synthesized templates to a directory other than cdk\.out\.

```
cdk synth –output=~/templates
```

## Deploying stacks<a name="cli-deploy"></a>

The `cdk deploy` subcommand deploys the specified stack\(s\) to your AWS account\.

```
cdk deploy        # if app contains only one stack
cdk deploy MyStack
cdk deploy Stack1 Stack2
cdk deploy "*"    # all stacks in app
```

**Note**  
The CDK Toolkit runs your app and synthesizes fresh AWS CloudFormation templates before deploying anything\. Therefore, most command line options you can use with `cdk synth` \(for example, `--context`\) can also be used with `cdk deploy`\.

See `cdk deploy --help` for all available options\. A few of the most\-frequently\-used options are covered below\.

### Specifying AWS CloudFormation parameters<a name="w4aac23b7c23c11"></a>

The AWS CDK Toolkit supports specifying parameters at deployment\. You may provide these on the command line following the `--parameters` flag\.

```
cdk deploy MyStack --parameters uploadBucketName=UploadBucket
```

To define multiple parameters, use multiple `--parameters` flags\.

```
cdk deploy MyStack --parameters uploadBucketName=UpBucket --parameters downloadBucketName=DownBucket
```

If you are deploying multiple stacks, you can specify a different value of each parameter for each stack by prefixing the name of the parameter with the stack name and a colon\. Otherwise, the same value is passed to all stacks\.

```
cdk deploy MyStack YourStack --parameters MyStack:uploadBucketName=UploadBucket --parameters YourStack:uploadBucketName=UpBucket
```

By default, the AWS CDK retains values of parameters from previous deployments and uses them in later deployments if they are not specified explicitly\. Use the `--no-previous-parameters` flag to require all parameters to be specified\.

### Specifying outputs file<a name="w4aac23b7c23c13"></a>

If your stack declares AWS CloudFormation outputs, these are normally displayed on the screen at the conclusion of deployment\. To write them to a file in JSON format, use the `--output-file` flag\.

```
cdk deploy –output-file outputs.json MyStack
```

## Security\-related changes<a name="cli-security"></a>

To protect you against unintended changes that affect your security posture, the AWS CDK Toolkit prompts you to approve security\-related changes before deploying them\.

You can change the level of change that requires approval by specifying:

```
cdk deploy --require-approval LEVEL
```

*LEVEL* can be one of the following:


| Term | Meaning | 
| --- | --- | 
| `never` | Approval is never required | 
| `any-change` | Requires approval on any IAM or security\-group\-related change | 
| `broadening` \(default\) | Requires approval when IAM statements or traffic rules are added; removals don't require approval | 

The setting can also be configured in the `cdk.json` file\.

```
{
  "app": "...",
  "requireApproval": "never"
}
```

## Comparing stacks<a name="cli-diff"></a>

The `cdk diff` command compares the current version of a stack defined in your app with the already\-deployed version, or with a saved AWS CloudFormation template, and displays a list of differences\.

```
[~] AWS::S3::Bucket MyFirstBucket MyFirstBucketB8884501
 ├─ [~] DeletionPolicy
 │   ├─ [-] Retain
 │   └─ [+] Delete
 └─ [~] UpdateReplacePolicy
     ├─ [-] Retain
     └─ [+] Delete
```

To compare your app's stack\(s\) with the existing deployment:

```
cdk diff MyStack
```

To compare your app's stack\(s\) with a saved CloudFormation template:

```
cdk diff --template ~/stacks/MyStack.old MyStack
```

## Help us help you<a name="cli-feedback"></a>

Did you find what you were looking for? We recently revamped this topic and are keenly interested in hearing about how it's working for you\. Please click **Provide feedback** below and let us know\! Providing your e\-mail address is optional, but very helpful in case we have further questions\. \(We'll keep it to ourselves\.\)