![Kevin's Website](docs/kevinlegerwebsite.jpg)

# AWS Static Website Introduction - ft. Kevin's Photography Gallery 
The goal of this project is to build a robust static website hosting architecture in AWS focusing on reliability, security, and automation. Kevin Leger is a professional photographer with a focus on wildlife photography who wants to share what he captures with the world. Please visit [kevinleger.com](https://kevinleger.com) to check out this project live.

## Architecture
![Website Architecture](docs/Architecture.jpg)

### Architecture Descriptions ([link to AWS docs](https://docs.aws.amazon.com/))

1. **Simple Storage Service (S3)** is an object storage service that offers industry-leading scalability, data availability, security, and performance. You can use Amazon S3 to store and retrieve any amount of data at any time, from anywhere.

2. **CloudFront** is a content delivery network (CDN) that accelerates delivery of static and dynamic web content to end users.

3. **Route 53** is a highly available and scalable Domain Name System (DNS) web service. You can use Route 53 to perform three main functions in any combination: domain registration, DNS routing, and health checking.

4. **Certificate Manager (ACM)** is a service that lets you easily provision, manage, and deploy public and private Secure Sockets Layer/Transport Layer Security (SSL/TLS) certificates for use with AWS services and your internal connected resources.

5. **CodePipeline** is a continuous delivery service you can use to model, visualize, and automate the steps required to release your software.

6. **GitHub** is a platform offering free and paid products for storing and collaborating on code.

7. **API Gateway** is a fully managed service that makes it easy for developers to create, publish, maintain, monitor, and secure APIs at any scale. 

8. **Lambda** is a serverless, event-driven compute service that lets you run code for virtually any type of application or backend service without provisioning or managing servers.

9. **Simple Email Service (SES)** is a cloud email service provider that can integrate into any application for bulk email sending. 

10. **CloudFormation** is a service that helps you model and set up your AWS resources so that you can spend less time managing those resources and more time focusing on your applications that run in AWS.

11. **Serverless Image Processing** is an event driven architecture written in python to transform uploaded images into thumbnails (using machine learning service Rekognition) and fullsize images with watermarks. Check out the project [here](https://github.com/levicherrin/aws-serverless-image-processing).

## Quick Deploy
If you're already familiar with AWS and the services used in this project then the fastest way to get up and running is to deploy one of three cloudformation templates.

Phase 1 requires a static website `index.html` file.

Phase 2 requires a github repository in addition to previous phase requirements.

Phase 3 requires a registered custom domain name and a Route53 public hosted zone in addition to previous phase requirements.

## Implementation Deep Dive

TODOS:
- add logging to cloudfront distribution? might add central logging bucket for all services
- need to narrow codepipeline policy permissions

### Phase1

### Prerequisites

1. 'index.html' document and any supporting assets for your website

### S3 Bucket
Create a bucket in S3 with all default settings applied and upload the website files.

![Create Bucket](docs/phase1/createBucket.jpg)

### CloudFront
Create a cloudfront distribution and set the distribution origin domain to point to the S3 bucket.
- Add an origin Access Control Identity (OAC) using the distribution wizard
- Set the viewer protocol to `redirect http to https`
- Choose the price class that aligns with expected end user locations
- Set the default root object to `index.html`

![Create Distribution](docs/phase1/createDistribution.jpg)

### S3 Bucket Policy
Create a resouce policy for the bucket which allows access from a cloudfront origin access control identity.

![Create Bucket Policy](docs/phase1/bucketPolicy.jpg)

```
{
    "Version": "2012-10-17",
    "Statement": [
        {
            "Effect": "Allow",
            "Principal": {
                "Service": "cloudfront.amazonaws.com"
            },
            "Action": "s3:GetObject",
            "Resource": "arn:aws:s3:::MY-BUCKET-NAME/*"
            },
            "Condition": {
                "StringEquals": {
                    "AWS:SourceArn": "arn:aws:cloudfront::MY-ACCOUNT-ID:distribution/MY-DISTRIBUTION-ID
                }
            }
        }
    ]
}
```

Now the website can be accessed via the cloudfront distribution domain name which is located on the details section of the distribution and looks something like `hj34l2kdfks.cloudfront.net`.

### Phase2

### Prerequisites

1. Completed previous phase(s).

2. Github account and repository storing the 'index.html' file and other supporting assets

### S3 Bucket
Create a new s3 bucket that will contain the artifacts created by CodePipeline.

### CodePipeline
First create a new IAM policy and role for CodePipeline to use.

```
{
"Version": "2012-10-17",
    "Statement": [
        {
            "Effect": "Allow",
            "Action": "*",
            "Resource": "*"
        }
    ]
}
```
Role
```
{
"Version": "2012-10-17",
    "Statement": [
        {
            "Effect": "Allow",
            "Principal": {
                "Service": [
                    "codepipeline.amazonaws.com"
                ]
            },
            "Action": [
                "sts:AssumeRole"
            ]
        }
    ]
}
```

Now create a new pipeline with the source as GitHub. This will require creating a github connection linked to your GitHub account/repository. Be sure to select the new bucket for the artifact store and also the new role to set permissions for this pipeline. Add a deploy stage and select the website bucket created previously.

### CloudFront
No changes required.

### Phase3

### Prerequisites

1. Completed previous phase(s).

2. A registered custom domain name and a Route53 public hosted zone with name servers configured for the domain name.

### ACM
Create a certificate with a wildcard as the domain name and set the alternative domain as the apex domain.

### Route53
Create DNS entries to validate the ACM certificate and route traffic from the custom domain to the cloudfront distribution.

### CloudFront
Add the custom domain and subdomains to aliases. Change the certificate to the new ACM certificate for the domain.

## Credits
Thanks to [AJ](https://twitter.com/ajlkn) for the website template and [Ram](https://twitter.com/ram__patra) who enhanced it. Check out the website template on GitHub [here](https://github.com/rampatra/photography).