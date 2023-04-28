![Under Construction](construction.jpg)

# Deep Dive - Cross-Account CI/CD Pipeline and IaC
This walkthrough will build off of The Basics by creating a pipeline for automated test and production enviorments.

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

## Phase1 - Account Setup

What:
1. AWS management account and IAM user with administrative permissions

2. An organization created in the management account with TOOLS, TEST, and PROD children accounts

3. A Route53 public hosted zone configured for a custom domain name in the PROD account

4. A Route53 public hosted zone configured for a subdomain in the TEST account

Resources:
- [AWS account getting started](https://docs.aws.amazon.com/accounts/latest/reference/welcome-first-time-user.html)

### Step 1 - Creating IAM Roles

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

The policy statmenent needs all permissions required to create the web hosting infrastructure.

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
          "Resource": "arn:aws:kms:us-east-1:MY-TOOLS-ACCOUNT-ID:key/MY-TOOLS-KMSKey-ID",
          "Effect": "Allow"
      }
  ]
}
```