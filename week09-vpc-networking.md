# Week 9: VPC Networking and Troubleshooting

## HarborTech Ticket Summary

TKT-2026-0009 is a VPC networking investigation involving an internet client that must reach an EC2 web application using TCP port 80.

The EC2 instance is running, the web service is listening on port 80, and the instance has a public IPv4 address. However, the provided network evidence does not prove that the subnet has a valid route to the internet.

The goal is to identify the first unsupported assumption, determine the most likely blocking layer, and recommend the smallest supported correction without changing unrelated network controls.

## Client Impact

Incorrect routing or network configuration can make a working application unavailable to customers.

An EC2 instance can be running and the web server can be listening correctly, but customers still cannot reach it if the subnet does not have a valid path to the internet.

Changing unrelated security settings could create additional problems without fixing the actual cause. For that reason, the network path should be checked layer by layer before making changes.

## AWS Services Involved

### Amazon VPC

Amazon VPC provides the software-defined network where the AWS resources are located.

### Subnets

Subnets divide the VPC address space into smaller network segments. Public and private subnets have different routing requirements.

### Route Tables

Route tables determine where network traffic is sent. A public subnet normally requires a valid route for internet-bound traffic.

### Internet Gateway

An Internet Gateway connects a VPC to the internet when the correct routing and addressing are also present.

### NAT Gateway

A NAT Gateway can provide outbound internet connectivity for resources in a private subnet without allowing unsolicited inbound internet connections directly to those resources.

### Security Groups

Security groups are stateful traffic controls attached to resources such as EC2 instances.

### Network ACLs

Network ACLs are subnet-level traffic controls that evaluate inbound and outbound network traffic.

### CIDR Blocks

CIDR blocks define IP address ranges for VPCs, subnets, routes, and traffic rules.

### Public and Private Subnets

A public subnet requires a valid route to an Internet Gateway for direct internet connectivity. A private subnet normally does not have a direct route to an Internet Gateway for its instances.

### VPC Flow Logs

VPC Flow Logs can provide network traffic information that may help with troubleshooting. No VPC Flow Log evidence was included in the simulated evidence packet.

## Virtualization Connection

Amazon VPC replaces many traditional physical networking components with software-defined configuration.

CIDR blocks, subnets, route tables, Internet Gateways, NAT Gateways, security groups, and network ACLs define how virtual AWS resources communicate.

The networking is virtual, but the same basic networking requirements still apply. Traffic needs a valid source and destination address, route, gateway, security policy, and return path.

## Evidence Reviewed

The simulated evidence packet showed:

- Required path: internet client to EC2 web server using TCP port 80
- EC2 instance state: running
- Web service: listening on TCP port 80
- Public IPv4 address: `198.51.100.24`
- VPC ID: `vpc-0a91`
- VPC CIDR: `10.42.0.0/16`
- Subnet ID: `subnet-0a91`
- Associated route table: `rtb-0a91`
- Route shown: `10.42.0.0/16 → local`
- Internet Gateway: `igw-0a91`
- Internet Gateway attached to the VPC
- Security group: inbound TCP 80 from `0.0.0.0/0` allowed
- Network ACL: inbound TCP 80 allowed
- Network ACL: outbound TCP ports 1024-65535 allowed

The evidence packet only shows selected rules. It does not include a complete configuration or an actual connectivity retest.

## Operational Analysis

The first unsupported assumption is that the application is located in a truly public subnet.

The instance has a public IPv4 address and the VPC has an attached Internet Gateway, but those facts alone do not prove internet reachability.

The subnet's associated route table only contains:

`10.42.0.0/16 → local`

There is no evidence showing:

`0.0.0.0/0 → Internet Gateway`

The security group allows inbound TCP port 80, and the shown network ACL rules allow the required inbound and return traffic. This makes those specific controls less likely to be the blocking point.

However, security rules only allow or deny traffic. They do not create the network route needed for the packets to reach the instance.

Based on the evidence provided, the route table is the most likely blocking layer.

Some uncertainty remains because the evidence packet is incomplete. The full route table, complete network ACL rules, subnet association, flow logs, and actual HTTP connectivity have not been independently verified.

## Recommendation

I recommend verifying the complete route table associated with `subnet-0a91` first.

If the evidence confirms that the default internet route is missing, the smallest supported correction would be to add:

`0.0.0.0/0 → igw-0a91`

to the correct public-subnet route table after receiving approval.

I would not change the security group because the provided evidence already shows that inbound TCP port 80 is allowed.

After the approved routing correction, I would perform a fresh HTTP connectivity test to the EC2 public IPv4 address `198.51.100.24` on port 80.

A successful HTTP response such as `HTTP/1.1 200 OK` would provide evidence that the customer-facing web service is reachable.

## Escalation Notes

As a HarborTech Cloud Operations Intern, I can inspect the configuration, analyze the evidence, identify the likely problem, and recommend a correction.

A production route-table change requires approval before implementation.

Changes involving production Internet Gateways, NAT Gateways, security groups, network ACLs, subnet design, or overall network architecture should also be escalated to the appropriate supervisor or network administrator.

No unrelated security or routing settings should be changed simply because the application is unreachable.

## Lessons Learned

Week 9 showed me that a resource being labeled public does not prove that it actually has internet connectivity.

A public IPv4 address and an attached Internet Gateway are also not enough by themselves. The subnet must have the correct route and the traffic must be allowed by all applicable security controls.

I also learned that troubleshooting should follow the complete network path instead of immediately changing familiar settings such as security groups.

Security groups and network ACLs can allow traffic, but they cannot replace a missing route.

The safest approach is to identify the earliest unsupported part of the path, make the smallest approved correction, and verify the result with a real connectivity test.

## Professional Vocabulary

### VPC

A Virtual Private Cloud is a logically isolated software-defined network in AWS.

### Subnet

A subnet is a smaller IP address range inside a VPC where AWS resources can be placed.

### Route Table

A route table contains rules that determine where network traffic should be sent.

### Internet Gateway

An Internet Gateway provides a connection between a VPC and the public internet when routing and addressing are configured correctly.

### NAT Gateway

A NAT Gateway allows resources in a private subnet to start outbound internet connections without directly accepting unsolicited inbound internet connections.

### Security Group

A security group is a stateful firewall that controls allowed traffic for AWS resources.

### Network ACL

A network ACL is a subnet-level traffic control that evaluates inbound and outbound traffic.

### CIDR

CIDR is a way of defining an IP address range, such as `10.42.0.0/16`.

### Public Subnet

A public subnet is a subnet that has the required routing path to an Internet Gateway for resources that need direct internet connectivity.

### Private Subnet

A private subnet does not provide its instances with a direct internet path through an Internet Gateway. Outbound internet access can use a NAT pattern when required.
