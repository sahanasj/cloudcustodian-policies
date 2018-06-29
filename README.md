# Sahana's Cloud Custodian Policies 

## Policies in Production

| Policy | Description |
|--------|-------------|
| [mailer.yml](https://github.com/sahanasj/cloudcustodian-policies/blob/master/mailer.yml)<br> | Sends email notification via Simple Email Service (SES) using notify action |
| [s3-bucket-versioning.yml](https://github.com/sahanasj/cloudcustodian-policies/blob/master/s3-bucket-versioning.yml)<br> | Rectifies and enables all suspended versioning on S3 buckets, then sends notifications.  |
|[s3-bucket-public-access.yml](https://github.com/sahanasj/cloudcustodian-policies/blob/master/s3-bucket-public-access.yml)<br>| Rectifies and corrects the Global grants and secured S3 buckets as private, then sends notifications.  |
| [s3-toggle-logging.yml](https://github.com/sahanasj/cloudcustodian-policies/blob/master/s3-toggle-logging.yml)<br> | Configure New Buckets Settings and Standards such as enabling the default S3 AES256 bucket encryption, turns on object versioning, enables logging on the bucket, and tags the user that created the bucket.  |

## Cloud Custodian Architecture and AWS Services

<!--![alt text](https://github.com/sahanasj/cloudcustodian-policies/blob/master/Custodian-Architecture.PNG)-->
<img src="https://github.com/sahanasj/cloudcustodian-policies/blob/master/Custodian-Architecture.PNG" width="950">

Cloud Custodian (a.k.a C7N) notifies users in real-time AWS resources behavior changes, Compliance (Security/Access Control, Encryption, Backups, etc) and drives Cost savings (Off-hours, Monitoring and Garbage Collection of unused and underutilized resources).

# Getting Started
<details>
<summary>Quick Install</summary>

```
*** Install dependencies (with virtualenv) ***
$ sudo apt-get -y install virtualenv or sudo yum install virtualenv
$ virtualenv custodian_env
$ source custodian_env/bin/activate

*** Install AWS CLI and C7N ***
$ pip install awscli c7n

** Configure AWSCLI **
$ aws configure
(Configure with AWS Credentials and Region)

*** Verify AWSCLI Installation with any CLI command ***
$ aws ec2 describe-regions

*** To Install Cloud Custodian Mailer ***
*** Install repository***
$ git clone https://github.com/capitalone/cloud-custodian
$ cd cloud-custodian/tools/c7n_mailer
$ pip install -r requirements.txt
$ python setup.py develop

*** Verify Installation ***
$ c7n-mailer
$ custodian
```
For more info, check out [Cloud Custodian in GitHub](https://github.com/capitalone/cloud-custodian)
</details>


# Usage
<details>
<summary>Getting Started</summary>

<pre>
Cloud Custodian must be run within a virtual environment.

$ cd ~
$ source custodian_env/bin/activate
$ cd cloudcustodian_scripts  (this is the folder where all the custodian policies reside)

** Execute/run the Cloud Custodian Policies **

# Validate the configuration
$ custodian validate s3-bucket-public-access.yml

# Dryrun the policies 
$ custodian run --dryrun -s check-public-access s3-bucket-public-access.yml
(Note: Make sure If you get a match (e.g. count > 0), then run the below command)

# Run the policy 
$ custodian run -s check-public-access s3-bucket-public-access.yml

** Invoking c7n Mailer **
# Validate the configuration
$ custodian validate s3-bucket-public-access.yml

# Dryrun the policies 
$ custodian run --dryrun -s check-public-access s3-bucket-public-access.yml
(Note: Make sure If you get a match (e.g. count > 0), then run the below command)

# Run the policy to invoke custodian mailer
$ c7n-mailer --config mailer.yml --update-lambda && custodian run -c s3-bucket-public-access.yml -s .

When we run this policy, Check the AWS console for a new Lambda named `cloud-custodian-mailer`. 
The mailer runs every five minutes, so wait a bit and then look for an email in your inbox. (Orelse manually, edit CWE scheduled time less than 5 mins for the quick response)

<img src="https://github.com/sahanasj/cloudcustodian-policies/blob/master/Custodian-mailer-lambda-function.PNG" width="850">

 Cloud Custodian will create a log files in the ~/cloudcustodian_scripts/check-public-access/ subdirectory IF there are any matches. 
</pre>
</details>

# Environment Settings
<details>
<summary>mailer.yml</summary>
<pre>

#Which queue should we listen to for messages
queue_url: https://sqs.us-east-1.amazonaws.com/930337447539/c7n_mailer_for_s3_events

#Standard Lambda Function Config
region: us-east-1
role: arn:aws:iam::930337447539:role/lambda-s3-governance

#Default from address
from_address: sjayaramu@eplus.com
</pre>
</details>

<details>
<summary>Cloud Custodian Lambda AWS Role</summary>
<pre>

Note: Based on your use case, additional permissions may be needed. 
Cloud Custodian will generate a msg if that is the case after invocation.
AWS IAM Role & policies plays an important role to allows Lambda functions to call AWS services. (Make a note of IAM ARN ex: arn:aws:iam::930337447539:role/S3-GovernanceForLincoln)

Trust relationship:
"Service": "lambda.amazonaws.com"

<img src="https://github.com/sahanasj/cloudcustodian-policies/blob/master/IAM-Trust-relationship.PNG" width="550">


Reference: 
| [AWSS3CustomPolicyForLincoln.json](https://github.com/sahanasj/cloudcustodian-policies/blob/master/AWSS3CustomPolicyForLincoln.json)<br> | A policy defines the AWS permissions that you can assign to a user, group, or role.  |
</pre>
</details>

## Schemas Used

<details>
<summary>s3</summary>
<pre>

(custodian_env) [root@localhost custodian_scripts]# custodian schema s3
aws.s3:
  actions: [attach-encrypt, auto-tag-user, configure-lifecycle, delete, delete-bucket-notification,
    delete-global-grants, encrypt-keys, encryption-policy, invoke-lambda, mark-for-op,
    no-op, notify, put-metric, remove-statements, remove-website-hosting, set-bucket-encryption,
    set-inventory, set-statements, tag, toggle-logging, toggle-versioning, unmark]
  filters: [and, bucket-encryption, bucket-notification, cross-account, data-events,
    event, global-grants, has-statement, inventory, is-log-target, marked-for-op,
    metrics, missing-policy-statement, missing-statement, no-encryption-statement,
    not, or, value]
    
[ OR ]

** For S3 Schema Filters **

(custodian_env) [root@localhost custodian_scripts]# custodian schema s3.filters
aws.s3:
  filters: [and, bucket-encryption, bucket-notification, cross-account, data-events,
    event, global-grants, has-statement, inventory, is-log-target, marked-for-op,
    metrics, missing-policy-statement, missing-statement, no-encryption-statement,
    not, or, value]
    
** For S3 Schema actions **

(custodian_env) [root@localhost lfg-custodian]# custodian schema s3.actions
aws.s3:
  actions: [attach-encrypt, auto-tag-user, configure-lifecycle, delete, delete-bucket-notification,
    delete-global-grants, encrypt-keys, encryption-policy, invoke-lambda, mark-for-op,
    no-op, notify, put-metric, remove-statements, remove-website-hosting, set-bucket-encryption,
    set-inventory, set-statements, tag, toggle-logging, toggle-versioning, unmark]

** To undesrtand a particular filter & action: **

(custodian_env) [root@localhost custodian_scripts]# custodian schema s3.filters.global-grants
Help
----

Filters for all S3 buckets that have global-grants

:example:
.. code-block:: yaml

        policies:
          - name: s3-delete-global-grants
            resource: s3
            filters:
              - type: global-grants
            actions:
              - delete-global-grants
Schema
------

{
    "additionalProperties": false,
    "required": [
        "type"
    ],
    "type": "object",
    "properties": {
        "allow_website": {
            "type": "boolean"
        },
        "operator": {
            "enum": [
                "or",
                "and"
            ],
            "type": "string"
        },
        "type": {
            "enum": [
                "global-grants"
            ]
        },
        "permissions": {
            "items": {
                "enum": [
                    "READ",
                    "WRITE",
                    "WRITE_ACP",
                    "READ",
                    "READ_ACP"
                ],
                "type": "string"
            },
            "type": "array"
        }
    }
}

</pre>
</details>

## Troubleshooting Tips

<details>
<pre>

Use 'custodian validate' to find syntax errors
Check 'name' of policy doesn't contain spaces
Check SQS to see if Custodian payload is entering the queue
Check cloud-custodian-mailer lambda CloudWatch rule schedule (5 minute by default)
Check Lambda error logs (this requires CloudWatch logging)
Check role for lambda(s) have adequate permissions
Remember to update the cloud-custodian-mailer lambda when making changes to a policy that uses notifications
Clear the cache if you encounter errors due to stale information (**rm ~/.cache/cloud-custodian.cache**)

</pre>
</details>

## Lambda Code Cheatsheet

```
mode:
  type: cloudtrail
  role: arn:aws:iam::930337447539:role/S3-GovernanceForLincoln
  events:
    - CreateBucket
```

```
mode:
  type: periodic
  role: arn:aws:iam::930337447539:role/S3-GovernanceForLincoln
  schedule: "rate(15 minutes)"
```

```
mode:
  type: periodic
  role: arn:aws:iam::930337447539:role/S3-GovernanceForLincoln
  schedule: 'cron(0/2 * * * ? *)'
```
</details>

<details>
<summary>Sending Notifications via SES</summary>
  
```
actions:
 - type: notify
   template: default.html
   template_format: 'html'
   priority_header: '5'
   subject: "ALERT! - S3 : Invalid Global ACL on Bucket [AWS Account: {{ account }} - Region: {{ region }}]"
   comments: "Violation of S3 policy"
   violation_desc: <Message_Of_Mail_Body>
   action_desc: "Actions Taken: Corrects the ACLs/Policy and Notify User"
   to:
     - <your-email-address-goes-here>
   owner_absent_contact:
     - <your-emails-address-goes-here>
   transport:
     type: sqs
     queue: https://sqs.us-east-1.amazonaws.com/930337447539/c7n_mailer_for_s3_events
```
</details>

Reference: 
<a href="https://docs.aws.amazon.com/AmazonCloudWatch/latest/events/ScheduledEvents.html/">Schedule Expressions for Rules</a> <br>
Useful Tool: <a href="https://crontab.guru/">Quick simple editor for cron schedule expressions.</a>

**Note** <br>
**Config**:	May run in a different region but not cross-account <br>
**Event**:	Only run in the same region and account <br>
**Periodic**:	May run in a different region and different account <br>

# Cloud Custodian Important Resources
[Cloud Custodian - All Resources](http://capitalone.github.io/cloud-custodian/)<br>
[Cloud Custodian - Getting Started](http://capitalone.github.io/cloud-custodian/docs/quickstart/index.html)<br>
[Cloud Custodian - Github](https://github.com/capitalone/cloud-custodian)<br>
[Cloud Custodian - Docs](http://capitalone.github.io/cloud-custodian/docs/index.html)<br>
[Cloud Custodian - 400+ actions and 300+ filters to build policies with](https://pypi.org/project/c7n/)<br>
[Cloud Custodian - Features](https://developer.capitalone.com/opensource-projects/cloud-custodian/)<br>
[Cloud Custodian - S3 Module](http://capitalone.github.io/cloud-custodian/docs/generated/aws/c7n.resources.html#module-c7n.resources.s3)<br>
[Blog - Using Cloud Custodian for Cloud Governance in AWS](https://www.linkedin.com/pulse/using-cloud-custodian-governance-aws-tejaswi-konduri)<br>
[Lambda Support](http://capitalone.github.io/cloud-custodian/docs/policy/lambda.html)<br>
[Lambda](https://github.com/capitalone/cloud-custodian/blob/master/docs/source/policy/lambda.rst)<br>
[AWS CloudWatch Schedule Rules](https://docs.aws.amazon.com/AmazonCloudWatch/latest/events/ScheduledEvents.html)<br>
[S3 Data Events](https://docs.aws.amazon.com/AmazonCloudWatch/latest/events/log-s3-data-events.html)<br>
[CloudWatch Rules Expressions](https://docs.aws.amazon.com/AmazonCloudWatch/latest/events/ScheduledEvents.html)<br>
[Adding Custom Fields to Reports](http://capitalone.github.io/cloud-custodian/docs/quickstart/advanced.html#adding-custom-fields-to-reports)<br>
[Custodian Mailer](https://github.com/capitalone/cloud-custodian/tree/master/tools/c7n_mailer)<br>
[C7N_Mailer](https://pypi.org/project/c7n_mailer/)<br>







