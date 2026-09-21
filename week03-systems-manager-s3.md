# Week 3: Systems Manager and S3

## HarborTech Ticket Summary

Ticket TKT-2026-0003 focuses on two issues for Bright Path Community Services. Dana needs to run the same maintenance command on five EC2 instances, which creates unnecessary manual work. Bright Path also needs a simple public webpage for program hours, contact information, images, and downloadable forms. The goal is to reduce repeated administration and use AWS services that fit each workload.

## Client Impact

Running the same maintenance task manually on five EC2 instances takes more time and increases the chance of mistakes or inconsistent configurations. Centralized management can make the work faster and more consistent. It can also make future troubleshooting and support easier because the same process can be used across all systems.

## AWS Services Involved

The AWS services and concepts involved in this investigation include:

* AWS Systems Manager
* Run Command
* Session Manager
* Inventory
* Parameter Store
* Amazon S3
* Static Website Hosting
* AWS CLI
* CloudShell

Run Command can execute the same command across multiple managed EC2 instances. Session Manager can provide interactive access to an instance. Inventory can collect information about managed systems, while Parameter Store can keep configuration values in one central location. Amazon S3 can store and host static website files without requiring a separate web server.

## Virtualization Connection

AWS Systems Manager provides a centralized way to manage virtual machines such as EC2 instances. Instead of connecting to every EC2 instance separately, an administrator can use Systems Manager to perform supported tasks from one location.

Amazon S3 also shows that not every workload requires a virtual machine. A basic static website can be stored as files in S3 instead of running an EC2 instance with an operating system and web server.

## Evidence Reviewed

Dana needs the same maintenance command executed on five EC2 instances. Because the task is repeated and does not require an interactive session, Systems Manager Run Command is a good fit.

Before Systems Manager can manage the instances, they need the SSM Agent installed and running, the proper IAM permissions or instance profile, and network connectivity to Systems Manager.

Bright Path also has environment-specific values that can be stored centrally with Parameter Store. The application or automation must be configured to retrieve the stored value by its parameter name.

For the website portion of the investigation, I created an S3 bucket named **brightpath-al-350011** in the **us-east-1** Region.

The uploaded website object was:

**index.html**

The static website endpoint was:

**http://brightpath-al-350011.s3-website-us-east-1.amazonaws.com**

The Learner Lab blocked the public-read configuration needed to make the website publicly accessible. The website endpoint therefore returned an access-denied result. I documented the sandbox restriction instead of attempting to bypass it.

I also used CloudShell to update the website with the AWS CLI.

Command used:

`aws s3 sync ./ s3://brightpath-al-350011/`

The output showed:

`upload: ./index.html to s3://brightpath-al-350011/index.html`

## Operational Analysis

Dana's repeated maintenance task is better suited for centralized management and automation than manually connecting to each EC2 instance. Run Command allows HarborTech to execute the same non-interactive maintenance command across the managed instances.

Session Manager would be more appropriate if an administrator needed an interactive command-line session with an EC2 instance. Inventory would be useful for collecting information about the managed systems rather than executing maintenance commands.

Parameter Store is useful for storing shared configuration values in one place. The application or automation must request the value from Parameter Store using the parameter name. Parameter Store does not automatically search for and rewrite existing configuration files.

For the Bright Path public resource page, S3 static website hosting is appropriate because the website contains static information such as hours, contact information, images, and downloadable forms. It does not require a traditional server.

## Recommendation

HarborTech should use **AWS Systems Manager Run Command** for Dana's repeated maintenance tasks across the five EC2 instances. The instances should first be confirmed as managed nodes with the SSM Agent running, proper IAM permissions, and network connectivity to Systems Manager.

HarborTech should use **Parameter Store** for shared environment-specific configuration values and update the application or automation so it retrieves those values by parameter name.

Bright Path should use **Amazon S3 static website hosting** for its public informational webpage because the site currently contains only static content.

## Escalation Notes

The Learner Lab restricted the public-read settings required for the S3 static website to be publicly accessible. In a normal AWS account, an administrator with the correct permissions would need to review the S3 Block Public Access settings and apply an appropriate bucket policy that allows public read access to the website files.

I did not attempt to bypass the Learner Lab restrictions and documented the denied access as required.

The five EC2 instances would also need to meet the Systems Manager prerequisites before Run Command could be used successfully.

## Lessons Learned

Week 3 showed me how centralized management can reduce repetitive administration and improve consistency. Systems Manager can manage multiple EC2 instances without requiring an administrator to manually perform the same task on each system.

I also learned that AWS services should be selected based on the type of workload. Run Command is useful for repeated commands, Session Manager is better for interactive access, Parameter Store can centralize configuration values, and S3 can host simple static content without requiring a traditional server.

## Professional Vocabulary

**Systems Manager:** An AWS service used to centrally manage and operate AWS resources and supported servers.

**Managed Node:** A server or virtual machine that is configured so AWS Systems Manager can manage it.

**Run Command:** A Systems Manager feature that allows commands to be remotely executed on one or more managed nodes.

**Session Manager:** A Systems Manager feature that allows administrators to interactively connect to managed systems.

**Inventory:** A Systems Manager feature that collects information about managed systems, such as software and configuration details.

**Parameter Store:** A Systems Manager feature used to centrally store configuration values that applications or automation can retrieve by name.

**Automation:** Using tools and predefined processes to complete repeated tasks with less manual work.

**Static Website Hosting:** Hosting website files such as HTML, images, and downloads without using server-side processing.

**Object Storage:** A storage method where files are stored as objects inside containers such as Amazon S3 buckets.

**Management Plane:** The tools and services administrators use to configure, control, and manage infrastructure.
