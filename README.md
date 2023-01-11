<img src="https://i.imgur.com/aDE5ogZ.jpeg" alt="Kevin's Gallery Demo"/>

# AWS Static Website Introduction - ft. Kevin's Photography Gallery 
The goal of this project is to build a robust static website hosting architecture in AWS focusing on reliability, security, and automation. Kevin Leger is a professional photographer with a focus on wildlife photography who wants to share what he captures with the world. Please visit [kevinleger.com](https://kevinleger.com) to check out this project live.

## Quick Deploy
If you're already familiar with AWS and the services used in this project then the fastest way to get up and running is to deploy one of three cloudformation templates.

Phase 1 requires a static website `index.html` file.

Phase 2 requires a github repository in addition to previous phase requirements.

Phase 3 requires a registered custom domain name and a Route53 public hosted zone in addition to previous phase requirements.

## Implementation Deep Dive
To get started please review [phase 1](docs/phase1.md).

## Architecture
![Website Architecture](docs/Architecture.jpg)

### Service Descriptions

1. **Simple Storage Service (S3)** is an object storage service that offers industry-leading scalability, data availability, security, and performance. You can use Amazon S3 to store and retrieve any amount of data at any time, from anywhere.

    Buckets are the containers for objects. You can have one or more buckets. For each bucket, you can control access to it (who can create, delete, and list objects in the bucket), view access logs for it and its objects, and choose the geographical region where Amazon S3 will store the bucket and its contents.

2. **Amazon CloudFront** is a content delivery network (CDN) that accelerates delivery of static and dynamic web content to end users.

    CloudFront delivers content through a worldwide network of data centers called edge locations. When an end user requests content that you’re serving with CloudFront, the request is routed to the edge location nearest to the end user with the lowest latency.

3. **Amazon Route 53** is a highly available and scalable Domain Name System (DNS) web service. You can use Route 53 to perform three main functions in any combination: domain registration, DNS routing, and health checking.

4. **AWS Certificate Manager (ACM)** is a service that lets you easily provision, manage, and deploy public and private Secure Sockets Layer/Transport Layer Security (SSL/TLS) certificates for use with AWS services and your internal connected resources. SSL/TLS certificates are used to secure network communications and establish the identity of websites over the Internet as well as resources on private networks.

5. **AWS CodePipeline** is a continuous delivery service you can use to model, visualize, and automate the steps required to release your software. You can quickly model and configure the different stages of a software release process. CodePipeline automates the steps required to release your software changes continuously.

6. **GitHub** is a platform offering free and paid products for storing and collaborating on code.

7. **API Gateway** is a fully managed service that makes it easy for developers to create, publish, maintain, monitor, and secure APIs at any scale. APIs act as the "front door" for applications to access data, business logic, or functionality from your backend services. 

8. **Lambda** is a serverless, event-driven compute service that lets you run code for virtually any type of application or backend service without provisioning or managing servers. You can trigger Lambda from over 200 AWS services and software as a service (SaaS) applications, and only pay for what you use.

9. **Simple Email Service (SES)** is a cloud email service provider that can integrate into any application for bulk email sending. Whether you send transactional or marketing emails, you pay only for what you use. Amazon SES also supports a variety of deployments including dedicated, shared, or owned IP addresses.

10. **AWS CloudFormation** is a service that helps you model and set up your AWS resources so that you can spend less time managing those resources and more time focusing on your applications that run in AWS. You create a template that describes all the AWS resources that you want (like Amazon EC2 instances or Amazon RDS DB instances), and CloudFormation takes care of provisioning and configuring those resources for you.

## Credits
Thanks to [AJ](https://twitter.com/ajlkn) for the website template and [Ram](https://twitter.com/ram__patra) who enhanced it. Check out the website template on GitHub [here](https://github.com/rampatra/photography).