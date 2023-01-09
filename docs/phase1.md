# Phase1

## Prerequisites

1. 'index.html' document and any supporting assets for your website

### S3 Bucket
Create a bucket in S3. Disable the block public access ACLs. Enable website hosting and set 'index document' to the website 'index.html' file.

Create a resouce policy which allows access from the cloudfront origin access control identity.
```
{
    "Version": "2012-10-17",
    "Statement": [
        {
            "Action": [
                "s3:GetObject"
            ],
            "Effect": "Allow",
            "Resource":
            {
                "Fn::Sub": "arn:aws:s3:::${S3Bucket}/*"
            },
            "Principal": {
                "Service": "cloudfront.amazonaws.com"
            },
            "Condition": {
                "StringEquals": {
                    "AWS:SourceArn": {"Fn::Sub": "arn:aws:cloudfront::${AWS::AccountId}:distribution/${cloudFrontDistribution}"}
                }
            }
        }
    ]
}
```
### CloudFront
Create a cloudfront distribution with a custom origin pointing to the domain name of the S3 bucket. Set the viewer protocol to 'redirect to https' and the default root object to index.html.

Also  create an origina access control identity which will allow read only access to the S3 bucket.