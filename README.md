# Sahana's Cloud Custodian Policies 

## Policies in Production

| Policy | Description |
|--------|-------------|
| [mailer.yml](https://github.com/sahanasj/cloudcustodian-policies/blob/master/mailer.yml)<br> | Sends email notification via Simple Email Service (SES) using notify action |
| [s3-bucket-versioning.yml](https://github.com/sahanasj/cloudcustodian-policies/blob/master/s3-bucket-versioning.yml)<br> | Rectifies and enables all suspended versioning on S3 buckets, then sends notifications.  |
|[s3-bucket-public-access.yml](https://github.com/sahanasj/cloudcustodian-policies/blob/master/s3-bucket-public-access.yml)<br>| Rectifies and corrects the Global grants and secured S3 buckets as private, then sends notifications.  |
| [s3-toggle-logging.yml](https://github.com/sahanasj/cloudcustodian-policies/blob/master/s3-toggle-logging.yml)<br> | Configure New Buckets Settings and Standards such as enabling the default S3 AES256 bucket encryption, turns on object versioning, enables logging on the bucket, and tags the user that created the bucket.  |
