# Week 6: Containers and Serverless Computing

## HarborTech Ticket Summary

TKT-2026-0006 evaluates whether Bright Path Community Services is using the best compute model for its workloads.

The client currently runs a daily reporting task and a small REST API on one Amazon EC2 instance. The server works, but it remains lightly utilized for long periods while the client continues to manage the guest operating system, application runtime, patching, monitoring, and other server responsibilities.

The goal of this investigation was not to assume that every workload should move to serverless. Instead, I compared virtual machines, containers, and serverless computing based on the actual behavior and technical requirements of each workload.

The daily report runs briefly once per day and keeps no local state between executions. The REST API receives intermittent requests. A separate continuously running workload with custom operating-system requirements was also evaluated to show why a virtual machine or container may still be the better choice in some situations.

## Client Impact

A poorly matched compute model can increase cost, operational effort, and risk.

If a short workload runs on a continuously available EC2 instance, the client may pay for and manage compute capacity that remains idle most of the time. HarborTech must also continue handling operating-system maintenance, patching, service monitoring, runtime updates, and troubleshooting.

Moving suitable event-driven workloads to managed or serverless services can reduce direct infrastructure-management responsibility and allow the application to run only when needed.

However, choosing too much abstraction can also create problems. A workload that requires operating-system customization, persistent processes, or host-level troubleshooting may be a poor fit for Lambda.

The correct model must therefore be selected from workload evidence rather than service popularity.

## AWS Services Involved

### Containers

Containers package an application and its dependencies into a consistent execution environment. They provide more application isolation and portability than installing software directly on a traditional server while generally requiring less infrastructure than a full virtual machine.

### Serverless Computing

Serverless computing allows application code to execute without the customer operating the underlying server infrastructure directly. The customer still manages the code, configuration, permissions, monitoring, and application behavior.

### AWS Lambda

AWS Lambda is a serverless compute service that can execute application code in response to an event. It is a strong candidate for short and intermittent workloads that do not require a continuously running server.

### Amazon API Gateway

Amazon API Gateway can provide a managed HTTP interface for an application. For a serverless REST API, API Gateway can receive and route requests while Lambda performs the application logic.

### AWS Step Functions

AWS Step Functions can coordinate multiple tasks or services in a defined workflow. It can be useful when a workload contains several steps, decisions, retries, or failure paths rather than one simple function invocation.

### IAM Execution Roles

An IAM execution role provides the permissions a workload such as a Lambda function needs to interact with authorized AWS services.

Permissions remain an important operational responsibility even when the underlying compute infrastructure is managed by AWS.

### Event-Driven Computing

Event-driven computing executes work in response to an event or trigger rather than requiring a process to run continuously while waiting for work.

The Bright Path daily report is a strong candidate for this model because it runs for a short period once each day.

### VPC-Connected Lambda

A Lambda function can require VPC connectivity when it must communicate with private resources inside a VPC. That configuration can introduce additional networking considerations that must be validated before migration.

### Monitoring and Logs

Serverless computing reduces direct server management, but it does not remove the need for monitoring. HarborTech would still need visibility into application execution, errors, performance, logs, and failed events.

## Virtualization Connection

Virtual machines, containers, and serverless computing provide different levels of abstraction.

A virtual machine abstracts physical hardware while still giving the customer a full guest operating system. This provides significant control but also leaves the customer responsible for more operating-system administration, patching, configuration, and troubleshooting.

Containers package an application and its dependencies while sharing a host operating-system kernel. This can reduce the amount of environment-specific configuration required by the application while preserving more runtime control than a serverless function.

Serverless computing increases abstraction further. With AWS Lambda, the customer provides the application code and configuration while AWS manages the underlying execution infrastructure.

Increasing abstraction can reduce direct infrastructure management, but it also reduces the level of host control available to the application team. Therefore, the workload requirements must determine the appropriate model.

## Evidence Reviewed

### Daily Reporting Workload

The reporting task:

- Runs once each morning.
- Completes quickly.
- Keeps no local state between runs.
- Is idle for the remainder of the day.

This evidence weakens the need for a continuously running EC2 instance.

### REST API Workload

The REST API:

- Receives intermittent requests.
- Returns lightweight responses.
- Does not require constant processing based on the supplied workload evidence.

This makes a managed API and event-driven compute model worth evaluating.

### Continuously Running Workload

The comparison workload:

- Runs continuously.
- Requires a custom operating-system package.
- Requires host-level troubleshooting access.

This evidence favors greater infrastructure control and weakens the case for Lambda.

### Operational Responsibility

The current EC2 approach requires responsibility for the guest operating system and application runtime even while the server is lightly utilized.

Serverless services could move more infrastructure-management responsibility to AWS, but HarborTech would continue to own application code, permissions, configuration, monitoring, security, testing, and operational response.

### Validation Evidence Still Required

Before recommending an actual migration, I would collect:

- Real execution duration
- Execution frequency
- API request volume
- Runtime dependencies
- Operating-system requirements
- Application state requirements
- Storage requirements
- IAM permissions
- Network and VPC connectivity requirements
- Performance and latency requirements
- Logs and error behavior
- Monitoring requirements
- Failure and retry behavior
- Application integration dependencies

## Operational Analysis

The current EC2 server is capable of running the Bright Path workloads, but technical capability does not automatically mean it is the best operational model.

The daily reporting task spends almost all of its time doing nothing. It runs once per day, completes quickly, and requires no local state between executions. Operating a full server for that behavior creates unnecessary infrastructure responsibility.

AWS Lambda is therefore the compute model I would evaluate first for the report.

The REST API also has workload characteristics that support a serverless evaluation. Requests are intermittent rather than continuous. API Gateway could receive HTTP requests and manage the API interface, while Lambda could execute the application logic when requests arrive.

The continuously running comparison workload is different. Its custom operating-system package and host-level troubleshooting requirement indicate a need for more control.

If direct operating-system access is truly required, EC2 would be the strongest option.

A container model could become appropriate if the required package can be included in the application container and direct host-level troubleshooting is not actually necessary. More evidence would be required before choosing between a VM and container for that workload.

## Recommendation

### Daily Report

I recommend evaluating **AWS Lambda** first.

The strongest evidence is its short execution duration, once-per-day schedule, lack of local state, and long idle period.

A serverless function would allow the task to run when needed without HarborTech operating a continuously running guest operating system for this workload.

### REST API

I recommend evaluating **Amazon API Gateway with AWS Lambda**.

API Gateway could provide the managed HTTP endpoint and handle incoming requests, while Lambda executes the application logic only when required.

This design could reduce direct server-management responsibility while still allowing HarborTech to control the application behavior.

### Continuously Running Application

I would initially favor an **EC2 virtual machine** if host-level access and operating-system customization are confirmed requirements.

I would also evaluate a **container** if the custom package and dependencies can be packaged inside the container and the application does not actually need direct access to the host operating system.

I would not recommend Lambda for this workload based on the supplied evidence.

## Remaining Operational Responsibilities

Moving a workload to serverless does not eliminate operational responsibility.

AWS would manage more of the underlying compute infrastructure, but HarborTech and the client would still be responsible for:

- Application code
- Runtime configuration
- IAM permissions
- Authentication and authorization
- API configuration
- Input validation
- Application security
- Data protection
- Monitoring
- Logging
- Error handling
- Testing
- Dependency management
- Cost monitoring
- Operational response
- Application architecture

For a container model, HarborTech would additionally need to manage container images, dependencies, deployment configuration, and the application lifecycle.

For EC2, HarborTech would retain the most direct responsibility, including the guest operating system, patching, installed packages, service configuration, and server troubleshooting.

## Validation Items

Before recommending a production migration, I would validate the assumptions used in this analysis.

For the daily report, I would confirm its actual duration, schedule, memory and CPU needs, dependencies, storage requirements, network access, retry behavior, and whether it truly requires no persistent local state.

For the REST API, I would collect actual request volume, request patterns, latency requirements, authentication requirements, payload sizes, integrations, dependency behavior, and expected error handling.

For any Lambda design, I would validate IAM execution permissions, runtime compatibility, logging, monitoring, VPC connectivity if required, and integration behavior.

For the continuously running workload, I would verify whether host-level access is truly required and whether the custom operating-system package could instead be packaged inside a container.

A recommendation should be based on verified production workload evidence rather than assumptions from a simplified scenario.

## Escalation Notes

As an intern, I can analyze evidence and recommend an execution model, but I should not independently approve or perform a production architecture migration.

The following actions require escalation and appropriate approval:

- Migrating the production reporting workload
- Replacing or terminating the production EC2 instance
- Changing the production API architecture
- Creating or modifying production IAM permissions
- Changing production networking
- Connecting Lambda to private VPC resources
- Changing authentication or authorization
- Moving production data
- Selecting the final production architecture
- Making changes that affect availability or security

These decisions should be reviewed by the appropriate HarborTech engineer, security administrator, system owner, or change-control authority.

## Lessons Learned

Week 6 showed that selecting a compute service should begin with workload behavior rather than with a preferred AWS service.

A virtual machine provides the most direct operating-system control but also requires the customer to manage more infrastructure.

Containers can package an application and its dependencies consistently while reducing some of the operating-system dependency of a traditional server deployment.

Serverless computing removes even more direct infrastructure management and is especially useful for short, event-driven workloads.

The Bright Path daily report demonstrates why an always-running server may be unnecessary for a task that executes for only a short period once per day.

The REST API demonstrates how multiple managed services can divide responsibilities. API Gateway can provide the API interface while Lambda performs application processing.

The continuously running application demonstrates why serverless is not automatically the best solution. Custom operating-system requirements and host-level troubleshooting can justify a VM or possibly a container-based solution.

The most important lesson is that architecture decisions should follow evidence about timing, state, dependencies, triggers, control requirements, integrations, and operational responsibility.

## Professional Vocabulary

### Container

A container packages an application and its dependencies into a consistent execution environment while generally sharing the host operating-system kernel.

### Serverless Computing

Serverless computing is an execution model where the cloud provider manages the underlying server infrastructure while the customer focuses primarily on application code and configuration.

### AWS Lambda

AWS Lambda is a serverless compute service that executes code in response to events without requiring the customer to operate a continuously running server.

### API Gateway

Amazon API Gateway is a managed service used to create and operate APIs. It can receive requests and route them to backend services such as Lambda.

### Execution Role

An execution role is an IAM role that grants a workload permission to access specific AWS resources or services.

### Event Trigger

An event trigger is an event or condition that causes a workload such as a Lambda function to execute.

### Step Functions

AWS Step Functions is a workflow service used to coordinate multiple tasks, decisions, retries, and service integrations.

### VPC-Connected Lambda

A VPC-connected Lambda function is configured to access resources through an Amazon VPC when private network connectivity is required.

### Runtime

A runtime is the execution environment in which application code runs, including the supported programming-language environment.

### Abstraction

Abstraction hides lower-level infrastructure details from the layer above them. Moving from virtual machines to containers to serverless generally increases infrastructure abstraction and reduces direct server-management responsibility.
