<img src="https://i.imgur.com/aDE5ogZ.jpeg" alt="Kevin's Gallery Demo"/>

# Kevin's Photography Gallery Introduction
The goal of this project is to build a robust static website hosting architecture in AWS for a photographer to display their work. Kevin Leger is a professional photographer with a focus on wildlife photography who wants to share what he captures on a website with some light blogging to complement. 

## Architecture
<img src="https://i.imgur.com/HT9Dy1A.jpg" alt="AWS Architecture"/>

1. Amazon Simple Storage Service (Amazon S3) is an object storage service that offers industry-leading scalability, data availability, security, and performance. You can use Amazon S3 to store and retrieve any amount of data at any time, from anywhere.

Buckets are the containers for objects. You can have one or more buckets. For each bucket, you can control access to it (who can create, delete, and list objects in the bucket), view access logs for it and its objects, and choose the geographical region where Amazon S3 will store the bucket and its contents.

2. Amazon CloudFront is a content delivery network (CDN) that accelerates delivery of static and dynamic web content to end users.

CloudFront delivers content through a worldwide network of data centers called edge locations. When an end user requests content that you’re serving with CloudFront, the request is routed to the edge location nearest to the end user with the lowest latency.

3. Amazon Route 53 is a highly available and scalable Domain Name System (DNS) web service. You can use Route 53 to perform three main functions in any combination: domain registration, DNS routing, and health checking.

4. AWS Certificate Manager (ACM) is a service that lets you easily provision, manage, and deploy public and private Secure Sockets Layer/Transport Layer Security (SSL/TLS) certificates for use with AWS services and your internal connected resources. SSL/TLS certificates are used to secure network communications and establish the identity of websites over the Internet as well as resources on private networks.

5. AWS CodePipeline is a continuous delivery service you can use to model, visualize, and automate the steps required to release your software. You can quickly model and configure the different stages of a software release process. CodePipeline automates the steps required to release your software changes continuously.

6. GitHub offers free and paid products for storing and collaborating on code.

7. AWS CloudFormation is a service that helps you model and set up your AWS resources so that you can spend less time managing those resources and more time focusing on your applications that run in AWS. You create a template that describes all the AWS resources that you want (like Amazon EC2 instances or Amazon RDS DB instances), and CloudFormation takes care of provisioning and configuring those resources for you.

### OLD
The website is hosted in an S3 bucket and served using CloudFront as the content delivery network. The website connection is encrypted using a SSL/TLS certificate through AWS Certificate Manager and integrated with the CloudFront distribution. DNS servers are hosted with Route 53 where we purchased a custom domain name and forward traffic to the CloudFront distribution.

GitHub hosts a remote repository for the code and is integrated with CodePipeline to continuously deliver changes to the S3 bucket. Future work is planned to allow an end user to upload photos and change their gallery on the fly.
### OLD


## Quick Start
CloudFormation templates in work.

## Deep Dive
Insert link to deep dive page here.

## Credits
Thanks to [AJ](https://twitter.com/ajlkn) for the website template and [Ram](https://twitter.com/ram__patra) who enhanced it. Check out the website template [here](https://github.com/rampatra/photography).

