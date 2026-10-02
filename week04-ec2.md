# Week 4: EC2 Evidence Lab

## HarborTech Ticket Summary

Riverside Goods needed HarborTech to build and troubleshoot an Amazon EC2 web server. The purpose of the lab was to provision an Amazon Linux EC2 instance running Apache, reproduce a controlled HTTP reachability failure, diagnose the problem using AWS and guest evidence, correct only the layer supported by the evidence, verify the repair, observe stop/start lifecycle behavior, and clean up the resources.

The controlled failure was created by launching the EC2 instance with a security group that did not initially allow inbound TCP port 80.

## Client Impact

The Riverside Goods web server was running but initially could not be reached through its public IPv4 address. In a production environment, this could prevent customers from accessing the website even though the application itself is functioning correctly.

The investigation therefore needed to determine whether the problem was caused by the EC2 instance, Apache, the operating system, or AWS network access controls before making a corrective change.

## Environment and Resource Names

- AWS Region: `us-east-1`
- VPC: `vpc-04f193830f5a3f155`
- VPC CIDR: `172.31.0.0/16`
- Subnet: `subnet-06ce19b831987fa56`
- Availability Zone: `us-east-1a`
- Amazon Linux AMI: `ami-0d27e0fb3bac4d724`
- Instance type: `t3.micro`
- Instance ID: `i-09e03d624a359b265`
- Initial public IPv4: `34.200.231.145`
- Security Group: `sg-0a92451aa6810321d`
- Security Group Name: `week4-riverside-sg-1790979552`
- Temporary IAM Role: `Week4SSMRole`
- Temporary Instance Profile: `Week4SSMInstanceProfile`

The Learner Lab initially contained no usable EC2 instance profile for Session Manager. I verified that the AWS-managed `AmazonSSMManagedInstanceCore` policy was available and created a temporary EC2 role and instance profile for this lab.

## AWS Documentation Evidence

### Security Groups

**Document title:** Amazon Virtual Private Cloud User Guide

**PDF page:** 517

**Exact quote:** “When you first create a security group, it has no inbound rules.”

**Application to this lab:**  
This supports my observation that the new security group initially allowed no inbound HTTP traffic. My AWS evidence showed an empty inbound-rule list, and an external HTTP request timed out. After I added TCP port 80, the same server became reachable.

### User Data

**Document title:** Amazon Elastic Compute Cloud User Guide for Linux Instances

**PDF page:** 315

**Exact quote:** “User data scripts and cloud-init directives only run during the first boot cycle when an instance is launched.”

**Application to this lab:**  
I used user data to install Apache, enable and start the service, and create the Riverside Goods page. However, the script itself only proves what I instructed the instance to do. I separately used Session Manager, `systemctl`, and `curl http://localhost` to verify that the application was actually running.

### EC2 Lifecycle

**Document title:** Amazon Elastic Compute Cloud User Guide for Linux Instances

**PDF page:** 586

**Exact quote:** “We release the public IP address for your instance when it's stopped or terminated.”

**Application to this lab:**  
This supports the result of my lifecycle test. The same EC2 instance remained after the stop/start operation, but its automatically assigned public IPv4 address changed. The web-page files also remained available because they were stored on the EBS-backed instance.

## CloudShell Command Record

### Identity and Region Verification

I began by verifying my AWS identity:

```bash
aws sts get-caller-identity
```

Account-specific identity values are intentionally not included in this public GitHub entry.

I checked the configured Region:

```bash
aws configure get region
```

That command returned no value, so I verified the environment variable:

```bash
echo $AWS_REGION
```

Result:

```text
us-east-1
```

I also verified the Region using EC2:

```bash
aws ec2 describe-availability-zones \
  --query 'AvailabilityZones[0].RegionName' \
  --output text
```

Result:

```text
us-east-1
```

### VPC and Subnet Discovery

```bash
aws ec2 describe-vpcs \
  --query 'Vpcs[].{VpcId:VpcId,Default:IsDefault,Cidr:CidrBlock}' \
  --output table
```

Relevant result:

```text
VpcId: vpc-04f193830f5a3f155
CIDR: 172.31.0.0/16
Default: True
```

I inspected the available subnets and selected:

```text
Subnet: subnet-06ce19b831987fa56
Availability Zone: us-east-1a
Public IP on launch: True
```

### AMI Discovery

```bash
aws ssm get-parameter \
  --name /aws/service/ami-amazon-linux-latest/al2023-ami-kernel-default-x86_64 \
  --query 'Parameter.Value' \
  --output text
```

Result:

```text
ami-0d27e0fb3bac4d724
```

### Session Manager Setup

I checked for existing instance profiles:

```bash
aws iam list-instance-profiles \
  --query 'InstanceProfiles' \
  --output json
```

Result:

```json
[]
```

I also checked for the common lab resources `LabInstanceProfile` and `LabRole`, but AWS returned `NoSuchEntity` for both.

I created a temporary role and instance profile for Session Manager and attached the AWS-managed `AmazonSSMManagedInstanceCore` policy.

Verification showed:

```text
Instance Profile: Week4SSMInstanceProfile
Role: Week4SSMRole
```

### Security Group Creation

I created a unique security group for the lab:

```bash
SG_NAME="week4-riverside-sg-$(date +%s)"
```

The resulting resources were:

```text
SG_NAME=week4-riverside-sg-1790979552
SG_ID=sg-0a92451aa6810321d
```

I inspected the security group:

```bash
aws ec2 describe-security-groups \
  --group-ids "$SG_ID" \
  --query 'SecurityGroups[0].{GroupId:GroupId,GroupName:GroupName,InboundRules:IpPermissions}' \
  --output json
```

Result:

```json
{
  "GroupId": "sg-0a92451aa6810321d",
  "GroupName": "week4-riverside-sg-1790979552",
  "InboundRules": []
}
```

### Apache User Data

I prepared the following user-data script:

```bash
#!/bin/bash
dnf -y install httpd
systemctl enable httpd
systemctl start httpd
cat > /var/www/html/index.html <<'HTML'
<h1>Riverside Goods</h1>
<p>HarborTech Week 4 EC2 Evidence Lab</p>
HTML
```

### Initial Launch Failure and Correction

My first EC2 launch attempt used `t2.micro`.

AWS returned:

```text
InvalidParameterCombination:
The specified instance type is not eligible for Free Tier.
```

I investigated the available free-tier instance types:

```bash
aws ec2 describe-instance-types \
  --filters Name=free-tier-eligible,Values=true \
  --query "InstanceTypes[?contains(ProcessorInfo.SupportedArchitectures, 'x86_64')].InstanceType" \
  --output table
```

The result included `t3.micro`.

I also verified that the selected AMI architecture was:

```text
x86_64
```

I corrected the launch command to use `t3.micro`.

Result:

```text
INSTANCE_ID=i-09e03d624a359b265
```

The instance reached the running state.

Verification showed:

```text
AMI: ami-0d27e0fb3bac4d724
InstanceId: i-09e03d624a359b265
State: running
Type: t3.micro
```

The initial public IPv4 address was:

```text
34.200.231.145
```

## Baseline Evidence

### Evidence A — Instance ID and AMI ID

```text
Instance ID: i-09e03d624a359b265
AMI ID: ami-0d27e0fb3bac4d724
Instance Type: t3.micro
State: running
```

This proves that the EC2 instance successfully launched from the selected Amazon Linux AMI. It does not prove that Apache was functioning correctly.

### Evidence B — Initial Public IPv4

```text
34.200.231.145
```

This proves that AWS assigned the instance a public IPv4 address. A public address alone does not prove that inbound HTTP traffic is allowed.

### Evidence C — Status Checks

At the time I first collected the status evidence:

```text
InstanceState: running
SystemStatus: ok
InstanceStatus: initializing
```

The system status had passed, but the instance check was still initializing at that moment. I therefore did not treat that observation as proof that both status checks had completed.

### Evidence D — Security Group Before Fix

```json
{
  "GroupId": "sg-0a92451aa6810321d",
  "InboundRules": []
}
```

This proves that there was no inbound rule allowing HTTP traffic.

### Evidence E — Failed HTTP Test

```bash
curl -v --max-time 5 "http://34.200.231.145"
```

Result:

```text
Trying 34.200.231.145:80...
Connection timed out after 5002 milliseconds
curl: (28) Connection timed out after 5002 milliseconds
```

This proved that HTTP was not externally reachable, but the timeout alone did not identify which layer caused the problem.

## Root-Cause Analysis

The evidence supported the security group as the root cause rather than Apache or the EC2 workload.

The instance was running and had a public IPv4 address, but its security group had no inbound rules. An external HTTP request to TCP port 80 timed out.

I then used Session Manager to inspect the guest operating system. Apache was active and `curl http://localhost` returned the expected Riverside Goods webpage. That evidence ruled out Apache being stopped or the local webpage being unavailable.

Because the application worked locally while inbound HTTP traffic could not reach the instance, the missing TCP port 80 security-group rule was the supported root cause.

Rebuilding the EC2 instance was not justified because the existing workload was functioning correctly.

## Corrective Action

I made the smallest change supported by the evidence:

```bash
aws ec2 authorize-security-group-ingress \
  --group-id "$SG_ID" \
  --protocol tcp \
  --port 80 \
  --cidr 0.0.0.0/0
```

AWS returned:

```text
Return: true
SecurityGroupRuleId: sgr-0c415d56372d2a040
Protocol: tcp
FromPort: 80
ToPort: 80
CIDR: 0.0.0.0/0
```

I did not rebuild or replace the instance.

## Verification Evidence

I inspected the security group again after remediation.

It now contained:

```text
Protocol: tcp
FromPort: 80
ToPort: 80
CIDR: 0.0.0.0/0
```

I then repeated the external test:

```bash
curl -v --max-time 10 "http://$PUBLIC_IP"
```

Result:

```text
HTTP/1.1 200 OK
Server: Apache/2.4.68 (Amazon Linux)

<h1>Riverside Goods</h1>
<p>HarborTech Week 4 EC2 Evidence Lab</p>
```

The same instance that previously timed out became reachable after only the security-group rule was changed. This before-and-after result supports the security-group root-cause finding.

## IMDSv2 and Guest Evidence

Systems Manager reported the instance as:

```text
InstanceId: i-09e03d624a359b265
PingStatus: Online
Platform: Amazon Linux
```

I opened a Session Manager session and checked Apache:

```bash
systemctl is-active httpd
```

Result:

```text
active
```

I tested the application locally:

```bash
curl -s http://localhost
```

Result:

```html
<h1>Riverside Goods</h1>
<p>HarborTech Week 4 EC2 Evidence Lab</p>
```

I then performed an IMDSv2 request from inside the EC2 instance.

```bash
TOKEN=$(curl --noproxy "*" -sS -X PUT \
  -H "X-aws-ec2-metadata-token-ttl-seconds: 21600" \
  http://169.254.169.254/latest/api/token)
```

Verification showed:

```text
TOKEN_LENGTH=56
```

I used that token to request the instance ID:

```bash
curl --noproxy "*" -sS \
  -H "X-aws-ec2-metadata-token: $TOKEN" \
  http://169.254.169.254/latest/meta-data/instance-id
```

Result:

```text
i-09e03d624a359b265
```

The IMDSv2 result matched the instance ID reported by the AWS control plane.

The guest evidence answers a different question from the CloudShell AWS CLI evidence. CloudShell showed AWS-side resource configuration, while Session Manager proved the actual service state and webpage behavior from inside the operating system.

A later metadata request was accidentally attempted from CloudShell after I had exited the instance. That request was not used as IMDSv2 evidence because the required verification was the successful metadata request executed from inside the EC2 guest.

## Stop/Start Lifecycle Test

I recorded the instance information and website before stopping the instance.

For the documented lifecycle cycle:

```text
Instance ID: i-09e03d624a359b265
Old public IPv4: 32.196.134.21
```

Before stopping the instance:

```bash
curl -s "http://32.196.134.21"
```

Result:

```html
<h1>Riverside Goods</h1>
<p>HarborTech Week 4 EC2 Evidence Lab</p>
```

I stopped the instance:

```bash
aws ec2 stop-instances \
  --instance-ids "$INSTANCE_ID"
```

AWS showed:

```text
Previous: running
Current: stopping
InstanceId: i-09e03d624a359b265
```

I waited for the stopped state:

```bash
aws ec2 wait instance-stopped --instance-ids "$INSTANCE_ID"
```

While stopped, AWS showed:

```text
InstanceId: i-09e03d624a359b265
State: stopped
PublicIP: None
```

I restarted the same instance:

```bash
aws ec2 start-instances \
  --instance-ids "$INSTANCE_ID"
```

AWS showed:

```text
Previous: stopped
Current: pending
InstanceId: i-09e03d624a359b265
```

I waited for both running state and status checks:

```bash
aws ec2 wait instance-running --instance-ids "$INSTANCE_ID"
aws ec2 wait instance-status-ok --instance-ids "$INSTANCE_ID"
```

After restart:

```text
Instance ID: i-09e03d624a359b265
Old public IPv4: 32.196.134.21
New public IPv4: 44.222.95.57
```

I tested the new public address:

```bash
curl -v --max-time 10 "http://44.222.95.57"
```

Result:

```text
HTTP/1.1 200 OK
Server: Apache/2.4.68 (Amazon Linux)

<h1>Riverside Goods</h1>
<p>HarborTech Week 4 EC2 Evidence Lab</p>
```

### Lifecycle Analysis

The instance ID remained `i-09e03d624a359b265`, proving that this was the same EC2 instance before and after the stop/start operation.

The Riverside Goods page also persisted and Apache returned HTTP 200 after restart. This is consistent with the web files being stored on the EBS-backed root volume, which persists through a normal EC2 stop/start cycle.

The automatically assigned public IPv4 address did not persist. The instance had `32.196.134.21` before the documented stop and `44.222.95.57` after restart. This matched the AWS documentation explaining that the automatically assigned public IP is released when an instance is stopped and a new public IP can be assigned when it restarts.

The test demonstrated that instance identity and EBS-backed application data can persist even though the automatically assigned public network address changes.

## Cleanup Evidence

After completing all verification, I terminated the disposable EC2 instance.

```bash
aws ec2 terminate-instances \
  --instance-ids "$INSTANCE_ID"
```

AWS returned:

```text
Previous: running
Current: shutting-down
InstanceId: i-09e03d624a359b265
```

I waited for termination:

```bash
aws ec2 wait instance-terminated --instance-ids "$INSTANCE_ID"
```

Final verification showed:

```text
InstanceId: i-09e03d624a359b265
State: terminated
```

I then deleted the Week 4 security group after the instance was no longer attached to it and verified that the group no longer existed.

I also cleaned up the temporary Session Manager IAM resources created for the lab:

```bash
aws iam remove-role-from-instance-profile \
  --instance-profile-name Week4SSMInstanceProfile \
  --role-name Week4SSMRole

aws iam delete-instance-profile \
  --instance-profile-name Week4SSMInstanceProfile

aws iam detach-role-policy \
  --role-name Week4SSMRole \
  --policy-arn arn:aws:iam::aws:policy/AmazonSSMManagedInstanceCore

aws iam delete-role \
  --role-name Week4SSMRole
```

This removed the disposable compute and temporary management resources created specifically for the Week 4 exercise.

## Escalation and Change-Control Notes

I would not run `aws ec2 authorize-security-group-ingress` against a production client's security group without authorization.

An incorrect inbound rule could expose a production application to unintended traffic. Before modifying a production security group, I would seek approval from the appropriate system owner, security administrator, or change manager and verify the required protocol, port, source range, and business purpose.

I would also document the existing security-group configuration before making the change so that the new rule could be revoked if validation failed.

Stopping, starting, or terminating a production EC2 instance would also require authorization because those operations can directly affect application availability.

The evidence in this lab did not justify rebuilding the server. Apache and the application were already operating correctly, so only the supported network-access layer needed to be changed.

## Lessons Learned

This lab demonstrated why cloud troubleshooting should follow an evidence-based process instead of immediately rebuilding resources.

A running EC2 instance and a public IPv4 address do not prove that an application is reachable. AWS infrastructure state, security-group rules, operating-system state, application state, and the network path answer different questions.

The first external HTTP timeout proved that Riverside Goods was unreachable but did not identify the cause by itself. Session Manager showed that Apache was active and `curl http://localhost` returned the expected page. That evidence helped isolate the failure to the inbound network-access layer.

The before-and-after security-group evidence was especially useful. Before the correction there were no inbound rules and HTTP timed out. After only TCP port 80 was allowed, the same instance returned HTTP 200.

The lifecycle test also demonstrated that different EC2 resources have different persistence behavior. The instance ID and EBS-backed application content remained, while the automatically assigned public IPv4 address changed.

I also learned that unsuccessful commands are part of a professional troubleshooting record. My initial `t2.micro` launch failed because that instance type was not eligible for the lab's free tier. I investigated the allowed types, selected `t3.micro`, and successfully launched the instance.

## Professional Vocabulary

### Amazon EC2

Amazon Elastic Compute Cloud is the AWS service used to create and operate virtual server instances.

### AMI

An Amazon Machine Image is a reusable template used to launch EC2 instances.

### Instance Type

An instance type determines the compute, memory, networking, and other resource characteristics available to an EC2 instance.

### Security Group

A security group is a stateful virtual firewall that controls allowed inbound and outbound network traffic for associated resources.

### Inbound Rule

An inbound rule defines traffic that is allowed to reach a resource based on protocol, port, and source.

### Public IPv4 Address

A public IPv4 address allows a resource to be addressed publicly when routing and security controls also permit the traffic.

### User Data

User data is information or a script supplied to an EC2 instance that can perform configuration tasks during launch.

### Apache HTTP Server

Apache is the web-server software used to serve the Riverside Goods test webpage.

### AWS Systems Manager

AWS Systems Manager provides management capabilities for AWS resources.

### Session Manager

Session Manager provides managed shell access to supported instances without requiring inbound SSH access.

### IAM Role

An IAM role provides temporary permissions that can be assumed by AWS services or workloads.

### Instance Profile

An instance profile is used to provide an IAM role to an EC2 instance.

### AmazonSSMManagedInstanceCore

`AmazonSSMManagedInstanceCore` is an AWS-managed IAM policy commonly used to provide Systems Manager permissions to managed instances.

### IMDSv2

Instance Metadata Service Version 2 uses a token-based process to retrieve metadata about the EC2 instance from inside the instance.

### Instance Metadata

Instance metadata is information about a running EC2 instance that can be retrieved through the instance metadata service.

### Status Checks

EC2 status checks monitor infrastructure-level and instance-level conditions.

### Amazon EBS

Amazon Elastic Block Store provides persistent block storage used by EC2 instances.

### Stop

Stopping an EBS-backed instance shuts down its compute resources without terminating the EC2 instance.

### Start

Starting a stopped EC2 instance returns that same instance to the running state.

### Terminate

Terminating an instance permanently ends the EC2 instance.

### Root Cause

The root cause is the underlying condition responsible for an observed technical problem.

### Remediation

Remediation is the corrective action taken to address an identified problem.

### Verification

Verification is the process of collecting evidence after a change to confirm that the intended result was achieved.

### Change Control

Change control is the process of reviewing, approving, documenting, implementing, and validating changes to managed systems.
