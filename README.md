<img src="https://i.imgur.com/aDE5ogZ.jpeg" alt="Kevin's Gallery Demo"/>

# AWS Static Website Introduction - ft. Kevin's Photography Gallery 
The goal of this project is to build a robust static website hosting architecture in AWS focusing on reliability, security, and automation. Kevin Leger is a professional photographer with a focus on wildlife photography who wants to share what he captures with the world. Please visit [kevinleger.com](https://kevinleger.com) to check out this project live.

## Quick Deploy
If you're already familiar with AWS and the services used in this project then the fastest way to get up and running is to deploy one of three cloudformation templates.

Phase 1 requires a static website `index.html` file.

Phase 2 requires a github repository in addition to previous phase requirements.

Phase 3 requires a registered custom domain name and a Route53 public hosted zone in addition to previous phase requirements.

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

## Implementation Deep Dive
To get started please review [phase 1](docs/phase1.md).

## Credits
Thanks to [AJ](https://twitter.com/ajlkn) for the website template and [Ram](https://twitter.com/ram__patra) who enhanced it. Check out the website template on GitHub [here](https://github.com/rampatra/photography).