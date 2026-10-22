# AWS VPC Peering — Cross-VPC Private Communication

**Project:** EWB AWS Internship
**Cloud Platform:** Amazon Web Services (AWS)
**Region:** US East (Ohio) — us-east-2
**Service Category:** Networking & Security
**Project Type:** Hands-on Cloud Infrastructure Project

## 1. Project Overview

This project demonstrates how to establish private network connectivity between two Amazon Virtual Private Clouds (VPCs) using AWS VPC Peering.

Two VPCs were configured with separate IPv4 CIDR blocks. Each VPC contained an EC2 instance, and route tables and security group rules were configured to support communication between the private IP addresses of the instances.

The project also explored SSH connectivity, private IP communication, route configuration and connectivity testing from the EC2 command line.

## 2. Problem Statement

AWS resources deployed in separate VPCs are isolated by default. Applications running in different VPCs may need to communicate privately without relying on public IP addresses for communication between those networks.

The objective of this project was to establish a VPC peering connection, configure the required routes and security rules, and validate private communication between EC2 instances in two VPCs.

## 3. Project Objectives

- Create and configure two VPCs with distinct CIDR ranges.
- Configure public and private subnet resources where required.
- Establish a VPC peering connection between the VPCs.
- Update both VPC route tables to direct traffic to the peer network through the peering connection.
- Configure security group rules for required SSH and ICMP traffic.
- Connect to EC2 instances using SSH and test private IP connectivity.
- Validate connectivity from both sides and document the results.

## 4. Architecture and Configuration

### VPC 1 — VPC-A

| Item | Value |
|---|---|
| VPC name / ID | `VPC-A` (`vpc-0ebb6c8c31773064`) |
| VPC CIDR | 28.0.0.0/16 |
| Public subnet | `Public-subnet-A` (`subnet-0e16277dfaf59b08a`) |
| Private subnet | `Private-subnet-A` (`subnet-07448323dd2eac786`) |
| Internet gateway | `igw-06fad86de166ac51c` |
| Public route table | `rtb-public-a` (`rtb-0fd0f64f7e945d910`) |
| Private route table A | `rtb-00b5393eeb4bb44cc` |
| EC2 instance (bastion) | `Bastion-vpc-A` (`i-0713213471bdb4793`) |
| Private IP shown in evidence | 28.0.1.194 |
| Public IP shown during testing | 3.144.103.155 |
| Security group | `Bastion-SG` (`sg-0656999fdb60ae672`) |

### VPC 2 — VPC-B

| Item | Value |
|---|---|
| VPC name / ID | `VPC-B` (`vpc-049a3a3032d0e4f3c`) |
| VPC CIDR | 28.200.0.0/16 |
| Subnet | `subnet-0b05917c357f00745` |
| Private route table B | `rtb-0759ef2502c2e1bf7` |
| EC2 instance | `Remote-app-vpc-B` (`i-0cc4b1af930d4f717`) |
| Private IP shown in evidence | 28.200.1.179 |
| Public IPv4 address | None shown for this instance |
| Security group | `Appserver-SG` (`sg-05e04db63f87f28db`) |

### Default VPC

Default VPC ID: `vpc-08e14feb23b0da827`

### VPC Peering Connection

| Item | Value |
|---|---|
| Connection name | `VPC-A-to-VPC-B` |
| Connection ID | `pcx-00536c471c42e9d73` |
| Region | us-east-2 |
| Observed status | Active |

The AWS console evidence shows the peering connection between the two VPCs and corresponding routes to the peer CIDR blocks.

**Architecture note:** VPC peering is a private, one-to-one network connection. It does not automatically configure routes or permit traffic; the appropriate route table entries and security rules are also required.

## 5. Implementation Steps

### Step 1 — Configure the VPCs

Created two VPCs with separate IPv4 CIDR blocks:

- VPC-A: 28.0.0.0/16
- VPC-B: 28.200.0.0/16

The separate address ranges allow the networks to be routed independently.

### Step 2 — Configure Subnets and EC2 Instances

Configured the network resources and EC2 instances used for testing. VPC-A has `Public-subnet-A` and `Private-subnet-A`, with an internet gateway (`igw-06fad86de166ac51c`) attached for the public subnet. The instances are `Bastion-vpc-A` and `Remote-app-vpc-B`.

The bastion instance had a public IPv4 address during SSH testing. The second instance was accessed through the bastion using SSH proxying.

### Step 3 — Create the VPC Peering Connection

Created a VPC peering connection named `VPC-A-to-VPC-B` between the two VPCs in the same AWS region.

The console screenshot records the connection as Active.

### Step 4 — Configure Route Tables

Added routes so that each VPC could direct traffic to the other VPC's CIDR block through the peering connection.

| Network | Destination | Route target |
|---|---|---|
| VPC-A | 28.200.0.0/16 | VPC peering connection (`pcx-00536c471c42e9d73`) |
| VPC-B | 28.0.0.0/16 | VPC peering connection (`pcx-00536c471c42e9d73`) |

The screenshots show these routes with an Active status. The VPC-A public route table (`rtb-public-a`) also contains its local route and a default route through the internet gateway.

### Step 5 — Configure Security Groups

Configured the EC2 security groups (`Bastion-SG` and `Appserver-SG`) to support the traffic required for testing.

The supplied console evidence shows SSH rules on TCP port 22 and ICMP rules for connectivity testing. One security group also shows an SSH source restricted to a single IPv4 address using /32.

Security group sources should be restricted to the appropriate trusted IP address or network CIDR. Avoid exposing SSH to the entire internet unless there is a specific, temporary requirement.

### Step 6 — Connect Using SSH

Used Windows PowerShell and SSH key files to connect to the bastion instance.

A proxy connection was then used to access `Remote-app-vpc-B` through the bastion. This demonstrates a bastion-style access pattern without requiring a public IPv4 address on the second instance.

**Important:** Private key files must remain local and must never be committed to a public GitHub repository.

## 6. Connectivity Testing and Results

The supplied terminal screenshots demonstrate the following:

| Test | Observed result |
|---|---|
| TCP connectivity to the bastion (3.144.103.155) on port 22 | TcpTestSucceeded: True |
| SSH connection to `Bastion-vpc-A` | Successful |
| SSH proxy connection to `Remote-app-vpc-B` | Successful |
| Ping from 28.0.1.194 to 28.200.1.179 | 4 packets received, 0% packet loss |
| Initial ping from 28.0.1.194 to 28.200.1.179 in another session | 100% packet loss |
| Later ping from 28.0.1.194 to 28.200.1.179 | 4 packets received, 0% packet loss |
| Ping from 28.200.1.179 to 28.0.1.194 | 4 packets received, 0% packet loss in the later screenshot |

The initial failed ping followed by a successful test is useful evidence that connectivity troubleshooting and verification were part of the exercise.

Successful ping tests demonstrate ICMP reachability at the time of testing. They do not, by themselves, verify every application protocol or prove that all security controls are configured correctly.

## 7. Troubleshooting Considerations

If instances cannot communicate across a VPC peering connection, check:

- The peering connection is active.
- Both VPCs use non-overlapping CIDR blocks.
- Each relevant route table contains a route to the other VPC's CIDR.
- Security groups permit the required source, protocol and port.
- Network ACLs permit the required inbound and outbound traffic.
- The destination instance and its operating system allow the traffic being tested.
- SSH key permissions, usernames and proxy settings are correct.

A failed ping does not always mean the peering connection is broken. ICMP may be blocked by a security group, network ACL or operating-system firewall.

## 8. Key Learning Outcomes

- AWS VPC architecture and IPv4 CIDR planning.
- VPC peering connection configuration.
- Route table configuration for private network communication.
- EC2 instance access using SSH.
- Bastion-style SSH access and proxy connections.
- Security group rules for SSH and ICMP.
- Connectivity verification using PowerShell and Linux terminal commands.
- Troubleshooting network reachability and documenting cloud infrastructure.

## 9. Limitations and Future Improvements

- The setup was created as a hands-on learning environment, not a production-ready enterprise network.
- The public IP and instance state shown in screenshots reflect the environment at the time of testing and may change.
- Additional validation would be required for application-level connectivity, monitoring, high availability and production security.
- Future improvements could include private access through AWS Systems Manager Session Manager, tighter security group rules, centralized logging, infrastructure as code and automated connectivity checks.
- Network architecture and address ranges should be reviewed before using the configuration in a production environment.

## 10. Evidence and Screenshots

Evidence images are stored in `03_Evidence_Screenshots/`. Use the actual supplied screenshots; do not create replacement screenshots that imply a result was observed when it was not.

- `01_VPC_Peering_Active.png` — Active VPC peering connection.
- `02_VPC1_Route_Table.png` — VPC-A route table.
- `03_VPC2_Route_Table.png` — VPC-B route table.
- `04_Third_Route_Table.png` — Additional/third route table.
- `05_VPC1_Security_Group.png` — `Bastion-SG` rules.
- `06_VPC2_Security_Group.png` — `Appserver-SG` rules.
- `07_VPC2_Security_Group_SSH_Rule.png` — SSH rule for VPC-B.
- `08_EC2_Resource_Details_VPC1.png` — EC2 resource details for VPC-A.
- `09_EC2_Resource_Details_VPC2.png` — EC2 resource details for VPC-B.
- `10_Forward_Ping_Success.png` — Successful forward-direction ICMP test.
- `11_Reverse_Ping_Success.png` — Successful reverse-direction ICMP test.
- `12_Private_SSH_App_B.png` — Private SSH access evidence for App B.
- `13_Controlled_ICMP_Failure.png` — Controlled ICMP failure during troubleshooting.
- `14_Available_Before_Cleanup.png` — EC2 instances available before cleanup.
- `15_Cleanup_Evidence.png` — Manually redacted cleanup/termination evidence.
- `16_ICMP_Recovery_Success.png` — Successful ICMP test after troubleshooting.
- `17_Architecture_and_Resource_Inventory.png` — Architecture and resource inventory evidence.

## 11. Repository Structure

```
EWB-AWS-Internship/
├── README.md
└── 03_Evidence_Screenshots/
    ├── 01_VPC_Peering_Active.png
    ├── ...
    └── 17_Architecture_and_Resource_Inventory.png
```

## Conclusion

This project provided practical experience in connecting separate AWS VPCs through VPC peering. By configuring routes, security group rules and SSH access, I demonstrated private IP communication between EC2 instances and practised troubleshooting connectivity issues.

The project strengthened my understanding of AWS networking fundamentals, routing, access control and cloud infrastructure validation.
