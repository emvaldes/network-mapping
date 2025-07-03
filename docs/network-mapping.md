# Network Mapping Automation Project

## Project Overview

The **Network Mapping Automation Project** is a comprehensive solution designed to assist cloud engineers, DevOps practitioners, and network architects in the inspection, documentation, and verification of complex AWS network infrastructures. It provides a practical, automated framework for analyzing and validating network configurations across a multi-VPC environment. This project includes two main components that work together to simulate, deploy, and thoroughly map cloud-based networking topologies:

1. **CloudFormation Template (****`network-mapping.yaml`****):**

   * Deploys a sample AWS environment that simulates a multi-VPC topology, complete with Transit Gateways, NAT Gateways, Internet Gateways, and VPC peering connections.
   * Intended as a testbed to prototype, test, and analyze networking setups in a controlled environment. It is **not recommended for production deployments**.
   * Provides the complexity required to fully exercise the network-mapping shell script and generate rich data sets for analysis and reporting.

2. **Network-Mapping Shell Script (****`network-mapping.shell`****):**

   * Serves as the core of the project, orchestrating all discovery and mapping operations.
   * Automates a wide range of AWS API calls to discover and document your AWS network architecture in real-time.
   * Collects and organizes detailed data on subnets, VPCs, route tables, gateways, security groups, VPC endpoints, EC2 instances, Elastic IPs, and much more.
   * Generates structured, timestamped JSON reports, creating a reproducible and complete snapshot of your AWS networking environment at a given point in time.
   * Designed to replace error-prone manual AWS Console inspections with **automated, repeatable infrastructure discovery processes**.
   * Can be run interactively or integrated seamlessly into CI/CD pipelines to validate deployments as part of a continuous delivery process.

---

## What the Script Actually Does

The script performs a thorough and structured network discovery process, including but not limited to the following operations:

* Enumerates **subnets**, including metadata, availability zones, and associated route tables.
* Maps **VPC configurations**, pulling details like DNS support, DHCP options, and network CIDR blocks.
* Extracts comprehensive **route table data**, covering both default and custom routes for all route tables in scope.
* Identifies and documents **Internet Gateways, NAT Gateways, and Transit Gateways**, along with their attachments.
* Inspects **Transit Gateway attachments, route tables, and route propagations**, creating a full picture of transit routing flows.
* Detects **VPC peering connections**, validating their usage in route propagation and transitive communication paths.
* Lists all **VPC endpoints**, detailing interface and gateway types, attachment status, and service associations.
* Inventories **security groups**, clearly separating attached and unattached groups to highlight potential cleanup candidates.
* Verifies **Elastic IP allocations**, mapping them to EC2 instances, NAT Gateways, and network interfaces.
* Identifies **SSM-managed EC2 instances**, ensuring proper IAM role attachment and tag compliance.
* Analyzes **reachability between public and private subnets**, particularly from bastion hosts or through NAT.
* Maps **interface endpoint connections**, confirming health and coverage of AWS-managed services within the private network.
* Captures metadata for **Elastic Network Interfaces (ENIs)**, **DHCP Options**, and **prefix lists** used in routing and security.
* Extracts **Transit Gateway route advertisements and propagations**, offering insight into cross-account and multi-region propagation.
* Builds a detailed **private reachability map** to validate isolation boundaries for sensitive subnets.

### How It Works

For each subnet defined in the **JSON input file (****`network-mapping.json`****)**, the script performs a hierarchical and recursive analysis:

1. Resolves the subnet's SubnetId and VpcId using the Name tag, or falls back to CIDR block matching if the Name tag is unavailable.
2. Collects all subnet metadata and saves it to a structured JSON file for later reporting.
3. Uses the associated VPC as an anchor point to recursively discover related infrastructure components like route tables, gateways, and instances.
4. Organizes the collected information into per-subnet, per-VPC, and shared folders, eliminating redundancy of shared resources between subnets.
5. Creates a snapshot of your AWS network in its current state, reflecting both static configurations and dynamic service attachments.

### JSON Input Example

The script expects a structured **JSON file** containing the VPCs and their subnets to inspect. Example:

```json
{
  "vpcs": [
    {
      "OwnerId": "***",
      "InstanceTenancy": "default",
      "CidrBlockAssociationSet": [
        {
          "AssociationId": "vpc-cidr-assoc-0533fb09faee7e675",
          "CidrBlock": "10.0.0.0/16",
          "CidrBlockState": {
            "State": "associated"
          }
        }
      ],
      "IsDefault": false,
      "Tags": [
        {
          "Key": "DevOps",
          "Value": "true"
        },
        {
          "Key": "Name",
          "Value": "DevOps-PublicVPC"
        },
        {
          "Key": "aws:cloudformation:stack-name",
          "Value": "netowrk-mapping"
        },
        {
          "Key": "aws:cloudformation:stack-id",
          "Value": "arn:aws:cloudformation:***:***:stack/netowrk-mapping/b0364fb0-5ac2-11f0-bcb5-1213df58d077"
        },
        {
          "Key": "aws:cloudformation:logical-id",
          "Value": "PublicVPC"
        }
      ],
      "BlockPublicAccessStates": {
        "InternetGatewayBlockMode": "off"
      },
      "VpcId": "vpc-093dac6c9f6900507",
      "State": "available",
      "CidrBlock": "10.0.0.0/16",
      "DhcpOptionsId": "dopt-b1e813ca",
      "Subnets": [
        {
          "AvailabilityZoneId": "use1-az1",
          "MapCustomerOwnedIpOnLaunch": false,
          "OwnerId": "***",
          "AssignIpv6AddressOnCreation": false,
          "Ipv6CidrBlockAssociationSet": [],
          "Tags": [
            {
              "Key": "aws:cloudformation:logical-id",
              "Value": "PublicSubnet"
            },
            {
              "Key": "aws:cloudformation:stack-id",
              "Value": "arn:aws:cloudformation:***:***:stack/netowrk-mapping/b0364fb0-5ac2-11f0-bcb5-1213df58d077"
            },
            {
              "Key": "aws:cloudformation:stack-name",
              "Value": "netowrk-mapping"
            },
            {
              "Key": "Name",
              "Value": "DevOps-PublicSubnet"
            },
            {
              "Key": "DevOps",
              "Value": "true"
            }
          ],
          "SubnetArn": "arn:aws:ec2:***:***:subnet/subnet-0c22020068931f068",
          "EnableDns64": false,
          "Ipv6Native": false,
          "PrivateDnsNameOptionsOnLaunch": {
            "HostnameType": "ip-name",
            "EnableResourceNameDnsARecord": false,
            "EnableResourceNameDnsAAAARecord": false
          },
          "BlockPublicAccessStates": {
            "InternetGatewayBlockMode": "off"
          },
          "SubnetId": "subnet-0c22020068931f068",
          "State": "available",
          "VpcId": "vpc-093dac6c9f6900507",
          "CidrBlock": "10.0.1.0/24",
          "AvailableIpAddressCount": 248,
          "AvailabilityZone": "***",
          "DefaultForAz": false,
          "MapPublicIpOnLaunch": true
        }
      ]
    }
  ]
}
```

---

## Why This Matters

* Removes the time-consuming and error-prone process of clicking through the AWS Console for network discovery.
* Provides **reproducible, timestamped network snapshots** that serve as auditable evidence of your AWS network configurations.
* Validates that your **Infrastructure-as-Code (IaC) deployments** produce the correct network architecture and interconnections.
* Accelerates troubleshooting and design validation by providing a machine-readable map of your cloud network.
* Forms the foundation for **real-time visualizations** of your network infrastructure, such as Lucidchart or other diagram-as-code solutions.

---

## About the CloudFormation Template (`network-mapping.yaml`)

### Purpose

* Deploys a simulated but realistic AWS network environment suitable for use with the mapping script.
* Includes multiple AWS networking constructs:

  * Public, Private, and DevOps VPCs with isolated routing domains.
  * Transit Gateway attachments for inter-VPC and potential cross-region routing.
  * VPC peering between the public and private VPCs for direct routing paths.
  * A NAT Gateway with an Elastic IP to provide outbound internet access to private subnets.
  * EC2 instances with SSM integration for secure management without SSH.
  * VPC interface and gateway endpoints to emulate private service access patterns.

### Deployment Methods

#### Recommended: Change Sets

Change sets allow you to preview all infrastructure changes before applying them, minimizing risk during deployment.

1. **Create the Change Set:**

   * Use the AWS CLI or Management Console to define the proposed stack changes.

2. **Review the Change Set:**

   * Carefully inspect the resources to be created or modified and validate them against your network architecture.

3. **Execute the Change Set:**

   * Apply the changes to deploy the full stack.

#### Alternate: Direct Deployment

* For quick prototypes or testing, deploy the template directly via the AWS Console or CLI.
* Change Sets are still preferred for reviewable, auditable deployments.

---

## Script Technical Reference

### CLI Arguments

```
./network-mapping.shell <region> <aws_profile>
```

* `region`: The AWS region to target (e.g., `us-east-1`).
* `aws_profile`: The AWS CLI profile to use for API calls (e.g., `default`, `controller`).

### Key Components Collected

| Component                         | Description                                                 |
| --------------------------------- | ----------------------------------------------------------- |
| `subnet.json`                     | Subnet configuration and metadata                           |
| `vpc.json`                        | VPC-level configuration and tags                            |
| `route_tables_all.json`           | All route tables defined in the VPC                         |
| `route_table_subnet.json`         | Route table specifically associated with the subnet         |
| `default_route_check.json`        | Checks for default routes (0.0.0.0/0)                       |
| `igw.json`                        | Internet Gateway configuration                              |
| `nat_gateways.json`               | NAT Gateway configuration                                   |
| `elastic_ips.json`                | Elastic IP allocations and their attachments                |
| `security_groups.json`            | Security Groups and rules                                   |
| `security_groups_unattached.json` | Security Groups not currently in use                        |
| `network_acls.json`               | Network ACL configurations                                  |
| `tgw_attachments.json`            | Transit Gateway attachment metadata                         |
| `transit_gateways.json`           | Transit Gateway configuration and status                    |
| `tgw_routes_*.json`               | Routes in each Transit Gateway route table                  |
| `vpc_peering.json`                | VPC peering connections and metadata                        |
| `vpc_endpoints.json`              | VPC endpoint configurations and service links               |
| `endpoint_connection_*.json`      | Status of interface endpoint connections                    |
| `dhcp_options.json`               | DHCP Options Set attached to the VPC                        |
| `network_interfaces.json`         | Elastic Network Interfaces (ENIs)                           |
| `ec2_instances.json`              | EC2 instances in the subnet                                 |
| `prefix_lists.json`               | AWS-managed prefix lists used in routes and security groups |
| `private_reachability_map.json`   | Private subnet reachability information from bastion or SSM |

### Network Relationship Mapping

* Builds a complete view of network interconnections and dependencies, including:

  * Default route destinations (IGW, NAT Gateway, Transit Gateway, or Peering).
  * Transit Gateway route propagation and advertised routes.
  * VPC peering paths and cross-VPC reachability.
  * VPC endpoint exposure across subnets and security groups.
  * Elastic IP usage and their associated network interfaces.
  * Security Groups and Prefix Lists applied at both VPC and subnet levels.

---

## Where This Is Going

* Build **Lucidchart diagrams** and other visualizations directly from the generated JSON reports.
* Develop a **real-time infrastructure visualization engine** that reads these mappings and shows network topology changes over time.
* Integrate the entire discovery process into CI/CD pipelines to **validate network architectures post-deployment**.
* Lay the groundwork for automated drift detection and compliance checks across your AWS network resources.

---

## When to Use This Project

* **Post-deployment validation:** Ensure that your IaC deployments have correctly provisioned and connected network resources.
* **On-demand inspection:** Empower engineers to gain quick, deep insights into the network configuration without AWS Console navigation.
* **Audit preparation:** Capture structured network reports to prepare for compliance reviews and audits.
* **Training/testing:** Use the CloudFormation template to spin up complex AWS networking environments for hands-on learning and troubleshooting exercises.

---
