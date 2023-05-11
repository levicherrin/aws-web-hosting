![Under Construction](construction.jpg)

# Deep Dive - Cross-Account CI/CD Pipeline + IaC
This walkthrough will build off of phase5 and phase6 by creating a pipeline for automated test and production infrastructure deployments.

## To-Do List:
- finalize architecture diagram
- automate account setup
- refine iam permissions
- review template portability (regions)

## Architecture Overview
The pipeline operates across multiple accounts to create the web hosting infrastructure in one account for testing and review then another account for production use. Infrastructure deployments will be performed through CloudFormation with minimum management console interactions.

#### Service Descriptions
<!-- (TOC:collapse=true&collapseText=Click to expand) -->
<details>
<summary>(click to expand)</summary>

- **CloudFormation** is a service that helps you model and set up your AWS resources so that you can spend less time managing those resources and more time focusing on your applications that run in AWS.

- **CodeBuild** is a fully managed continuous integration service that compiles source code, runs tests, and produces ready-to-deploy software packages.

- **CodePipeline** is a continuous delivery service you can use to model, visualize, and automate the steps required to release your software.

- **Simple Storage Service (S3)** is an object storage service that offers industry-leading scalability, data availability, security, and performance. You can use Amazon S3 to store and retrieve any amount of data at any time, from anywhere.

</details>

![Pipeline Architecture](pipelineArchitecture.jpg)


## Walkthrough


**NOTE**: Throughout each phase there are `my statements` which are placeholder values that must be replaced by the actual values specific to your deployment. Example below:

`arn:aws:s3:::MY-BUCKET-NAME/*` ------SHOULD BE CHANGED TO------> `arn:aws:s3:::cool-website-123/*`

### Pre Reqs

What:
1. AWS management account and IAM user with administrative permissions

2. An organization created in the management account with TOOLS, TEST, and PROD children accounts

3. A Route53 public hosted zone configured for a custom domain name in the PROD account

    3a. NS record pointing to the subdomain's servers

4. A Route53 public hosted zone configured for a subdomain in the TEST account

Resources:
- [AWS account getting started](https://docs.aws.amazon.com/accounts/latest/reference/welcome-first-time-user.html)

### Step 1 - Create the TEST and PROD IAM Roles

Two IAM roles are needed in both the TEST and PROD accounts before the pipeline can function. First, a cross account role is required for the pipeline action with a trust relationship to the TOOLS account.

```
{
  "Version": "2012-10-17",
  "Statement": [
      {
          "Effect": "Allow",
          "Principal": {
              "AWS": "arn:aws:iam::MY-TOOLS-ACCOUNT-ID:root"
          },
          "Action": "sts:AssumeRole",
          "Condition": {}
      }
  ]
}
```

This role will be used to decrypt the S3 artifact bucket objects and create the cloudformation stack which requires the following policy.

```
{
  "Version": "2012-10-17",
  "Statement": [
      {
          "Action": [
              "cloudformation:*",
              "s3:*",
              "iam:PassRole"
          ],
          "Resource": "*",
          "Effect": "Allow"
      },
      {
          "Action": [
              "kms:Decrypt",
              "kms:Encrypt"
          ],
          "Resource": "arn:aws:kms:us-east-1:MY-TOOLS-ACCOUNT-ID:key/MY-TOOLS-KMSKey-ID",
          "Effect": "Allow"
      }
  ]
}
```

The second role is for cloudformation to assume when executing the stack. A basic trust policy with cloudformation is required.

```
{
  "Version": "2012-10-17",
  "Statement": [
      {
          "Sid": "",
          "Effect": "Allow",
          "Principal": {
              "Service": "cloudformation.amazonaws.com"
          },
          "Action": "sts:AssumeRole"
      }
  ]
}
```

This role needs a policy with permissions required to create the web hosting infrastructure.

```
{
  "Version": "2012-10-17",
  "Statement": [
      {
          "Effect": "Allow",
          "Resource": "*",
          "Action": [
              "s3:*",
              "cloudformation:*",
              "lambda:*",
              "route53:*",
              "codestar-connections:*",
              "ses:*",
              "cloudfront:*",
              "acm:*",
              "apigateway:*",
              "iam:*"
          ]
      },
      {
          "Action": [
              "kms:Decrypt",
              "kms:Encrypt"
          ],
          "Resource": "MY-TOOLS-KMSKey-ARN",
          "Effect": "Allow"
      }
  ]
}
```

### Step 2 - Create the TOOLS IAM Roles

Create the CodeBuild role with a trust relationship to the codebuild service and policy for s3 and logs.

```
  codebuildRole:
    Type: AWS::IAM::Role
    Properties:
      AssumeRolePolicyDocument:
        Version: '2012-10-17'
        Statement:
        - Effect: Allow
          Principal:
            Service:
            - codebuild.amazonaws.com
          Action:
          - sts:AssumeRole
      Policies:
      - PolicyName: codebuildPolicy
        PolicyDocument:
          Version: '2012-10-17'
          Statement:
          - Action:
            - s3:*
            - logs:*
            Resource: "*"
            Effect: Allow
```


Create the Pipeline role with a policy to use the connection, perform s3 actions, pass IAM roles, and assume the cross account roles.

```
  pipelineRole:
    Type: AWS::IAM::Role
    Properties:
      AssumeRolePolicyDocument:
        Version: '2012-10-17'
        Statement:
        - Effect: Allow
          Principal:
            Service:
            - codepipeline.amazonaws.com
          Action:
          - sts:AssumeRole
      Policies:
      - PolicyName: pipePolicy
        PolicyDocument:
          Version: '2012-10-17'
          Statement:
          - Action:
            - codestar-connections:UseConnection
            Resource:
              Ref: myConnection
            Effect: Allow
          - Action:
            - s3:PutObject
            - s3:GetObject
            Resource: "*"
            Effect: Allow
          - Action:
            - cloudformation:*
            - iam:PassRole
            - codebuild:*
            Resource: "*"
            Effect: Allow
          - Action:
            - sts:AssumeRole
            Resource:
              Fn::Sub: arn:aws:iam::${prodAccount}:role/cross-account-role
            Effect: Allow
          - Action:
            - sts:AssumeRole
            Resource:
              Fn::Sub: arn:aws:iam::${testAccount}:role/cross-account-role
            Effect: Allow
```

### Step 3 - Create Pipeline Supporting Resources

Create the encryption key to encrypt/decrypt the artfact bucket objects.

```
  KMSKey:
    Type: AWS::KMS::Key
    Properties:
      EnableKeyRotation: true
      KeyPolicy:
        Version: '2012-10-17'
        Id: pipeline-kms-key
        Statement:
        - Sid: Allows admin of the key
          Effect: Allow
          Principal:
            AWS:
              Fn::Sub: arn:aws:iam::${AWS::AccountId}:root
          Action:
          - kms:*
          Resource: "*"
        - Sid: Allow use of the key from the other accounts
          Effect: Allow
          Principal:
            AWS:
            - Fn::Sub: arn:aws:iam::${prodAccount}:root
            - Fn::Sub: arn:aws:iam::${prodAccount}:role/prodCfnDeployRole
            - Fn::Sub: arn:aws:iam::${testAccount}:root
            - Fn::Sub: arn:aws:iam::${testAccount}:role/testCfnDeployRole
            - Fn::GetAtt:
              - pipelineRole
              - Arn
            - Fn::GetAtt:
              - codebuildRole
              - Arn
          Action:
          - kms:Encrypt
          - kms:Decrypt
          - kms:ReEncrypt*
          - kms:GenerateDataKey*
          - kms:DescribeKey
          Resource: "*"
```

Create the artifact bucket and a bucket policy that allows the TOOLS, TEST, and PROD IAM roles access

```
artBucket:
Type: AWS::S3::Bucket
DeletionPolicy: Retain
Properties:
    PublicAccessBlockConfiguration:
    BlockPublicAcls: 'true'
    BlockPublicPolicy: 'true'
    IgnorePublicAcls: 'true'
    RestrictPublicBuckets: 'true'
artBucketPolicy:
Type: AWS::S3::BucketPolicy
Properties:
    Bucket:
    Ref: artBucket
    PolicyDocument:
    Version: '2012-10-17'
    Statement:
    - Action:
        - s3:*
        Effect: Allow
        Resource:
        - Fn::Sub: arn:aws:s3:::${artBucket}
        - Fn::Sub: arn:aws:s3:::${artBucket}/*
        Principal:
        AWS:
        - Fn::Sub: arn:aws:iam::${prodAccount}:role/cross-account-role
        - Fn::Sub: arn:aws:iam::${prodAccount}:role/prodCfnDeployRole
        - Fn::Sub: arn:aws:iam::${testAccount}:role/cross-account-role
        - Fn::Sub: arn:aws:iam::${testAccount}:role/testCfnDeployRole
        - Fn::GetAtt:
            - pipelineRole
            - Arn
        - Fn::GetAtt:
            - codebuildRole
            - Arn
```

Create the CodeBuild project

```
  myBuild:
    Type: AWS::CodeBuild::Project
    Properties:
      Artifacts:
        Type: CODEPIPELINE
      Environment:
        ComputeType: BUILD_GENERAL1_SMALL
        EnvironmentVariables:
        - Name: S3_BUCKET
          Value:
            Ref: artBucket
        Image: aws/codebuild/amazonlinux2-x86_64-standard:4.0
        Type: LINUX_CONTAINER
      Name: my-build-project
      ServiceRole:
        Fn::GetAtt:
        - codebuildRole
        - Arn
      EncryptionKey:
        Fn::GetAtt:
        - KMSKey
        - Arn
      Source:
        Type: CODEPIPELINE
      TimeoutInMinutes: '5'
```

Create the connection to the GitHub repository

```
  myConnection:
    Type: AWS::CodeStarConnections::Connection
    Properties:
      ConnectionName: myConnection
      ProviderType: GitHub
```

### Step x - Create the Pipeline

Source stage

```
- Name: GitHubSource
    InputArtifacts: []
    ActionTypeId:
    Category: Source
    Owner: AWS
    Provider: CodeStarSourceConnection
    Version: '1'
    OutputArtifacts:
    - Name: SourceArtifact
    Configuration:
    ConnectionArn:
        Fn::GetAtt:
        - myConnection
        - ConnectionArn
    FullRepositoryId:
        Ref: myRepo
    BranchName:
        Ref: myBranch
    RunOrder: '1'
```

Build stage

```
- Name: Build
Actions:
- Name: PackageExport
    InputArtifacts:
    - Name: SourceArtifact
    ActionTypeId:
    Category: Build
    Owner: AWS
    Provider: CodeBuild
    Version: '1'
    OutputArtifacts:
    - Name: websiteArt
    - Name: lambdaDeploymentArt
    Configuration:
    ProjectName: my-build-project
    RunOrder: '1'
```

Deploy to test stage

```
- Name: DeployToTest
Actions:
- Name: CreateChangeSetTest
    InputArtifacts:
    - Name: SourceArtifact
    - Name: lambdaDeploymentArt
    ActionTypeId:
    Category: Deploy
    Owner: AWS
    Provider: CloudFormation
    Version: '1'
    Configuration:
    ActionMode: CHANGE_SET_REPLACE
    Capabilities: CAPABILITY_NAMED_IAM
    ChangeSetName: test-pipeline-changeset
    ParameterOverrides: |-
        {
        "myBucket" : { "Fn::GetArtifactAtt" : ["lambdaDeploymentArt", "BucketName"]},
        "myLambdaPackage" : { "Fn::GetArtifactAtt" : ["lambdaDeploymentArt", "ObjectKey"]}
        }
    RoleArn:
        Fn::Sub: arn:aws:iam::${testAccount}:role/testCfnDeployRole
    StackName: test-infra
    TemplateConfiguration: SourceArtifact::cloudFormation/testConfig.yml
    TemplatePath: SourceArtifact::cloudFormation/master.yml
    RoleArn:
    Fn::Sub: arn:aws:iam::${testAccount}:role/cross-account-role
    RunOrder: '1'
- Name: ExecuteChangeSetTest
    Namespace: testCloudformationVars
    InputArtifacts:
    - Name: SourceArtifact
    - Name: lambdaDeploymentArt
    ActionTypeId:
    Category: Deploy
    Owner: AWS
    Provider: CloudFormation
    Version: '1'
    Configuration:
    ActionMode: CHANGE_SET_EXECUTE
    ChangeSetName: test-pipeline-changeset
    ParameterOverrides: |-
        {
        "myBucket" : { "Fn::GetArtifactAtt" : ["lambdaDeploymentArt", "BucketName"]},
        "myLambdaPackage" : { "Fn::GetArtifactAtt" : ["lambdaDeploymentArt", "ObjectKey"]}
        }
    RoleArn:
        Fn::Sub: arn:aws:iam::${testAccount}:role/testCfnDeployRole
    StackName: test-infra
    RoleArn:
    Fn::Sub: arn:aws:iam::${testAccount}:role/cross-account-role
    RunOrder: '2'
- Name: DeployTestBucket
    InputArtifacts:
    - Name: websiteArt
    ActionTypeId:
    Category: Deploy
    Owner: AWS
    Provider: S3
    Version: '1'
    Configuration:
    BucketName: "#{testCloudformationVars.webBucket}"
    Extract: 'true'
    RoleArn:
    Fn::Sub: arn:aws:iam::${testAccount}:role/cross-account-role
    RunOrder: '3'
```

Manual Review  stage

```
- Name: ApproveProdDeploy
Actions:
- Name: ApproveDeployProd
    ActionTypeId:
    Category: Approval
    Owner: AWS
    Provider: Manual
    Version: '1'
    Configuration:
    CustomData: Review and verify changes in the DEV account before approving
```

Deploy to prod  stage

```
- Name: DeployToProd
Actions:
- Name: CreateChangeSetProd
    InputArtifacts:
    - Name: SourceArtifact
    - Name: lambdaDeploymentArt
    ActionTypeId:
    Category: Deploy
    Owner: AWS
    Provider: CloudFormation
    Version: '1'
    Configuration:
    ActionMode: CHANGE_SET_REPLACE
    Capabilities: CAPABILITY_NAMED_IAM
    ChangeSetName: prod-pipeline-changeset
    ParameterOverrides: |-
        {
        "myBucket" : { "Fn::GetArtifactAtt" : ["lambdaDeploymentArt", "BucketName"]},
        "myLambdaPackage" : { "Fn::GetArtifactAtt" : ["lambdaDeploymentArt", "ObjectKey"]}
        }
    RoleArn:
        Fn::Sub: arn:aws:iam::${prodAccount}:role/prodCfnDeployRole
    StackName: prod-infra
    TemplateConfiguration: SourceArtifact::cloudFormation/prodConfig.yml
    TemplatePath: SourceArtifact::cloudFormation/master.yml
    RoleArn:
    Fn::Sub: arn:aws:iam::${prodAccount}:role/cross-account-role
    RunOrder: '1'
- Name: ExecuteChangeSetProd
    Namespace: prodCloudformationVars
    InputArtifacts:
    - Name: SourceArtifact
    - Name: lambdaDeploymentArt
    ActionTypeId:
    Category: Deploy
    Owner: AWS
    Provider: CloudFormation
    Version: '1'
    Configuration:
    ActionMode: CHANGE_SET_EXECUTE
    ChangeSetName: prod-pipeline-changeset
    ParameterOverrides: |-
        {
        "myBucket" : { "Fn::GetArtifactAtt" : ["lambdaDeploymentArt", "BucketName"]},
        "myLambdaPackage" : { "Fn::GetArtifactAtt" : ["lambdaDeploymentArt", "ObjectKey"]}
        }
    RoleArn:
        Fn::Sub: arn:aws:iam::${prodAccount}:role/prodCfnDeployRole
    StackName: prod-infra
    RoleArn:
    Fn::Sub: arn:aws:iam::${prodAccount}:role/cross-account-role
    RunOrder: '2'
- Name: DeployToProdBucket
    InputArtifacts:
    - Name: websiteArt
    ActionTypeId:
    Category: Deploy
    Owner: AWS
    Provider: S3
    Version: '1'
    Configuration:
    BucketName: "#{prodCloudformationVars.webBucket}"
    Extract: 'true'
    RoleArn:
    Fn::Sub: arn:aws:iam::${prodAccount}:role/cross-account-role
    RunOrder: '3'
```

### Step x - Deploy the Template

Review the test deployment

Approve/Reject based on review

Check the production website

### Step x - Make a Code Change

Step through pipeline

View changes on PROD