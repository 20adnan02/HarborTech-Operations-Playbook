# Week 2: IAM and AWS CLI Investigation

## HarborTech Ticket Summary

Ticket TKT-2026-0002 concerns an authorization problem reported by Riverside Goods. Marcus Webb can successfully sign in to the AWS console, but he receives AccessDenied when trying to access the riverside-inventory S3 bucket.

The evidence shows that Marcus can authenticate, but he does not have the authorization required to perform his inventory work.

## Client Impact

Marcus cannot complete his assigned inventory duties because he lacks the permissions needed to list the riverside-inventory bucket, read inventory report objects, and upload approved inventory files.

Granting too much access would also create unnecessary security risk, so the solution should provide only the permissions required for his job.

## AWS Services Involved

The investigation involves:

- AWS Identity and Access Management (IAM)
- IAM users, groups, roles, and policies
- Amazon S3
- AWS CloudShell
- AWS Command Line Interface (AWS CLI)
- AWS Regions
- Authentication
- Authorization
- Least privilege

## Virtualization Connection

Cloud infrastructure is software-defined, but access to virtual resources still depends on identity and permission controls.

IAM determines who can manage or use AWS resources such as storage, compute, and networking. A user may successfully sign in to AWS but still be blocked from a resource if the required permissions are not granted.

## Evidence Reviewed

The evidence reviewed included:

- Marcus successfully signed in to AWS.
- Marcus is an Inventory Coordinator.
- He needs to list the riverside-inventory bucket.
- He needs to read inventory report objects.
- He needs to upload approved inventory files.
- He does not administer EC2 resources or unrelated S3 buckets.
- No job-function group membership is listed.
- No directly attached permission policy is listed.
- The request to list the bucket returned AccessDenied.
- The client proposed AmazonS3FullAccess.
- AWS CLI caller identity evidence was reviewed.

## Operational Analysis

Marcus's successful sign-in proves authentication is working. AWS accepted his credentials and identified him as a valid user.

The AccessDenied result proves that he is not authorized to perform the requested S3 action.

The evidence therefore supports an authorization gap, not an authentication failure.

The client's proposal to attach AmazonS3FullAccess would grant more access than Marcus needs and would violate the principle of least privilege.

## Recommendation

Marcus should receive only the S3 permissions necessary for his inventory duties.

The required actions are:

- s3:ListBucket
- s3:GetObject
- s3:PutObject

These permissions should be scoped only to the riverside-inventory bucket and the appropriate objects.

A properly scoped job-function policy or group-based permission is more appropriate than AmazonS3FullAccess.

## Escalation Notes

The supported finding is that Marcus can authenticate successfully but lacks the required authorization for his inventory duties.

The production access change should be reviewed and implemented by an authorized HarborTech team member.

The intern should inspect, document, recommend, and escalate, but should not directly make production IAM permission changes.

## Learner Lab CLI Evidence

I ran:

`aws sts get-caller-identity`

The command returned:

Account: `618790904765`

ARN: `arn:aws:iam::618790904765:root`

This confirms the AWS account and caller identity associated with the CLI session.

The ARN shows that this session is using the root identity rather than the Learner Lab LabRole. Because this environment is not the AWS Academy Learner Lab, this result does not provide evidence about LabRole's trust relationship or permission sources.

A role's trust relationship determines which principals are allowed to assume the role, while permission policies determine what AWS actions the role may perform after it is assumed.

LabRole should not be recommended as the solution for Marcus because it belongs to a separate training environment and does not represent the Riverside Goods production access model.

## Lessons Learned

Week 2 showed that authentication and authorization are different.

A successful login does not mean a user has permission to access AWS resources. AccessDenied errors should be investigated by reviewing the requested action, the identity, the permission sources, and the resource scope.

I also learned that least privilege means granting only the permissions required for the user's job instead of using broad policies such as AmazonS3FullAccess.

## Professional Vocabulary

**Authentication:** The process of verifying the identity of a user or system.

**Authorization:** The process of determining what actions an authenticated identity is allowed to perform.

**IAM:** AWS Identity and Access Management, which manages identities and permissions.

**Policy:** A document that defines allowed or denied AWS actions and resources.

**Least Privilege:** Granting only the permissions necessary to complete required job duties.

**AccessDenied:** An AWS response showing that the requested action is not authorized.

**AWS CLI:** The AWS Command Line Interface used to interact with AWS through commands.

**CloudShell:** A browser-based terminal environment that includes the AWS CLI.

**Caller Identity:** The AWS account and identity associated with the current CLI request.

**Resource Scope:** The AWS resources to which a permission applies.
