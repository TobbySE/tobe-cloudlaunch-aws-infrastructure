CloudLaunch - AWS Cloud Infrastructure Project
Project Overview
CloudLaunch is a comprehensive AWS infrastructure demonstration showcasing secure static website hosting and enterprise-grade network architecture. This project implements AWS best practices for S3, IAM, and VPC services to create a scalable, secure cloud environment.
Project Architecture
┌─────────────────────────────────────────────────────────────────┐
│                          Internet                                │
└─────────────────────────┬───────────────────────────────────────┘
                          │
    ┌─────────────────────▼─────────────────────┐
    │              CloudFront CDN               │
    │        (Global Edge Locations)           │
    └─────────────────────┬─────────────────────┘
                          │
    ┌─────────────────────▼─────────────────────┐
    │           S3 Static Website               │
    │      (cloudlaunch-site-bucket)           │
    └───────────────────────────────────────────┘

    ┌─────────────────────────────────────────────┐
    │              CloudLaunch VPC               │
    │              (10.0.0.0/16)                │
    │                                            │
    │  ┌──────────────────────────────────────┐  │
    │  │        Public Subnet                 │  │
    │  │       (10.0.1.0/24)                │  │
    │  │    [Load Balancers/Bastion]        │  │
    │  └──────────────┬───────────────────────┘  │
    │                 │                          │
    │  ┌──────────────▼───────────────────────┐  │
    │  │      Application Subnet              │  │
    │  │       (10.0.2.0/24)                │  │
    │  │     [App Servers - Private]        │  │
    │  └──────────────┬───────────────────────┘  │
    │                 │                          │
    │  ┌──────────────▼───────────────────────┐  │
    │  │       Database Subnet                │  │
    │  │       (10.0.3.0/28)                │  │
    │  │      [RDS - Isolated]              │  │
    │  └────────────────────────────────────────┘  │
    └─────────────────────────────────────────────┘
Task 1: Secure S3 Static Website Hosting
Implementation Summary
Created a comprehensive S3-based static website hosting solution with granular access controls and global content delivery.
Components Implemented
S3 Buckets Configuration

tobe-cloudlaunch-site-bucket

Hosts static website content (HTML, CSS, JS)
Public read access for website visitors
Static website hosting enabled
Integrated with CloudFront for global distribution


tobe-cloudlaunch-private-bucket

Private storage for internal documents
IAM user has GetObject/PutObject permissions only
No public access or delete permissions


tobe-cloudlaunch-visible-only-bucket

List-only access for IAM user
Content remains inaccessible
Demonstrates granular permission control



Security Features

Principle of Least Privilege: IAM user has minimal required permissions
No Delete Access: Prevents accidental data loss
Bucket Isolation: Each bucket serves distinct purpose with appropriate access levels

Live Deployment URLs
Primary Website Access:

S3 Website Endpoint: http://tobe-cloudlaunch-site-bucket.s3-website-eu-west-1.amazonaws.com


IAM Policy Configuration
CloudLaunch User Permissions Policy
json{
    "Version": "2012-10-17",
    "Statement": [
        {
            "Sid": "ListAllBuckets",
            "Effect": "Allow",
            "Action": "s3:ListBucket",
            "Resource": [
                "arn:aws:s3:::tobe-cloudlaunch-site-bucket",
                "arn:aws:s3:::tobe-cloudlaunch-private-bucket",
                "arn:aws:s3:::tobe-cloudlaunch-visible-only-bucket"
            ]
        },
        {
            "Sid": "GetObjectSiteBucket",
            "Effect": "Allow",
            "Action": "s3:GetObject",
            "Resource": "arn:aws:s3:::tobe-cloudlaunch-site-bucket/*"
        },
        {
            "Sid": "GetPutObjectPrivateBucket",
            "Effect": "Allow",
            "Action": [
                "s3:GetObject",
                "s3:PutObject"
            ],
            "Resource": "arn:aws:s3:::tobe-cloudlaunch-private-bucket/*"
        },
        {
            "Sid": "DenyDeleteEverywhere",
            "Effect": "Deny",
            "Action": [
                "s3:DeleteObject",
                "s3:DeleteObjectVersion"
            ],
            "Resource": "*"
        },
        {
            "Sid": "DenyVisibleOnlyBucketContent",
            "Effect": "Deny",
            "Action": [
                "s3:GetObject",
                "s3:PutObject"
            ],
            "Resource": "arn:aws:s3:::tobe-cloudlaunch-visible-only-bucket/*"
        }
    ]
}
VPC Read-Only Access Policy
json{
    "Version": "2012-10-17",
    "Statement": [
        {
            "Sid": "VPCReadOnlyAccess",
            "Effect": "Allow",
            "Action": [
                "ec2:DescribeVpcs",
                "ec2:DescribeSubnets",
                "ec2:DescribeRouteTables",
                "ec2:DescribeInternetGateways",
                "ec2:DescribeSecurityGroups",
                "ec2:DescribeNetworkAcls",
                "ec2:DescribeVpcEndpoints",
                "ec2:DescribeNatGateways",
                "ec2:DescribeVpcPeeringConnections",
                "ec2:DescribeAvailabilityZones",
                "ec2:DescribeRegions"
            ],
            "Resource": "*"
        },
        {
            "Sid": "VPCSpecificAccess",
            "Effect": "Allow",
            "Action": [
                "ec2:DescribeVpcs",
                "ec2:DescribeSubnets",
                "ec2:DescribeRouteTables",
                "ec2:DescribeSecurityGroups"
            ],
            "Resource": "*",
            "Condition": {
                "StringEquals": {
                    "ec2:Region": "eu-west-1"
                }
            }
        }
    ]
}
 Task 2: Enterprise VPC Network Architecture
Implementation Summary
Designed and implemented a secure, scalable 3-tier VPC architecture following AWS best practices for network isolation and security.
Network Design Specifications
VPC Configuration

VPC Name: tobe-cloudlaunch-vpc
CIDR Block: 10.0.0.0/16 (65,536 IP addresses)
Internet Gateway: cloudlaunch-igw

Subnet Architecture
SubnetCIDR BlockPurposeInternet AccessRoute TablePublic10.0.1.0/24Load Balancers, Bastion Hosts Yescloudlaunch-public-rtApplication10.0.2.0/24Private App Servers Nocloudlaunch-app-rtDatabase10.0.3.0/28RDS, Database Services Nocloudlaunch-db-rt
Security Groups Configuration

tobe-cloudlaunch-app-sg

Inbound: HTTP (Port 80) from VPC only (10.0.0.0/16)
Purpose: Application server access control


tobe-cloudlaunch-db-sg

Inbound: MySQL (Port 3306) from App Subnet only (10.0.2.0/24)
Purpose: Database isolation and security



Route Tables Configuration

tobe-cloudlaunch-public-rt

Routes: 0.0.0.0/0 → Internet Gateway
Associated with: Public Subnet


tobe-cloudlaunch-app-rt

Routes: Local VPC traffic only
No internet access (private)


tobe-cloudlaunch-db-rt

Routes: Local VPC traffic only
Fully isolated from internet



Security Implementation
Network Security Matrix
Source → Destination | Port |Status | Justification
---------------------|------|-------|---------------
Internet → Public    | Any  | Allow | Public services
Internet → App       | Any  | Deny  | Private tier
Internet → DB        | Any  | Deny  | Database isolation
Public → App         | 80   | Allow | Load balancer to app
Public → DB          | Any  | Deny  | No direct DB access
App → DB             | 3306 | Allow | App to database
App → Internet       | Any  | Deny  | No outbound (NAT can be added)
DB → Internet        | Any  | Deny  | Complete isolation
Access Control Features

Zero Trust Network: No default internet access for private resources
Principle of Least Privilege: Minimal required network permissions
Defense in Depth: Multiple security layers (Route Tables + Security Groups)
Microsegmentation: Each tier isolated with specific access rules

Security Best Practices Implemented
1. Identity & Access Management

Custom IAM policies with minimal required permissions
No administrative access for service accounts
Explicit deny rules for destructive operations
Resource-specific access controls

2. Network Security

Private subnets with no internet gateway routes
Security groups with specific port/protocol restrictions
Network isolation between application tiers
Controlled communication pathways

3. Data Protection

Private bucket configurations
HTTPS enforcement via CloudFront
Bucket versioning enabled
Access logging capabilities

4. Operational Security

Read-only operational access for monitoring
Separation of duties between infrastructure and operations
Audit trail through AWS CloudTrail (recommended)


Repository Structure
cloudlaunch-aws-infrastructure/
├── README.md                          # This file
├── policies/
│   ├── cloudlaunch-user-s3-policy.json
│   └── cloudlaunch-user-vpc-policy.json
└── websites/
    ├── index.html                     # Website homepage
    └── style.css                      # Website styling

Testing & Validation
Functional Testing
bash# Test IAM user S3 permissions
aws s3 ls --profile cloudlaunch-user
aws s3 cp test-file.txt s3://cloudlaunch-private-bucket/ --profile cloudlaunch-user

# Test VPC read access
aws ec2 describe-vpcs --profile cloudlaunch-user
aws ec2 describe-security-groups --profile cloudlaunch-user

# Test denied operations (should fail)
aws s3 rm s3://cloudlaunch-private-bucket/test-file.txt --profile cloudlaunch-user
Security Validation

 Website accessible via S3
 Private buckets inaccessible from internet
 IAM user cannot delete objects
 VPC components viewable but not modifiable
 Database subnet completely isolated


Project Status: Complete | AWS Services: S3, CloudFront, IAM, VPC, EC2 Security Groups | Security Level: Enterprise-Ready
