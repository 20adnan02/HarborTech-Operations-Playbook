# Week 7: Managed Database Services

## HarborTech Ticket Summary

TKT-2026-0007 is a database operations assessment for a café that currently runs a transactional MySQL database on Amazon EC2.

The database is working, but the client is responsible for the guest operating system, MySQL maintenance, patching, backups, storage management, and recovery planning. HarborTech was asked to determine whether continuing to manage MySQL on EC2 is still the best operational model or whether a managed relational database service should be evaluated.

The current application uses structured tables, SQL queries, relationships, and transactions. Because of those requirements, the main option I would evaluate first is Amazon RDS for MySQL rather than immediately changing the application to a different database model.

No production migration has been approved as part of this investigation.

## Client Impact

The database supports the café's application and business transactions, so database availability and reliability directly affect business operations.

Keeping MySQL on EC2 provides a high level of control, but it also means the client must spend time managing the operating system, database software, patching, backups, storage, monitoring, and recovery.

A poorly planned migration could create downtime, application compatibility problems, data loss, performance issues, or unexpected costs.

Choosing the wrong database model could also require unnecessary changes to the application. For example, moving a relational MySQL workload directly to DynamoDB could require changes to queries, relationships, transactions, and application code.

For those reasons, the client should evaluate the workload carefully before making a production change.

## AWS Services Involved

### Amazon RDS

Amazon RDS is a managed relational database service. It can reduce some of the infrastructure and maintenance responsibilities involved with running MySQL directly on EC2.

### Amazon Aurora

Amazon Aurora is a managed relational database option that is compatible with MySQL or PostgreSQL. I would evaluate Aurora only if the workload evidence shows that its availability, performance, or scaling characteristics are needed.

### Amazon DynamoDB

Amazon DynamoDB is a managed NoSQL database service. It is designed around key-based access patterns and is not a direct replacement for every relational database workload.

### Relational Databases

A relational database organizes data into structured tables and supports relationships, SQL queries, and transactions. The café's existing MySQL workload uses this model.

### NoSQL Databases

NoSQL databases use data models other than traditional relational tables. DynamoDB is an example of a NoSQL database.

### Multi-AZ

Multi-AZ is an availability option that can provide database redundancy across Availability Zones for supported RDS configurations.

### Database Migration Service

AWS Database Migration Service is used to help move database data between supported source and target databases.

### Schema Conversion

Schema conversion is the process of adapting database objects or structures when the source and target database engines are different.

### Security Groups

Security groups control allowed network traffic to AWS resources. Database connectivity must be limited to the systems that actually require access.

### Database Connectivity

Database connectivity includes the network path, endpoint, port, security rules, credentials, and application configuration required for an application to communicate with a database.

## Virtualization Connection

Running MySQL on EC2 virtualizes the physical server, but the customer still operates the guest operating system and database software.

The client is responsible for tasks such as operating-system patching, MySQL maintenance, backups, storage management, monitoring, and recovery planning.

A managed database service such as Amazon RDS abstracts more of the underlying compute, operating system, storage, and database infrastructure.

This can reduce the amount of server administration HarborTech and the client must perform.

The responsibility does not completely move to AWS. The customer still owns the data, database design, users and permissions, application queries, security configuration, monitoring decisions, and application integration.

The tradeoff is therefore less infrastructure management in exchange for operating within the limits and configuration options of a managed service.

## Evidence Reviewed

### Current Workload Evidence

The café application currently uses:

- MySQL
- Structured relational tables
- SQL queries
- Transactions
- A database running on Amazon EC2

The client is currently responsible for:

- Guest operating-system maintenance
- Database patching
- Backup execution and verification
- Storage management
- Recovery planning

This evidence supports evaluating a managed relational database instead of immediately changing to a different database model.

### CloudShell Evidence

CloudShell was available in my AWS Learner Lab.

I ran:

```bash
aws configure get region
```

No default Region value was returned, so I explicitly used:

```text
us-east-1
```

My first database discovery commands accidentally used the placeholder:

```text
YOUR_REGION
```

AWS returned:

```text
Provided region_name 'YOUR_REGION' doesn't match a supported format.
```

I corrected the commands by replacing the placeholder with `us-east-1`.

### RDS Evidence

I ran:

```bash
aws rds describe-db-instances \
  --region us-east-1 \
  --query 'DBInstances[].{Name:DBInstanceIdentifier,Engine:Engine,Status:DBInstanceStatus}' \
  --output table
```

My final result showed:

```text
Name: database-1
Engine: sqlserver-ex
Status: available
```

This confirmed that the RDS service was accessible and that one RDS database instance was available in the lab at the time of the final check.

The database shown in this evidence was SQL Server Express, so it does not prove that the café's MySQL application is compatible with RDS for MySQL or that a migration has been completed.

### RDS Console Evidence

I opened:

```text
AWS Console → Amazon RDS → Databases → Create database
```

I reviewed the database creation workflow and observed MySQL as an available database engine.

I also reviewed configuration areas for:

- Deployment and availability
- DB instance configuration
- Storage
- Connectivity
- Backups

This showed that RDS provides managed relational database configuration options that could reduce some of the infrastructure work currently handled on EC2.

### DynamoDB Evidence

I ran:

```bash
aws dynamodb list-tables \
  --region us-east-1 \
  --output table
```

The result was:

```text
------------
|ListTables|
+----------+
```

No DynamoDB tables were returned.

I also reviewed:

```text
AWS Console → DynamoDB → Tables → Create table
```

The page included options such as:

- Table name
- Partition key
- Optional sort key
- Capacity or billing settings

The DynamoDB data model is different from the café's existing relational MySQL model.

### Missing Evidence

The client has not yet provided enough information to verify:

- Transaction volume
- Database size
- Recovery Time Objective
- Recovery Point Objective
- Performance requirements
- Availability requirements
- Application dependencies
- Detailed MySQL compatibility requirements
- Migration downtime limits
- Budget
- Long-term growth
- Exact backup retention requirements

These items must be validated before a production migration recommendation is finalized.

## AWS Academy Activity: Actions and Results

### Evidence Entry 1 — RDS Discovery

**Action or command:**

```bash
aws rds describe-db-instances \
  --region us-east-1 \
  --query 'DBInstances[].{Name:DBInstanceIdentifier,Engine:Engine,Status:DBInstanceStatus}' \
  --output table
```

**Actual result:**

```text
Name: database-1
Engine: sqlserver-ex
Status: available
```

**Interpretation:**

This confirmed that Amazon RDS was accessible in the Learner Lab and that one RDS instance was available at the time of the check.

The result does not prove that the café's MySQL workload has been migrated or that the observed SQL Server database is compatible with the café application.

### Evidence Entry 2 — RDS Console Inspection

**Console action:**

```text
AWS Console → Amazon RDS → Databases → Create database
```

**Actual result:**

I observed MySQL as an available relational database engine and reviewed options for deployment, availability, instance configuration, storage, connectivity, and backups.

**Interpretation:**

This demonstrated that RDS provides managed MySQL as an option and can manage more of the underlying database infrastructure.

Viewing the configuration page alone does not prove migration compatibility, performance, cost, or successful application connectivity.

### Evidence Entry 3 — DynamoDB Discovery

**Action or command:**

```bash
aws dynamodb list-tables \
  --region us-east-1 \
  --output table
```

**Actual result:**

```text
------------
|ListTables|
+----------+
```

No DynamoDB tables were returned.

**Interpretation:**

The DynamoDB request completed successfully but returned no tables.

This confirms the service was reachable and no tables were returned during the investigation. It does not mean DynamoDB failed, and it does not prove DynamoDB is the correct database model for the café.

### Final Verification

I reran the RDS discovery command and reviewed the final database status.

The final result showed:

```text
database-1
sqlserver-ex
available
```

I also verified the DynamoDB table list and received an empty result.

These checks confirmed that AWS database discovery was working in the Learner Lab.

I did not verify a production MySQL migration, schema conversion, application connectivity, migration downtime, performance, transaction volume, recovery objectives, or cost.

Those items remain unresolved and would require additional testing and supervisor approval before any production change.

## Operational Analysis

The café's existing workload is relational and already uses MySQL, structured tables, SQL queries, and transactions.

That evidence makes a managed relational database the most natural option to evaluate first.

Running MySQL directly on EC2 provides more server-level control, but it also requires the client to manage the guest operating system, database software, patching, backups, storage, and recovery.

Amazon RDS for MySQL could reduce many of those infrastructure and maintenance responsibilities while allowing the application to continue using a relational MySQL database.

DynamoDB is better suited to workloads designed around known keys and NoSQL access patterns. It should not be treated as a direct replacement for this café workload because the application depends on relational behavior.

Aurora could also be evaluated because it supports MySQL-compatible workloads, but it should not be selected automatically. HarborTech would first need evidence showing that its availability, scaling, or performance benefits justify the additional migration considerations and cost.

The current evidence supports evaluating RDS for MySQL first while keeping the existing EC2 database in place until testing and approval are complete.

## Recommendation

I recommend that HarborTech evaluate Amazon RDS for MySQL as the first modernization option.

The existing application already uses MySQL, relational tables, SQL queries, and transactions. RDS for MySQL would allow the café to preserve that database model while reducing some of the operational burden associated with running MySQL directly on EC2.

AWS could manage more of the underlying database infrastructure, patching, storage platform, backup capabilities, recovery features, and availability options.

HarborTech and the client would still be responsible for:

- Data
- Database schema
- Users and permissions
- Application queries
- Security configuration
- Application integration
- Monitoring decisions
- Workload requirements
- Migration testing

I would not recommend DynamoDB as a direct replacement because it uses a different data model and could require significant application redesign.

Aurora may also be worth evaluating, but I would first validate application compatibility, transaction volume, availability needs, recovery objectives, performance requirements, migration complexity, and budget.

Before any production migration, HarborTech should perform compatibility testing, confirm backup and recovery requirements, test application connectivity, estimate cost, create a rollback plan, and document the proposed cutover process.

This recommendation is for evaluation only. It does not authorize a production migration.

## Escalation Notes

As a HarborTech Cloud Operations Intern, I can investigate the workload, compare AWS database services, document evidence, and recommend a next step.

I should not independently approve or perform a production database migration.

The following actions require approval and escalation:

- Production database migration
- Production cutover
- Database engine changes
- Schema conversion
- Production security-group changes
- Production database credentials or permissions changes
- Network connectivity changes
- Multi-AZ or availability architecture changes
- Production backup policy changes
- Moving production data
- Final architecture approval

These decisions should be reviewed by the appropriate HarborTech supervisor, database administrator, system owner, security team, and change-control authority.

## Lessons Learned

Week 7 showed that database decisions should start with the workload instead of choosing a service because it is popular or fully managed.

A relational application using MySQL, SQL queries, structured tables, and transactions is naturally different from a NoSQL workload designed around key-based access.

Amazon RDS can reduce infrastructure and maintenance work, but moving to a managed service does not remove all customer responsibilities.

The client still owns the data, schema, permissions, application behavior, security decisions, monitoring, and migration planning.

I also learned that DynamoDB should not be considered a direct replacement for every relational database. Its NoSQL model can be a better fit for workloads that do not require joins or relational transactions.

Aurora also should not automatically be chosen just because it is a managed AWS database. Compatibility, performance, availability requirements, migration complexity, and cost should be validated first.

Finally, actual AWS evidence is important. Empty lists, configuration screens, successful commands, and even errors such as using an incorrect Region placeholder all provide useful troubleshooting information when they are documented accurately.

## Professional Vocabulary

### Relational Database

A relational database organizes information into structured tables and supports relationships, queries, and transactions.

### NoSQL

NoSQL refers to database models that do not rely on the traditional relational table structure. These databases are often designed around specific access patterns and scalability needs.

### Amazon RDS

Amazon RDS is an AWS managed relational database service that reduces the amount of infrastructure and database server administration required from the customer.

### Amazon Aurora

Amazon Aurora is an AWS managed relational database engine that is compatible with MySQL or PostgreSQL and is designed for managed availability and performance.

### Amazon DynamoDB

Amazon DynamoDB is a managed NoSQL database service that stores data using key-based access patterns.

### Multi-AZ

Multi-AZ is an availability approach that places database resources across Availability Zones to improve resilience for supported configurations.

### Database Migration Service

AWS Database Migration Service helps move database data between supported source and target database systems.

### Schema Conversion

Schema conversion is the process of changing database structures or database objects so they can work with a different target database engine.

### Managed Service

A managed service is a cloud service where AWS operates more of the underlying infrastructure while the customer continues to manage the application, data, configuration, and security responsibilities that remain under the shared responsibility model.

### Workload Fit

Workload fit means choosing a technology based on the actual technical and operational requirements of the application rather than choosing a service simply because it is popular or available.
