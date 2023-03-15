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
- writeup phase4
- add logging to cloudfront distribution? might add central logging bucket for all services
- need artifact bucket policy and edit web bucket policy for pipeline?
- verify what `select extract before deploy` does
- add info about linking r53 name servers to external registered domain
- check if IPv6 dns records are obsolete
- add 'why should i' statements for each phase to explain new capabilities gained

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
- Choose the price class that aligns with your budget and expected end user locations
- Set the default root object to `index.html`

![Create Distribution](docs/phase1/createDistribution.jpg)

### S3 Bucket Policy
Edit the bucket policy and define permissions which allow access from a cloudfront origin access control identity.

**NOTE**: Replace any 'my' statements with the actual values specific to your deployment (e.g., `arn:aws:s3:::MY-BUCKET-NAME/* -> arn:aws:s3:::static-website-123/*`)

![Create Bucket Policy](docs/phase1/cloudfrontWebsite.jpg)

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
                    "AWS:SourceArn": "arn:aws:cloudfront::MY-ACCOUNT-ID:distribution/MY-DISTRIBUTION-ID"
                }
            }
        }
    ]
}
```

Now the website can be accessed via HTTPS with the cloudfront distribution domain name which is located on the details section of the distribution and looks like `hj34l2kdfks.cloudfront.net`.

![Cloudfront Website](docs/phase1/bucketPolicy.jpg)

### Phase2

### Prerequisites

1. Completed previous phase(s)

2. A registered custom domain name

3. A Route53 public hosted zone with name servers configured for the domain name

### ACM
Request a public certificate through Certificate Manager with a wildcard subdomain set as the domain name and the root domain as an additional domain. This setup will allow use of this one public certificate with multiple subdomains such as `www.example.com` and `blog.example.com` in addition to the root domain `example.com`.

The wildcard subdomain should read as `*.example.com` where the example.com portion is replaced with your custom domain name.

![Request Certificate](docs/phase3/requestCertificate.jpg)

**Note**: the CNAME data provided by the ACM console is required for creating DNS records in Route53 to issue the certificate.

### Route53
Create DNS entries to route traffic from the custom domain to the cloudfront distribution and validate the ACM certificate.

- A and AAAA alias records for the apex domain routing traffic to the cloudfront distribution domain name
- A and AAAA alias records for the any subdomains routing traffic to the cloudfront distribution domain name
- CNAME record(s) for ACM certificate validation

### CloudFront
Edit the distribution general setting's alternate domain name field adding the custom root domain and subdomains associated with the public ACM certificate. Select the public certificate issued by ACM in the Custom SSL certificate field and save changes.

![Edit Distribution](docs/phase3/editDistribution.jpg)

The website is now accessible from the custom domain via HTTPS.

![Custom Website](docs/phase3/customDomain.jpg)


### Phase3

### Prerequisites

1. Completed phase1

2. Github account

3. A repository on the Github account storing the website files that were previously stored in the website S3 bucket

### S3 Bucket
Create a new s3 bucket with default settings to contain the artifacts created by CodePipeline.

![Create Bucket](docs/phase1/createBucket.jpg)

### CodePipeline
First, create a new IAM policy defining permissions for this use case keeping least priviledge in mind. 

```
{
"Version": "2012-10-17",
    "Statement": [
        {
            "Action": [
                "codestar-connections:UseConnection"
            ],
            "Resource": "arn:aws:codestar-connections:us-east-1:MY-ACCOUNT-ID:connection/MY-CONNECTION-ID",
            "Effect": "Allow"
        },
        {
            "Action": [
                "s3:PutObject"
            ],
            "Resource": [
                "arn:aws:s3:::MY-WEBSITE-BUCKET-NAME/*",
                "arn:aws:s3:::MY-ARTIFACT-BUCKET-NAME/*"
            ],
            "Effect": "Allow"
        },
        {
            "Action": [
                "s3:GetObject"
            ],
            "Resource": "arn:aws:s3:::MY-ARTIFACT-BUCKET-NAME/*",
            "Effect": "Allow"
        }
    ]
}
```
Next, create a role defining CodePipeline as the trusted entity and attach the policy to this role.

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

Create a new pipeline selecting the artifact bucket for the artifact store (under advanced settings) and select the new role to provide permissions for this pipeline to funcion properly.

![Create Pipeline](docs/phase2/createPipeline.jpg)

Source Stage - Set the source as GitHub (Version 2) which will require integrating AWS connector as a GitHub app with your GitHub account/repository. This is a simple process that the setup wizard will walk you through to authorize aws connector. After the connection is made, select your repository name and branch name where the website files are stored while keeping other settings as default in the source stage settings.

![Create Connection](docs/phase2/createConnection.jpg)

Build Stage - Skip. 

Deploy Stage - Select S3 for the deploy stage provider and the website bucket for the deploy location. Select extract before deploy and keep other settings as default.

Now changes made in the GitHub repository will flow to the website S3 Bucket automatically.

![Successful Pipeline](docs/phase2/pipeline.jpg)

### Phase4

### Prerequisites

TBD


## Credits
Thanks to [AJ](https://twitter.com/ajlkn) for the website template and [Ram](https://twitter.com/ram__patra) who enhanced it. Check out the website template on GitHub [here](https://github.com/rampatra/photography).