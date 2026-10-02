# Week 5: Scaling, Load Balancing, and DNS

## HarborTech Ticket Summary

Riverside Goods is preparing for a seasonal promotion that is expected to cause a large increase in website traffic. The current application depends on one EC2 instance and one public endpoint, which creates both a capacity risk and a single point of failure. HarborTech needs to review the available AWS evidence and recommend ways to improve scaling, traffic distribution, and availability.

## Client Impact

If traffic increases beyond what the current EC2 instance can handle, customers could experience slow response times, errors, or service outages. Since the application also depends on one public endpoint, a failure of that endpoint could make the application unavailable even if the server itself is working. This could affect customers during the promotion and cause lost business.

## Provided Ticket Evidence

The ticket states that previous promotion traffic reached 92% CPU utilization and that the upcoming promotion is expected to double request volume.

The proposed Auto Scaling configuration uses:

- Minimum capacity: 2
- Desired capacity: 2
- Maximum capacity: 6

The ticket also states that the proposed Application Load Balancer reports both test targets as healthy.

A secondary recovery endpoint is available, but Route 53 failover has not been confirmed.

These details were provided by the ticket and were not personally verified in my AWS Learner Lab.

## AWS Commands Used

I used the following AWS CLI commands in CloudShell:

```bash
aws autoscaling describe-auto-scaling-groups --query 'AutoScalingGroups[].{Name:AutoScalingGroupName,Min:MinSize,Desired:DesiredCapacity,Max:MaxSize}' --output table

aws elbv2 describe-target-groups --query 'TargetGroups[].{Name:TargetGroupName,TargetGroupArn:TargetGroupArn,Protocol:Protocol,Port:Port}' --output table

aws route53 list-hosted-zones --query 'HostedZones[].{Name:Name,Id:Id,Private:Config.PrivateZone}' --output table
```

## AWS Evidence Collected

The Auto Scaling command completed successfully but returned no Auto Scaling groups in my assigned AWS Learner Lab.

The target group command completed successfully but returned no target groups in my assigned AWS Learner Lab. Because no target group was returned, I stopped the investigation and did not run the target health command.

For Route 53, I executed the hosted zone command in my assigned AWS Learner Lab.

**Route 53 Result:** REPLACE THIS SENTENCE WITH THE ACTUAL RESULT FROM YOUR AWS LAB.

## Virtualization Connection

Virtualization allows cloud servers to be created and replaced without depending on one physical machine. Auto Scaling can increase or decrease the number of EC2 instances based on demand. A load balancer can then distribute client requests across multiple healthy instances instead of sending all traffic to one server.

This makes virtual compute capacity more flexible and helps reduce the risk of depending on a single server.

## Operational Analysis

The ticket evidence shows that Riverside Goods may need more capacity because previous traffic reached 92% CPU utilization and the next promotion is expected to increase demand even more.

The proposed 2/2/6 Auto Scaling configuration would keep at least two instances running, normally maintain two, and allow the environment to scale up to six instances when demand increases. However, my AWS Learner Lab did not return an Auto Scaling group, so I could not personally verify this proposed configuration.

Adding more instances alone does not distribute client traffic. An Application Load Balancer is needed to distribute incoming requests across available targets. A target group identifies which resources can receive traffic, and health checks help determine whether those targets should continue receiving requests.

My AWS environment did not return a target group, so I could not personally verify the ticket statement that both test targets were healthy.

Healthy application targets also do not prove that DNS failover is working. Route 53 failover would need separate DNS records, routing configuration, primary and secondary endpoints, and health-check evidence before HarborTech could confirm that failover is ready.

## Recommendation

HarborTech should recommend an Auto Scaling configuration that provides additional EC2 capacity when traffic increases. The proposed minimum of 2, desired capacity of 2, and maximum of 6 could help provide additional capacity during the seasonal promotion.

An Application Load Balancer should also be used to distribute client traffic across healthy targets instead of relying on a single EC2 instance.

Route 53 failover should not be considered ready until the primary and secondary DNS records, failover routing policy, recovery endpoint, and health checks have been verified.

After implementation, HarborTech should monitor CPU utilization, the number of running instances, Auto Scaling activity, load balancer target health, traffic distribution, and Route 53 health-check status.

## Escalation Notes

Any production changes involving Auto Scaling, load balancers, Route 53 records, health checks, or DNS failover should be reviewed and approved by the appropriate administrator.

As a junior cloud operations intern, my responsibility is to investigate the available evidence, document my findings, recommend the appropriate controls, and escalate production changes for approval.

## Lessons Learned

This investigation showed me that scaling, load balancing, health checks, and DNS failover solve different operational problems.

Auto Scaling helps provide additional capacity when demand changes. A load balancer distributes traffic across available resources. Target health checks help prevent requests from being sent to unhealthy targets. Route 53 can control DNS routing and support failover, but it must be configured and verified separately.

I also learned that cloud operations decisions should be based on actual evidence. If a resource does not appear in my AWS environment, I should document that result instead of assuming that it exists.

## Professional Vocabulary

### Elasticity
The ability of a cloud environment to increase or decrease resources as demand changes.

### Scalability
The ability of a system to handle increased workload by adding or improving resources.

### Load Balancer
A service that distributes incoming client traffic across multiple available targets.

### Target Group
A collection of resources, such as EC2 instances, that can receive traffic from a load balancer.

### Health Check
A test used to determine whether a target or endpoint is healthy and able to receive traffic.

### Auto Scaling Group
A group of EC2 instances that can automatically maintain or adjust the number of running instances.

### Launch Template
A reusable configuration that defines how EC2 instances should be launched.

### Desired Capacity
The number of instances an Auto Scaling group attempts to keep running under normal conditions.

### Minimum Capacity
The lowest number of instances an Auto Scaling group is allowed to maintain.

### Maximum Capacity
The highest number of instances an Auto Scaling group is allowed to run.

### Route 53
AWS's DNS service used to route users to application endpoints.

### Failover
The process of directing traffic to a secondary resource when the primary resource becomes unavailable.
