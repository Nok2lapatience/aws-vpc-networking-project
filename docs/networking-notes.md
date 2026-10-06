# AWS VPC Networking Notes

## 1. VPC

A **Virtual Private Cloud (VPC)** is a logically isolated network within AWS where resources such as EC2 instances can be deployed.

### Project Configuration

```text
VPC Name: Lab VPC
CIDR Block: 10.0.0.0/16
```

The VPC provides the overall network boundary for the project.

---

## 2. Subnet Design

The VPC was divided into two subnets:

### Public Subnet

```text
Name: Public Subnet
CIDR: 10.0.0.0/24
```

The public subnet is designed for resources that require a route to the Internet Gateway.

The Bastion Server was deployed in this subnet.

### Private Subnet

```text
Name: Private Subnet
CIDR: 10.0.2.0/23
```

The private subnet is designed for resources that should not be directly reachable from the internet.

The private EC2 instance was deployed in this subnet.

---

## 3. Internet Gateway

An **Internet Gateway (IGW)** provides a connection between the VPC and the internet.

The Internet Gateway was attached to the VPC:

```text
Lab VPC
   │
   ▼
Internet Gateway
   │
   ▼
Internet
```

The public route table uses the Internet Gateway as its target for internet-bound traffic.

---

## 4. Route Tables

Route tables determine where network traffic is directed.

### Public Route Table

The public route table was associated with the Public Subnet.

```text
Destination: 0.0.0.0/0
Target: Internet Gateway
```

Traffic destined for the internet is therefore sent through the Internet Gateway.

```text
Public Subnet
     │
     ▼
Public Route Table
     │
     ▼
Internet Gateway
     │
     ▼
Internet
```

---

### Private Route Table

The private route table was associated with the Private Subnet.

```text
Destination: 0.0.0.0/0
Target: NAT Gateway
```

The private instance does not require a direct public route to the internet.

Instead, outbound traffic is sent through the NAT Gateway.

```text
Private EC2
     │
     ▼
Private Route Table
     │
     ▼
NAT Gateway
     │
     ▼
Internet Gateway
     │
     ▼
Internet
```

---

## 5. NAT Gateway

A **NAT Gateway** allows resources in a private subnet to initiate connections to the internet without requiring those resources to have public IP addresses.

In this project, the NAT Gateway was deployed in the Public Subnet.

```text
NAT Gateway
    │
    ├── Located in Public Subnet
    │
    └── Uses Elastic IP
```

The Private Route Table directs internet-bound traffic to the NAT Gateway.

---

## 6. Bastion Host

A **Bastion Host**, also known as a jump server, provides an access point for connecting to resources located in a private subnet.

The Bastion Server was deployed in the Public Subnet.

```text
User
 │
 ▼
Bastion Server
Public Subnet
 │
 ▼
Private EC2
Private Subnet
```

This demonstrates how a public-facing access point can be used to reach a private resource.

---

## 7. Private EC2 Instance

The private EC2 instance was deployed in the Private Subnet.

The instance was not directly exposed to the public internet.

Connectivity to the instance was achieved through the Bastion Server.

The private instance was also configured to use the NAT Gateway for outbound internet connectivity.

---

## 8. Connectivity Testing

After connecting to the private EC2 instance through the Bastion Server, outbound internet connectivity was tested.

Command used:

```bash
ping -c 3 amazon.com
```

Successful responses confirmed that the private instance was able to communicate with the internet through the configured NAT Gateway.

---

## 9. Network Traffic Flow

### Public Traffic

```text
Internet
   │
   ▼
Internet Gateway
   │
   ▼
Public Route Table
   │
   ▼
Public Subnet
   │
   ▼
Bastion Server
```

### Private Outbound Traffic

```text
Private EC2
   │
   ▼
Private Route Table
   │
   ▼
NAT Gateway
   │
   ▼
Internet Gateway
   │
   ▼
Internet
```

### Private Access

```text
User
   │
   ▼
Bastion Server
   │
   ▼
Private EC2
```

---

## 10. Key Networking Concepts

### CIDR

CIDR notation defines the IP address range available within a network.

Project VPC:

```text
10.0.0.0/16
```

Public subnet:

```text
10.0.0.0/24
```

Private subnet:

```text
10.0.2.0/23
```

---

### Public Subnet

A subnet is considered public when its route table contains a route to an Internet Gateway.

---

### Private Subnet

A private subnet does not have a direct route from the subnet to an Internet Gateway.

In this project, internet-bound traffic from the private subnet is routed through the NAT Gateway.

---

### Internet Gateway vs NAT Gateway

| Component           | Purpose                                                                      |
| ------------------- | ---------------------------------------------------------------------------- |
| Internet Gateway    | Provides internet connectivity for resources with appropriate public routing |
| NAT Gateway         | Allows private resources to initiate outbound internet connections           |
| Public Route Table  | Routes public-subnet traffic toward the Internet Gateway                     |
| Private Route Table | Routes private-subnet internet traffic toward the NAT Gateway                |

---

## 11. Security Considerations

The architecture separates resources according to their network exposure.

The Bastion Server is placed in the Public Subnet, while the application/private workload is placed in the Private Subnet.

For production environments, additional security controls should be implemented, including:

* Restricting security-group rules to trusted sources
* Avoiding unnecessary public IP addresses
* Using AWS Systems Manager Session Manager where appropriate
* Applying least-privilege IAM policies
* Enabling VPC Flow Logs
* Monitoring network activity
* Deploying across multiple Availability Zones for higher availability

---

## 12. Skills Practiced

This project provided hands-on practice with:

* AWS VPC
* IPv4 CIDR addressing
* Subnet design
* Public and private networking
* Route tables
* Internet Gateway
* NAT Gateway
* EC2 networking
* Bastion Host architecture
* Private-instance connectivity
* Linux networking commands
* AWS infrastructure troubleshooting

---

## 13. Future Improvements

The next version of this architecture could include:

1. Multiple Availability Zones
2. Additional private subnets
3. VPC Flow Logs
4. AWS Systems Manager Session Manager
5. More restrictive security-group rules
6. Infrastructure as Code using Terraform
7. Automated deployment using CI/CD
8. CloudWatch monitoring and alerting

This would move the project closer to a production-oriented AWS network architecture.

