# Phase2

## Prerequisites

1. Previous phase requirements.

2. Github account and repo storing the 'index.html' file and other supporting assets

### S3 Bucket
Create another s3 bucket that will contain the artifacts created by CodePipeline.

### CodePipeline
First create a new IAM policy and role for CodePipeline to use.

```
Policy *need to narrow permissions*
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

Role
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

Now create a new pipeline with the source as GitHub. This will require creating a github connection linked to your GitHub account/repository. Be sure to select the bucket created earlier for the artifact store and also the role just created to set permissions. Add a deploy stage and select the website bucket created in phase 1.
