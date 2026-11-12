# portfolio-site

Static portfolio site hosted on a private S3 bucket and served over HTTPS through CloudFront.

## Architecture

Browser -> CloudFront (HTTPS, HTTP redirects to HTTPS) -> OAC -> private S3 bucket

## What it does

- Serves `index.html` from a private S3 bucket (all public access blocked, SSE-S3 encryption on).
- CloudFront is the only reader of the bucket, using Origin Access Control (OAC).
- Viewers get HTTPS. HTTP requests redirect with a 301.

## Setup (done with AWS CLI and console)

1. AWS Budgets alert, MFA on root, IAM admin user, AWS CLI configured.
2. Private S3 bucket in eu-west-1, public access blocked, default encryption on.
3. CloudFront distribution (Free flat-rate plan) with OAC and default root object `index.html`.

## Verification

| Site over HTTPS | `curl.exe -I https://<cloudfront-domain>` | 200 |
| HTTP redirect | `curl.exe -I http://<cloudfront-domain>` | 301 |
| Bucket is private | `curl.exe -I https://<bucket>.s3.eu-west-1.amazonaws.com/index.html` | 403 |

## Cost

Estimated under EUR 1/month. CloudFront Free plan is $0/month. S3 holds about 1 KB.

## Teardown

Disable then delete the CloudFront distribution, delete the OAC, then:

    aws s3 rm s3://<bucket> --recursive
    aws s3api delete-bucket --bucket <bucket> --region eu-west-1
