# Phase3

## Prerequisites

1. Previous phase requirements.

2. A registered custom domain name and a Route53 public hosted zone with name servers configured for the domain name.

### ACM
Create a certificate with a wildcard as the domain name and set the alternative domain as the apex domain.

### Route53
Create DNS entries to validate the ACM certificate and route traffic from the custom domain to the cloudfront distribution.

### CloudFront
Add the custom domain and subdomains to aliases. Change the certificate to the new ACM certificate for the domain.