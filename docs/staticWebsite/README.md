#   Static Website Hosting with Amazon S3

This guide details the steps for hosting a static website (HTML, CSS, JavaScript files) using Amazon Simple Storage Service (S3). It also explores the use of Amazon CloudFront as a Content Delivery Network (CDN) to improve performance and security, and Amazon Route 53 for managing the domain name.

> [!NOTE]
> Throughout this guide, various AWS services are utilized.
> 
> To avoid incurring costs beyond the free tier, ensure all created resources are deleted upon completion of the project.

##  Phases

This walkthrough is divided into the following phases:

- [Phase 1 - Getting up and Running](#phase-1---getting-up-and-running): Quickly deploy a website with an unencrypted connection via HTTP.
- [Phase 2 - Encryption and Caching](#phase-2---encryption-and-caching): Secure the website with HTTPS using CloudFront and improve performance with caching.
- [Phase 3 - Custom Domain Name](#phase-3---custom-domain-name): Associate a custom domain name with the website.

Begin with the first phase.

### Phase 1 - Getting up and Running

**Goal:** Quickly deploy a website with an unencrypted connection via HTTP.

This phase explains how to create a simple website and make it accessible on the internet.

**What is needed:**

1.  An AWS account with administrator access.
2.  The HTML, CSS, and JavaScript files for the website (including an index document, typically named `index.html`).

**Resources:**

- [Free website templates](https://html5up.net/)
- [AWS account getting started](https://docs.aws.amazon.com/accounts/latest/reference/welcome-first-time-user.html)
- [AWS management console](https://docs.aws.amazon.com/awsconsolehelpdocs/latest/gsg/learn-whats-new.html)
- [S3 Documentation](https://docs.aws.amazon.com/AmazonS3/latest/userguide/Welcome.html)
- [Configuring an index document](https://docs.aws.amazon.com/AmazonS3/latest/userguide/IndexDocumentSupport.html)

**Steps:**

1.  **Create a new bucket** in the S3 console, accepting the default settings.

    ![Create Bucket](phase1/createBucket.jpg)

2.  **Upload the website files** (HTML, CSS, JavaScript, images) to the bucket.

3.  **Adjust the permissions** to allow public access to the files in the bucket. Navigate to the "Permissions" tab of the bucket and disable "Block all public access."

    ![Unblock Access](phase1/blockPublicAccess.jpg)

> [!WARNING]
> Making the bucket publicly accessible has security implications. Ensure the risks are understood before proceeding.

4.  **Edit the bucket policy** to allow any principal (*) to read objects from the bucket using the S3 `GetObject` action. Replace `MY-BUCKET-NAME` with the actual name of the bucket.

    ```json
    {
        "Version": "2012-10-17",
        "Statement": [
            {
                "Sid": "PublicReadGetObject",
                "Effect": "Allow",
                "Principal": "*",
                "Action": "s3:GetObject",
                "Resource": "arn:aws:s3:::MY-BUCKET-NAME/*"
            }
        ]
    }
    ```

5.  **Enable static website hosting** in the properties tab. Specify the name of the website's index document, commonly `index.html`.

6.  **Click on the bucket website endpoint** to access the site via HTTP in a new tab.

    ![Website](phase1/bucketWebsite.jpg)

### Phase 2 - Encryption and Caching

**Goal:** Secure the website with HTTPS and improve performance using CloudFront.

This phase explains how to configure CloudFront to deliver the website. CloudFront is a Content Delivery Network (CDN) that speeds up content delivery and provides HTTPS encryption.

**What is needed:**

1.  A completed Phase 1 setup.
2.  An understanding of the REST API endpoint (explained below).

**Resources:**

- [CloudFront docs](https://docs.aws.amazon.com/AmazonCloudFront/latest/DeveloperGuide/Introduction.html)
- [S3 endpoint differences](https://docs.aws.amazon.com/AmazonS3/latest/userguide/WebsiteEndpoints.html#WebsiteRestEndpointDiff)

**Steps:**

1.  **Disable static website hosting** on the S3 bucket.

> [!NOTE]
> Static website hosting is disabled because CloudFront will access the bucket using its REST API endpoint, which allows more control over content access. The REST API endpoint is different from the static website hosting endpoint.

2.  **Create a new CloudFront distribution.** Configure the distribution with the following:

    - **Origin domain:** Select the **REST API endpoint** of the S3 bucket as the origin.
    - **Origin Access Control (OAC):** Add a new OAC identity using the distribution wizard. Choose to manually add permissions to the S3 bucket.
    - **Viewer protocol policy:** Set this to "Redirect HTTP to HTTPS" for secure access.
    - **Price class:** Choose a price class aligned with the budget and expected user locations.
    - **Default root object:** Enter the website's index document (e.g., `index.html`).

    ![Create Distribution](phase2/createDistribution.jpg)

3.  **Update S3 bucket permissions.**

    - Navigate to the "Permissions" tab of the S3 bucket.
    - **Enable "Block all public access."**
    - **Edit the bucket policy** to grant CloudFront's OAC identity access to retrieve objects. Replace `MY-BUCKET-NAME`, `MY-ACCOUNT-ID`, and `MY-DISTRIBUTION-ID` with the specific values.

        ```json
        {
            "Version": "2012-10-17",
            "Statement": [
                {
                    "Effect": "Allow",
                    "Principal": {
                        "Service": "cloudfront.amazonaws.com"
                    },
                    "Action": "s3:GetObject",
                    "Resource": "arn:aws:s3:::MY-BUCKET-NAME/*",
                    "Condition": {
                        "StringEquals": {
                            "AWS:SourceArn": "arn:aws:cloudfront::MY-ACCOUNT-ID:distribution/MY-DISTRIBUTION-ID"
                        }
                    }
                }
            ]
        }
        ```

4.  **Access the website.**

    - The website is now accessible securely via HTTPS using the CloudFront distribution's domain name (e.g., `hj34l2kdfks.cloudfront.net`), found in the distribution's details.

    ![Cloudfront Website](phase2/cloudfrontWebsite.jpg)

### Phase 3 - Custom Domain Name

**Goal:** Enable access to the website using a custom domain name (e.g., `www.example.com`).

This phase explains how to associate a registered domain name with the CloudFront distribution, providing users with more intuitive URLs and leveraging the capabilities of Route 53.

**What is needed:**

1.  A completed Phase 2 setup.
2.  A registered custom domain name (e.g., `example.com`).
3.  A Route 53 public hosted zone configured for the custom domain name.

**Resources:**

- [Registering and managing domains](https://docs.aws.amazon.com/Route53/latest/DeveloperGuide/registrar.html)
- [AWS Certificate Manager Documentation](https://docs.aws.amazon.com/acm/latest/userguide/acm-overview.html)
- [Route 53 Documentation](https://docs.aws.amazon.com/Route53/latest/DeveloperGuide/Welcome.html)
- [CloudFront Documentation](https://docs.aws.amazon.com/AmazonCloudFront/latest/DeveloperGuide/Introduction.html)

**Steps:**

1.  **Request a new public SSL/TLS certificate** using AWS Certificate Manager (ACM).

    - Navigate to the AWS Certificate Manager console.
    - Choose **Request a certificate**.
    - Select **Request a public certificate**.
    - In the **Domain name** field, enter a wildcard subdomain (e.g., `*.example.com`).
    - Choose **Add another domain name** and enter the root domain (e.g., `example.com`).

> [!NOTE]
> Requesting a certificate with a wildcard subdomain and the root domain allows the same certificate to secure `www.example.com`, `blog.example.com`, and `example.com`.

    - Keep all other settings at their default values and choose **Request**.

        ![Request Certificate](phase3/requestCertificate.jpg)

2.  **Validate the ACM certificate request.**

    - Navigate to the certificate details page in the ACM console. The issuing status will be **Pending validation**.
    - Note down the **CNAME name** and **CNAME value** data provided for each domain listed. These values will be used to create DNS records to validate the certificate.

3.  **Create DNS records to validate the ACM certificate** in the Route 53 hosted zone for the domain.

    - For each domain name listed on the ACM certificate, create a **CNAME** record.
    - Use the **CNAME name** and **CNAME value** obtained from the ACM console in the previous step for each record.

4.  **Create DNS records to route traffic to the CloudFront distribution.**

    - In the Route 53 hosted zone, create the following records:
        - An **A** alias record for the root domain (`example.com`) pointing to the CloudFront distribution domain name.
        - An **AAAA** alias record for the root domain (`example.com`) pointing to the CloudFront distribution domain name (if IPv6 is enabled for the CloudFront distribution).
        - Repeat the above steps to create **A** and **AAAA** alias records for any subdomains (e.g., `www.example.com`) pointing to the same CloudFront distribution.

5.  **Update the CloudFront distribution settings.**

    - Navigate to the CloudFront console and select the distribution created in Phase 2.
    - Choose **Edit** under the **General** tab.
    - In the **Alternate domain name (CNAMEs) - optional** field, add the root domain (e.g., `example.com`).
    - Repeat the step above for any subdomains (e.g., `www.example.com`).
    - In the **Custom SSL certificate** section, choose the public certificate issued by ACM in Step 1.

        ![Edit Distribution](phase3/editDistribution.jpg)

The website is now accessible via HTTPS using the custom domain name (e.g., `https://www.example.com`).

![Custom Website](phase3/customDomain.jpg)