<img src="https://i.imgur.com/aDE5ogZ.jpeg" alt="Kevin's Gallery Demo"/>

# Kevin's Photography Gallery Introduction
The goal of this project is to build a robust static website hosting architecture in AWS for a photographer to display their work. Kevin Leger is a professional photographer with a focus on wildlife photography who wants to share what he captures on a website with some light blogging to complement. 

## Architecture
<img src="https://i.imgur.com/gJsyeQ2.png" alt="AWS Architecture"/>

The website is hosted in an S3 bucket and served using CloudFront as the content delivery network. The website connection is encrypted using a SSL/TLS certificate through AWS Certificate Manager and integrated with the CloudFront distribution. DNS servers are hosted with Route 53 where we purchased a custom domain name and forward traffic to the CloudFront distribution.

GitHub hosts a remote repository for the code and is integrated with CodePipeline to continuously deliver changes to the S3 bucket. Future work is planned to allow an end user to upload photos and change their gallery on the fly.

## Quick Start
CloudFormation templates in work.

## Credits
Thanks to [AJ](https://twitter.com/ajlkn) for the website template and [Ram](https://twitter.com/ram__patra) who enhanced it. Check out the website template [here](https://github.com/rampatra/photography).

