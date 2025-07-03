# Network Mapping Automation

## Project Overview

This project automates the deployment and mapping of an AWS network environment using:

* **CloudFormation**: Creates a test network with multiple VPCs, Transit Gateway, NAT Gateway, and EC2 instances.
* **Shell Scripts**: Extracts network configuration and relationships into JSON reports for analysis.

---

## Repository Structure

```
├── network-mapping.yaml      # CloudFormation template
├── network-mapping.shell     # Main network mapping script
├── network-mapping.json      # Example input JSON for subnets and VPCs
├── gateways.shell            # Gateways mapping functions
├── instances.shell           # EC2 instance mapping functions
├── interfaces.shell          # ENI mapping functions
├── prefix_lists.shell        # Prefix list mapping functions
├── route-tables.shell        # Route table mapping functions
├── security-groups.shell     # Security group mapping functions
├── transit-gateways.shell    # Transit gateway mapping functions
└── vpcs-subnets.shell        # VPC and subnet mapping functions
```

---

## Deployment Instructions

Define an environment variable for:

```bash
export TARGET_PROFILE="default";
export TARGET_REGION="us-east-1";
export CFN_STACK_NAME="network-mapping";
export CHANGESET_NAME="prototype";
```

### Step 1: Validate the CloudFormation Template

```bash
$ aws --profile "${TARGET_PROFILE}" \
      --region "${TARGET_REGION}" \
      cloudformation validate-template \
      --template-body "file://${CFN_STACK_NAME}.yaml" ;
```

```json
{
    "Parameters": [
        {
            "ParameterKey": "VpcCidrPublic",
            "DefaultValue": "10.0.0.0/16",
            "NoEcho": false,
            "Description": "CIDR block for the public VPC"
        },
        {
            "ParameterKey": "KeyName",
            "DefaultValue": "devops",
            "NoEcho": false,
            "Description": "EC2 KeyPair for SSH access to the bastion host"
        },
        {
            "ParameterKey": "PublicSubnetCidr",
            "DefaultValue": "10.0.1.0/24",
            "NoEcho": false,
            "Description": "CIDR block for the public subnet"
        },
        {
            "ParameterKey": "PrivateSubnetCidr",
            "DefaultValue": "10.1.1.0/24",
            "NoEcho": false,
            "Description": "CIDR block for the private subnet"
        },
        {
            "ParameterKey": "RemoteAccessCidr",
            "DefaultValue": "<REMOTE-ADDRESSS>/32",
            "NoEcho": false,
            "Description": "Bastion-Host SSH Access CIDR block (IP Address)"
        },
        {
            "ParameterKey": "VpcCidrPrivate",
            "DefaultValue": "10.1.0.0/16",
            "NoEcho": false,
            "Description": "CIDR block for the private VPC"
        }
    ],
    "Description": "Merged Network Mapping Stack: Combines public and private VPCs, peering, transit gateway, routing, IGW, subnets, route tables, VPC endpoints, NAT Gateway, EC2 instances (SSM only), and Elastic IP.\n",
    "Capabilities": [
        "CAPABILITY_IAM"
    ],
    "CapabilitiesReason": "The following resource(s) require capabilities: [AWS::IAM::Role]"
}
```

---

### Step 2: Create the CloudFormation Change Set

```bash
$ aws --profile "${TARGET_PROFILE}" \
      --region "${TARGET_REGION}" \
      cloudformation create-change-set \
      --stack-name "${CFN_STACK_NAME}" \
      --template-body "file://${CFN_STACK_NAME}.yaml" \
      --change-set-name "${CHANGESET_NAME}" \
      --change-set-type CREATE \
      --capabilities CAPABILITY_NAMED_IAM ;
```

```json
{
    "Id": "arn:aws:cloudformation:us-east-1:<ACCOUNT_NUMBER>:changeSet/<CHANGESET-NAME>/2cd44ac7-2558-4e2d-aa49-2ac1d321e407",
    "StackId": "arn:aws:cloudformation:us-east-1:<ACCOUNT_NUMBER>:stack/<CFN-STACK-NAME>/e5943b70-588f-11f0-bded-123b5db61e2b"
}
```

---

### Step 3: Review the Change Set Status

```bash
$ aws --profile "${TARGET_PROFILE}" \
      --region "${TARGET_REGION}" \
      cloudformation describe-change-set \
      --stack-name "${CFN_STACK_NAME}" \
      --change-set-name "${CHANGESET_NAME}" \
      --query "Status" \
      --output text ;
```

Expected output: `CREATE_COMPLETE`

or

```bash
$ aws --profile "${TARGET_PROFILE}" \
      --region "${TARGET_REGION}" \
      cloudformation describe-change-set \
      --stack-name "${CFN_STACK_NAME}" \
      --change-set-name "${CHANGESET_NAME}" ;
```

```json
{
    "Changes": [
        {
            "Type": "Resource",
            "ResourceChange": {
                "Action": "Add",
                "LogicalResourceId": "AttachIGW",
                "ResourceType": "AWS::EC2::VPCGatewayAttachment",
                "Scope": [],
                "Details": []
            }
        },
        {
            "Type": "Resource",
            "ResourceChange": {
                "Action": "Add",
                "LogicalResourceId": "BastionEIPAssociation",
                "ResourceType": "AWS::EC2::EIPAssociation",
                "Scope": [],
                "Details": []
            }
        },
        {
            "Type": "Resource",
            "ResourceChange": {
                "Action": "Add",
                "LogicalResourceId": "BastionEIP",
                "ResourceType": "AWS::EC2::EIP",
                "Scope": [],
                "Details": []
            }
        },
        ...
        {
            "Type": "Resource",
            "ResourceChange": {
                "Action": "Add",
                "LogicalResourceId": "TGWAttachmentPublic",
                "ResourceType": "AWS::EC2::TransitGatewayAttachment",
                "Scope": [],
                "Details": []
            }
        },
        {
            "Type": "Resource",
            "ResourceChange": {
                "Action": "Add",
                "LogicalResourceId": "TransitGateway",
                "ResourceType": "AWS::EC2::TransitGateway",
                "Scope": [],
                "Details": []
            }
        },
        {
            "Type": "Resource",
            "ResourceChange": {
                "Action": "Add",
                "LogicalResourceId": "VPCPeeringConnection",
                "ResourceType": "AWS::EC2::VPCPeeringConnection",
                "Scope": [],
                "Details": []
            }
        }
    ],
    "ChangeSetName": "<CHANGESET-NAME>",
    "ChangeSetId": "arn:aws:cloudformation:us-east-1:<ACCOUNT_NUMBER>:changeSet/<CHANGESET-NAME>/2cd44ac7-2558-4e2d-aa49-2ac1d321e407",
    "StackId": "arn:aws:cloudformation:us-east-1:<ACCOUNT_NUMBER>:stack/<CFN-STACK-NAME>/e5943b70-588f-11f0-bded-123b5db61e2b",
    "StackName": "<CFN-STACK-NAME>",
    "Description": null,
    "Parameters": [
        {
            "ParameterKey": "VpcCidrPublic",
            "ParameterValue": "10.0.0.0/16"
        },
        {
            "ParameterKey": "KeyName",
            "ParameterValue": "devops"
        },
        {
            "ParameterKey": "PublicSubnetCidr",
            "ParameterValue": "10.0.1.0/24"
        },
        {
            "ParameterKey": "PrivateSubnetCidr",
            "ParameterValue": "10.1.1.0/24"
        },
        {
            "ParameterKey": "RemoteAccessCidr",
            "ParameterValue": "104.28.111.171/32"
        },
        {
            "ParameterKey": "VpcCidrPrivate",
            "ParameterValue": "10.1.0.0/16"
        }
    ],
    "CreationTime": "<TIME-STAMP>",
    "ExecutionStatus": "AVAILABLE",
    "Status": "CREATE_COMPLETE",
    "StatusReason": null,
    "NotificationARNs": [],
    "RollbackConfiguration": {},
    "Capabilities": [
        "CAPABILITY_NAMED_IAM"
    ],
    "Tags": null,
    "ParentChangeSetId": null,
    "IncludeNestedStacks": false,
    "RootChangeSetId": null,
    "OnStackFailure": null,
    "ImportExistingResources": null
}
```

or

```bash
$ aws --profile "${TARGET_PROFILE}" \
      --region "${TARGET_REGION}" \
      cloudformation describe-change-set \
      --stack-name "${CFN_STACK_NAME}" \
      --change-set-name "${CHANGESET_NAME}" \
      --query '{Status: Status, ExecutionStatus: ExecutionStatus, Changes: Changes[*].ResourceChange}' \
      --output json ;
```

```json
{
    "Status": "CREATE_COMPLETE",
    "ExecutionStatus": "AVAILABLE",
    "Changes": [
        {
            "Action": "Add",
            "LogicalResourceId": "AttachIGW",
            "ResourceType": "AWS::EC2::VPCGatewayAttachment",
            "Scope": [],
            "Details": []
        },
        {
            "Action": "Add",
            "LogicalResourceId": "BastionEIPAssociation",
            "ResourceType": "AWS::EC2::EIPAssociation",
            "Scope": [],
            "Details": []
        },
        {
            "Action": "Add",
            "LogicalResourceId": "BastionEIP",
            "ResourceType": "AWS::EC2::EIP",
            "Scope": [],
            "Details": []
        },
        ...
        {
            "Action": "Add",
            "LogicalResourceId": "TGWAttachmentPublic",
            "ResourceType": "AWS::EC2::TransitGatewayAttachment",
            "Scope": [],
            "Details": []
        },
        {
            "Action": "Add",
            "LogicalResourceId": "TransitGateway",
            "ResourceType": "AWS::EC2::TransitGateway",
            "Scope": [],
            "Details": []
        },
        {
            "Action": "Add",
            "LogicalResourceId": "VPCPeeringConnection",
            "ResourceType": "AWS::EC2::VPCPeeringConnection",
            "Scope": [],
            "Details": []
        }
    ]
}
```

---

### Step 4: Execute the Change Set

```bash
$ aws --profile "${TARGET_PROFILE}" \
      --region "${TARGET_REGION}" \
      cloudformation execute-change-set \
      --stack-name "${CFN_STACK_NAME}" \
      --change-set-name "${CHANGESET_NAME}" ;
```

Or alternatively deploy directly:

```bash
$ aws --profile "${TARGET_PROFILE}" \
      --region "${TARGET_REGION}" \
      cloudformation deploy \
      --stack-name "${CFN_STACK_NAME}" \
      --template-file "${CFN_STACK_NAME}.yaml" \
      --capabilities CAPABILITY_NAMED_IAM ;
```

```bash
Waiting for changeset to be created..
Waiting for stack create/update to complete
Successfully created/updated stack - <STACK-NAME>
```

You can watch the deployment status here:

```bash
$ watch -n 5  "aws --profile ${TARGET_PROFILE} \
                  --region ${TARGET_REGION} \
                  cloudformation describe-stacks \
                  --stack-name ${CFN_STACK_NAME} \
                  --output text
              " ;
```

```console
$ watch -n 5  "aws --profile ${TARGET_PROFILE} --region ${TARGET_REGION} cloudformation describe-stacks --stack-name ${CFN_STACK_NAME} --output text" ;
$ while true; do aws --profile ${TARGET_PROFILE} --region ${TARGET_REGION} cloudformation describe-stacks --stack-name ${CFN_STACK_NAME} --output text; sleep 15; clear; done;

STACKS  arn:aws:cloudformation:us-east-1:***:changeSet/prototype/a43bbf56-cced-4ba4-bd07-9428f114ae6f  2025-07-09T04:10:38.810000+00:00
Merged Network Mapping Stack: Combines public and private VPCs, peering, transit gateway, routing, IGW, subnets, route tables, VPC endpoints, NAT Gateway, EC2 instances (SSM only), and Elastic IP.

False   False   2025-07-09T04:11:37.725000+00:00
arn:aws:cloudformation:us-east-1:***:stack/network-mapping/abc89460-5c7a-11f0-9326-0affc332652d
network-mapping CREATE_COMPLETE

CAPABILITIES    CAPABILITY_NAMED_IAM
DRIFTINFORMATION        NOT_CHECKED

OUTPUTS ARN of the Public Route Table           PublicRouteTable        rtb-0a105b3415eec5b66
OUTPUTS ID of the Interface Endpoint for Secrets Manager                VPCEndpointSecretsManager       vpce-0612cbc252b5682ed
OUTPUTS ID of the Interface Endpoint for SSM            VPCEndpointSSM  vpce-0a0579fa69e01569c
OUTPUTS Elastic IP associated with NAT Gateway          NatPublicIP     13.223.16.32
OUTPUTS DevOps EC2 Instance ID          EC2DevOpsInstanceId     i-0cfc29aca7e1130c3
OUTPUTS Interface Endpoint for SSM (DevOps VPC)         DevOpsEndpointSSMId     vpce-02956469f1b13181d
OUTPUTS Interface Endpoint for Secrets Manager (Private VPC)            VPCEndpointSecretsManagerId     vpce-0612cbc252b5682ed
OUTPUTS Elastic IP associated with Bastion Host         BastionPublicIP 3.213.170.119
OUTPUTS ID of the Interface Endpoint for Monitoring             DevOpsEndpointMonitoring        vpce-0183586ade28a3487
OUTPUTS Private EC2 Instance ID         EC2PrivateInstanceId    i-07dd74937c817548b
OUTPUTS ID of the Gateway VPC Endpoint for S3           VPCEndpointS3   vpce-0d3fa9c6ea83d144d
OUTPUTS Private Route Table ID          PrivateRouteTableId     rtb-0a01a0b09d8ff4547
OUTPUTS Transit Gateway Attachment (Public VPC)         TGWAttachmentPublicId   tgw-attach-0189b5caa6e869108
OUTPUTS Internet Gateway ID (No ARN in CFN)             InternetGateway igw-00c373c8feb8a9fc9
OUTPUTS Gateway VPC Endpoint for S3 (Private VPC)               VPCEndpointS3Id vpce-0d3fa9c6ea83d144d
OUTPUTS Public VPC ID   DevOps-PublicVPCId      PublicVPCId     vpc-028b2dc2728406fd9
OUTPUTS DevOps Route Table ID           DevOpsRouteTableId      rtb-094fea2f99bb0acc9
OUTPUTS Private VPC ID  DevOps-PrivateVPCId     PrivateVPCId    vpc-0dd98b0cb5d3ded56
OUTPUTS ID of the Interface Endpoint for EC2 Messages           DevOpsEndpointEC2Messages       vpce-07e3c737e14d68440
OUTPUTS ID of the Interface Endpoint for KMS            DevOpsEndpointKMS       vpce-086fb7be5b7d2272d
OUTPUTS ARN of the DevOps Subnet                DevOpsSubnet    subnet-0a7863c4791ae49f2
OUTPUTS Interface Endpoint for EC2 Messages (DevOps VPC)                DevOpsEndpointEC2MessagesId     vpce-07e3c737e14d68440
OUTPUTS Interface Endpoint for KMS (DevOps VPC)         DevOpsEndpointKMSId     vpce-086fb7be5b7d2272d
OUTPUTS Bastion Elastic Network Interface ID            BastionENIId    eni-03fd376cea628ecb2
OUTPUTS Interface Endpoint for Backup (DevOps VPC)              DevOpsEndpointBackupId  vpce-04586057e6b9b81db
OUTPUTS NAT Gateway ID          NatGatewayId    nat-049d8400adc86f196
OUTPUTS Interface Endpoint for CloudWatch Logs (DevOps VPC)             DevOpsEndpointCloudWatchLogsId  vpce-0e0e6e23dc2b93126
OUTPUTS VPC Peering Connection ID               VPCPeeringId    pcx-0b2f7050573bcea16
OUTPUTS ARN of the Public Subnet                PublicSubnetArn subnet-03ea1805ea65cba01
OUTPUTS Transit Gateway Attachment (DevOps VPC)         TGWAttachmentDevOpsId   tgw-attach-0a3e2ed35a7e0b36b
OUTPUTS NAT Gateway ID (No ARN in CFN)          NatGateway      nat-049d8400adc86f196
OUTPUTS Private Subnet ID               PrivateSubnetId subnet-0ebc71f9acbeba83c
OUTPUTS Public Route Table ID           PublicRouteTableId      rtb-0a105b3415eec5b66
OUTPUTS Interface Endpoint for EC2 Messages (Private VPC)               VPCEndpointEC2MessagesId        vpce-0bdcdce55c91654ce
OUTPUTS ID of the Interface Endpoint for EC2 Messages           VPCEndpointEC2Messages  vpce-0bdcdce55c91654ce
OUTPUTS Name of the IAM Role used for SSM access                SSMRoleName     network-mapping-SSMInstanceRole-PtzQ3Mv5em5R
OUTPUTS DevOps Subnet ID                DevOpsSubnetId  subnet-0a7863c4791ae49f2
OUTPUTS ARN of the IAM Role used for SSM access         SSMRoleArn      arn:aws:iam::***:role/network-mapping-SSMInstanceRole-PtzQ3Mv5em5R
OUTPUTS ID of the Interface Endpoint for Events         DevOpsEndpointEvents    vpce-08dbbd58f7573d512
OUTPUTS Interface Endpoint for SNS (DevOps VPC)         DevOpsEndpointSNSId     vpce-03cc85fabe0665b8a
OUTPUTS ID of the Interface Endpoint for SSM Messages           DevOpsEndpointSSMMessages       vpce-0b3ff0d121c4eb4ae
OUTPUTS ID of the Interface Endpoint for Secrets Manager                DevOpsEndpointSecretsManager    vpce-0e4c07aa29817be75
OUTPUTS ARN of the DevOps Route Table           DevOpsRouteTable        rtb-094fea2f99bb0acc9
OUTPUTS ARN of the Private Subnet               PrivateSubnet   subnet-0ebc71f9acbeba83c
OUTPUTS Instance Profile for SSM-managed EC2s           SSMInstanceProfile      network-mapping-SSMInstanceProfile-mcP8cbDL9bZW
OUTPUTS ID of the Interface Endpoint for CloudWatch Logs                DevOpsEndpointCloudWatchLogs    vpce-0e0e6e23dc2b93126
OUTPUTS Interface Endpoint for Events (DevOps VPC)              DevOpsEndpointEventsId  vpce-08dbbd58f7573d512
OUTPUTS Elastic Network Interface ID            BastionENI      eni-03fd376cea628ecb2
OUTPUTS ID of the Interface Endpoint for SSM            DevOpsEndpointSSM       vpce-02956469f1b13181d
OUTPUTS Internet Gateway ID             InternetGatewayId       igw-00c373c8feb8a9fc9
OUTPUTS Interface Endpoint for SSM (Private VPC)                VPCEndpointSSMId        vpce-0a0579fa69e01569c
OUTPUTS ARN of the Private Route Table          PrivateRouteTable       rtb-0a01a0b09d8ff4547
OUTPUTS ID of the Interface Endpoint for SNS            DevOpsEndpointSNS       vpce-03cc85fabe0665b8a
OUTPUTS Transit Gateway ID      DevOps-TransitGatewayId TransitGatewayId        tgw-001bc8550c79f14b9
OUTPUTS Interface Endpoint for Secrets Manager (DevOps VPC)             DevOpsEndpointSecretsManagerId  vpce-0e4c07aa29817be75
OUTPUTS Public Subnet ID                PublicSubnetId  subnet-03ea1805ea65cba01
OUTPUTS Bastion Host Elastic IP         BastionEIP      3.213.170.119
OUTPUTS Interface Endpoint for SSM Messages (DevOps VPC)                DevOpsEndpointSSMMessagesId     vpce-0b3ff0d121c4eb4ae
OUTPUTS Public EC2 Instance ID          EC2PublicInstanceId     i-0e64cbd9718ff9743
OUTPUTS DevOps VPC ID   DevOps-DevOpsVPCId      DevOpsVPCId     vpc-0f7fa8fdff02da590
OUTPUTS Transit Gateway Attachment (Private VPC)                TGWAttachmentPrivateId  tgw-attach-09fef679a102279f2
OUTPUTS Interface Endpoint for Monitoring (DevOps VPC)          DevOpsEndpointMonitoringId      vpce-0183586ade28a3487
OUTPUTS ID of the Interface Endpoint for Backup         DevOpsEndpointBackup    vpce-04586057e6b9b81db

PARAMETERS      VpcCidrPublic   10.0.0.0/16
PARAMETERS      KeyName devops
PARAMETERS      PublicSubnetCidr        10.0.1.0/24
PARAMETERS      PrivateSubnetCidr       10.1.1.0/24
PARAMETERS      RemoteAccessCidr        104.28.111.171/32
PARAMETERS      VpcCidrPrivate  10.1.0.0/16
```

---

### Step 5: Get CloudFormation Stack Outputs

```bash
$ aws --profile "${TARGET_PROFILE}" \
      --region "${TARGET_REGION}" \
      cloudformation describe-stacks \
      --stack-name "${CFN_STACK_NAME}" \
      --query 'Stacks[0].Outputs' \
      --output json ;
```

```json
{
    "Stacks": [
        {
            "StackId": "arn:aws:cloudformation:us-east-1:<ACCOUNT_NUMBER>:stack/<CFN-STACK-NAME>/e5943b70-588f-11f0-bded-123b5db61e2b",
            "StackName": "<CFN-STACK-NAME>",
            "ChangeSetId": "arn:aws:cloudformation:us-east-1:<ACCOUNT_NUMBER>:changeSet/<CHANGESET-NAME>/2cd44ac7-2558-4e2d-aa49-2ac1d321e407",
            "Description": "Merged Network Mapping Stack: Combines public and private VPCs, peering, transit gateway, routing, IGW, subnets, route tables, VPC endpoints, NAT Gateway, EC2 instances (SSM only), and Elastic IP.\n",
            "Parameters": [
                {
                    "ParameterKey": "VpcCidrPublic",
                    "ParameterValue": "10.0.0.0/16"
                },
                {
                    "ParameterKey": "KeyName",
                    "ParameterValue": "devops"
                },
                {
                    "ParameterKey": "PublicSubnetCidr",
                    "ParameterValue": "10.0.1.0/24"
                },
                {
                    "ParameterKey": "PrivateSubnetCidr",
                    "ParameterValue": "10.1.1.0/24"
                },
                {
                    "ParameterKey": "RemoteAccessCidr",
                    "ParameterValue": "104.28.111.171/32"
                },
                {
                    "ParameterKey": "VpcCidrPrivate",
                    "ParameterValue": "10.1.0.0/16"
                }
            ],
            "CreationTime": "<TIME-STAMP>",
            "LastUpdatedTime": "<TIME-STAMP>",
            "RollbackConfiguration": {},
            "StackStatus": "CREATE_COMPLETE",
            "DisableRollback": false,
            "NotificationARNs": [],
            "Capabilities": [
                "CAPABILITY_NAMED_IAM"
            ],
            "Outputs": [
                {
                    "OutputKey": "BastionEIPOutput",
                    "OutputValue": "44.208.125.8",
                    "Description": "Public Elastic IP (Bastion Host)"
                },
                {
                    "OutputKey": "PublicVPCId",
                    "OutputValue": "vpc-0ef6aabbd8e8bebf3",
                    "Description": "Public VPC ID"
                },
                {
                    "OutputKey": "TransitGatewayId",
                    "OutputValue": "tgw-0e89190a2c4909473",
                    "Description": "Transit Gateway ID"
                },
                {
                    "OutputKey": "VPCPeeringId",
                    "OutputValue": "pcx-092f8f71e3574333b",
                    "Description": "VPC Peering Connection ID"
                },
                {
                    "OutputKey": "PrivateVPCId",
                    "OutputValue": "vpc-0a0e18a8bc9e6d7d2",
                    "Description": "Private VPC ID"
                },
                {
                    "OutputKey": "DevOpsVPCId",
                    "OutputValue": "vpc-093f91b27e1fc494e",
                    "Description": "DevOps VPC ID"
                }
            ],
            "Tags": [],
            "EnableTerminationProtection": false,
            "DriftInformation": {
                "StackDriftStatus": "NOT_CHECKED"
            }
        }
    ]
}
```

---

### (Optional) Step 6: Delete the Change Set

```bash
$ aws --profile "${TARGET_PROFILE}" \
      --region "${TARGET_REGION}" \
      cloudformation delete-change-set \
      --stack-name "${CFN_STACK_NAME}" \
      --change-set-name "$(
        aws --profile "${TARGET_PROFILE}" \
            --region "${TARGET_REGION}" \
            cloudformation list-change-sets \
            --stack-name "${CFN_STACK_NAME}" \
            --query 'Summaries[0].ChangeSetName' \
            --output text
        )";
```

---

## Run the Network Mapping Script

```bash
bash ${CFN_STACK_NAME}.shell \
     "${TARGET_REGION}" \
     "${TARGET_PROFILE}" ;

Mapping subnet: "DevOps-LockedSubnet" (expected CIDR: "10.2.1.0/24")
Completed mapping for subnet: "DevOps-LockedSubnet" (subnet-0ee47faa6a08099bf)

Mapping subnet: "DevOps-PrivateSubnet" (expected CIDR: "10.1.1.0/24")
Completed mapping for subnet: "DevOps-PrivateSubnet" (subnet-0e123c2b35cc8f96a)

Mapping subnet: "DevOps-PublicSubnet" (expected CIDR: "10.0.1.0/24")
Completed mapping for subnet: "DevOps-PublicSubnet" (subnet-0dd3d6c072f085628)

All reports saved under: reports/20250703_214315
```

---

```console
------------------------------------------------------------------------
|                      DescribeInstanceInformation                     |
+----------------------------------------------------------------------+
||                       InstanceInformationList                      ||
|+-------------------------------+------------------------------------+|
||  AgentVersion                 |  3.3.2746.0                        ||
||  AssociationStatus            |  Pending                           ||
||  ComputerName                 |  ip-10-0-1-202.ec2.internal        ||
||  IPAddress                    |  10.0.1.202                        ||
||  InstanceId                   |  i-04f600ad6fc6aa11c               ||
||  IsLatestVersion              |  True                              ||
||  LastAssociationExecutionDate |  2025-07-03T21:40:25.133000-07:00  ||
||  LastPingDateTime             |  2025-07-03T21:42:30.794000-07:00  ||
||  PingStatus                   |  Online                            ||
||  PlatformName                 |  Amazon Linux                      ||
||  PlatformType                 |  Linux                             ||
||  PlatformVersion              |  2                                 ||
||  ResourceType                 |  EC2Instance                       ||
||  SourceId                     |  i-04f600ad6fc6aa11c               ||
||  SourceType                   |  AWS::EC2::Instance                ||
|+-------------------------------+------------------------------------+|
|||                        AssociationOverview                       |||
||+------------------------------------+-----------------------------+||
|||  DetailedStatus                    |  InProgress                 |||
||+------------------------------------+-----------------------------+||
||||            InstanceAssociationStatusAggregatedCount            ||||
|||+-------------------------------------------+--------------------+|||
||||  Pending                                  |  2                 ||||
||||  Success                                  |  3                 ||||
|||+-------------------------------------------+--------------------+|||
||                       InstanceInformationList                      ||
|+-------------------------------+------------------------------------+|
||  AgentVersion                 |  3.0.1124.0                        ||
||  AssociationStatus            |  Pending                           ||
||  ComputerName                 |  ip-10-1-1-58.ec2.internal         ||
||  IPAddress                    |  10.1.1.58                         ||
||  InstanceId                   |  i-0fb50127bdfa8b452               ||
||  IsLatestVersion              |  False                             ||
||  LastAssociationExecutionDate |  2025-07-03T21:43:55.470000-07:00  ||
||  LastPingDateTime             |  2025-07-03T21:43:36.786000-07:00  ||
||  PingStatus                   |  Online                            ||
||  PlatformName                 |  Amazon Linux                      ||
||  PlatformType                 |  Linux                             ||
||  PlatformVersion              |  2                                 ||
||  ResourceType                 |  EC2Instance                       ||
||  SourceId                     |  i-0fb50127bdfa8b452               ||
||  SourceType                   |  AWS::EC2::Instance                ||
|+-------------------------------+------------------------------------+|
|||                        AssociationOverview                       |||
||+------------------------------------+-----------------------------+||
|||  DetailedStatus                    |  InProgress                 |||
||+------------------------------------+-----------------------------+||
||||            InstanceAssociationStatusAggregatedCount            ||||
|||+-------------------------------------------+--------------------+|||
||||  Pending                                  |  4                 ||||
||||  Success                                  |  1                 ||||
|||+-------------------------------------------+--------------------+|||
||                       InstanceInformationList                      ||
|+-------------------------------+------------------------------------+|
||  AgentVersion                 |  3.3.2746.0                        ||
||  AssociationStatus            |  Pending                           ||
||  ComputerName                 |  ip-10-2-1-244.ec2.internal        ||
||  IPAddress                    |  10.2.1.244                        ||
||  InstanceId                   |  i-0cdbf44c39e6bc9ee               ||
||  IsLatestVersion              |  True                              ||
||  LastAssociationExecutionDate |  2025-07-03T21:40:49.928000-07:00  ||
||  LastPingDateTime             |  2025-07-03T21:41:34.833000-07:00  ||
||  PingStatus                   |  Online                            ||
||  PlatformName                 |  Amazon Linux                      ||
||  PlatformType                 |  Linux                             ||
||  PlatformVersion              |  2                                 ||
||  ResourceType                 |  EC2Instance                       ||
||  SourceId                     |  i-0cdbf44c39e6bc9ee               ||
||  SourceType                   |  AWS::EC2::Instance                ||
|+-------------------------------+------------------------------------+|
|||                        AssociationOverview                       |||
||+------------------------------------+-----------------------------+||
|||  DetailedStatus                    |  InProgress                 |||
||+------------------------------------+-----------------------------+||
||||            InstanceAssociationStatusAggregatedCount            ||||
|||+-------------------------------------------+--------------------+|||
||||  Pending                                  |  1                 ||||
||||  Success                                  |  4                 ||||
|||+-------------------------------------------+--------------------+|||
 [SSM] Command ID: 09464e55-240d-4606-a1c4-083b71175f31
```

---

```bash
$ cat ./reports/20250706_231530/network-mapping.csv ;

"vpc","subnet","cidr"
"DevOps-PublicVPC","DevOps-PublicSubnet","10.0.1.0/24"
"DevOps-PrivateVPC","DevOps-PrivateSubnet","10.1.1.0/24"
"DevOps-LockedVPC","DevOps-LockedSubnet","10.2.1.0/24"
```

### Output Structure

```
├── capture
│   └── default
│       └── personal
│           └── us-east-1
│               └── 20250708
│                   └── 212018
│                       ├── network-mapping.csv
│                       ├── network-mapping.json
│                       ├── shared
│                       │   ├── advertised
│                       │   │   └── tgw-rtb-0d450d799e35a6419.json
│                       │   ├── associated
│                       │   │   └── tgw-rtb-0d450d799e35a6419.json
│                       │   ├── caller_identity.json
│                       │   ├── elastic_ips.json
│                       │   ├── managed_instances.json
│                       │   ├── prefix_lists.json
│                       │   ├── propagated
│                       │   │   └── tgw-rtb-0d450d799e35a6419.json
│                       │   ├── tgw_route_tables.json
│                       │   ├── transit_gateways.json
│                       │   └── vpc_peering.json
│                       └── vpcs
│                           ├── devops-locked-vpc
│                           │   ├── dhcp_options.json
│                           │   ├── interfaces
│                           │   │   ├── ec2_instances.json
│                           │   │   ├── transit_gateway_attachments.json
│                           │   │   └── vpc_endpoints.json
│                           │   ├── network_acls.json
│                           │   ├── network_interfaces.json
│                           │   ├── security_groups_unattached.json
│                           │   ├── security_groups.json
│                           │   ├── subnets
│                           │   │   └── devops-locked-subnet
│                           │   │       ├── default_route_tables.json
│                           │   │       ├── ec2_instances.json
│                           │   │       ├── ssm_output_i-0cfc29aca7e1130c3.txt
│                           │   │       ├── subnet_route_tables.json
│                           │   │       └── subnet.json
│                           │   ├── vpc_ec2_instances.json
│                           │   ├── vpc_endpoints.json
│                           │   ├── vpc_route_tables_unassociated.json
│                           │   ├── vpc_route_tables.json
│                           │   ├── vpc_tgw_attachments.json
│                           │   └── vpc.json
│                           ├── devops-private-vpc
│                           │   ├── dhcp_options.json
│                           │   ├── interfaces
│                           │   │   ├── ec2_instances.json
│                           │   │   ├── transit_gateway_attachments.json
│                           │   │   └── vpc_endpoints.json
│                           │   ├── network_acls.json
│                           │   ├── network_interfaces.json
│                           │   ├── security_groups_unattached.json
│                           │   ├── security_groups.json
│                           │   ├── subnets
│                           │   │   └── devops-private-subnet
│                           │   │       ├── default_route_tables.json
│                           │   │       ├── ec2_instances.json
│                           │   │       ├── ssm_output_i-07dd74937c817548b.txt
│                           │   │       ├── subnet_route_tables.json
│                           │   │       └── subnet.json
│                           │   ├── vpc_ec2_instances.json
│                           │   ├── vpc_endpoints.json
│                           │   ├── vpc_route_tables_unassociated.json
│                           │   ├── vpc_route_tables.json
│                           │   ├── vpc_tgw_attachments.json
│                           │   └── vpc.json
│                           └── devops-public-vpc
│                               ├── dhcp_options.json
│                               ├── igw.json
│                               ├── interfaces
│                               │   ├── ec2_instances.json
│                               │   ├── nat_gateways.json
│                               │   └── transit_gateway_attachments.json
│                               ├── network_acls.json
│                               ├── network_interfaces.json
│                               ├── security_groups_unattached.json
│                               ├── security_groups.json
│                               ├── subnets
│                               │   └── devops-public-subnet
│                               │       ├── default_route_tables.json
│                               │       ├── ec2_instances.json
│                               │       ├── ssm_output_i-0e64cbd9718ff9743.txt
│                               │       ├── subnet_route_tables.json
│                               │       └── subnet.json
│                               ├── vpc_ec2_instances.json
│                               ├── vpc_nat_gateways.json
│                               ├── vpc_route_tables_unassociated.json
│                               ├── vpc_route_tables.json
│                               ├── vpc_tgw_attachments.json
│                               └── vpc.json
```

```console
├── exports
│   └── capture
│       └── default
│           └── personal
│               └── us-east-1
│                   └── 20250708
│                       └── 212018
│                           ├── caller_identity.csv
│                           ├── elastic_ips.csv
│                           ├── managed_instances.csv
│                           ├── prefix_lists.csv
│                           ├── tgw_advertised_routes.csv
│                           ├── tgw_route_tables.csv
│                           ├── transit_gateways.csv
│                           ├── vpc_peering.csv
│                           ├── vpcs_dhcp_options.csv
│                           ├── vpcs_ec2_instances.csv
│                           ├── vpcs_endpoints.csv
│                           ├── vpcs_igws.csv
│                           ├── vpcs_metadata.csv
│                           ├── vpcs_nat_gateways.csv
│                           ├── vpcs_network_acls.csv
│                           ├── vpcs_network_interfaces.csv
│                           ├── vpcs_route_tables_unassociated.csv
│                           ├── vpcs_route_tables.csv
│                           ├── vpcs_security_groups_unattached.csv
│                           ├── vpcs_security_groups.csv
│                           └── vpcs_tgw_attachments.csv
```
