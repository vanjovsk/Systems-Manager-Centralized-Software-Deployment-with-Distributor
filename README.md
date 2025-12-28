


AWS Systems Manager: Centralized Software Deployment with Distributor
📖 Project Overview
Manually installing agents and software across a distributed server fleet is inefficient and increases the risk of configuration drift. This project demonstrates how to use AWS Systems Manager Distributor to package a custom agent and Run Command to deploy it at scale across multiple EC2 instances.

The Scenario: To implement centralized monitoring, I needed to deploy the CloudWatch Agent to a target fleet. Instead of manual installation, I created a versioned, reusable package that can be updated and redeployed to thousands of instances with a single command.

🏗️ Architecture Flow
This diagram illustrates the flow from artifact creation to fleet-wide execution:

Snippet de código

graph TD
    A[Admin: CloudWatch Agent RPM] --> B[S3 Bucket: Central Storage]
    B --> C[SSM Distributor: Custom Package Created]
    C --> D{Run Command: AWS-ConfigureAWSPackage}
    D -->|Target: Name=Test-Instance| E[EC2 Instance 1]
    D -->|Target: Name=Test-Instance| F[EC2 Instance 2]
    E --> G[CloudWatch Agent Installed]
    F --> G
🛠️ Technical Stack
AWS Systems Manager (Distributor): For creating, versioning, and managing software packages.

AWS Systems Manager (Run Command): For executing remote installation scripts without requiring SSH access.

Amazon S3: Used as the backend repository for software binaries and JSON manifests.

Amazon EC2: Managed nodes with the SSM Agent installed.

🚀 Key Implementation Steps
1. Building the Custom Package
I leveraged the Distributor "Simple" creation mode to build the cloudwatchagent package:

Storage: Linked the package to a specific S3 bucket for reliable artifact retrieval.

Manifesting: Configured the mapping for Amazon Linux and x86_64 architecture, ensuring the correct RPM was delivered to the correct OS.

Automated Installers: Utilized the built-in installation scripts provided by Distributor to automate the yum install process.

2. Fleet-Wide Deployment
To install the package, I utilized Run Command with the AWS-ConfigureAWSPackage document:

Installation Logic: Set the action to Install and referenced the custom cloudwatchagent package name.

Target Selection: Identified targets manually for this lab, but documented the industry best practice of using Resource Tags for dynamic scaling.

Verification: Monitored execution logs until the "Success" status confirmed the agent was active and running on the target nodes.

🔧 Troubleshooting & Key Insights
S3 Access Issues: If an installation fails, the most common culprit is the IAM Instance Profile. The EC2 instance must have a policy (like AmazonSSMManagedInstanceCore) that allows it to read from the S3 bucket where the Distributor package resides.

Platform Mismatches: Distributor packages are platform-specific. If the Target Platform in the manifest doesn't match the instance OS, the installation will fail immediately.

Agent Connectivity: Run Command requires the SSM Agent to be "Online." If an instance is missing from the targets list, verify its outbound connectivity to the SSM service endpoints.

Stateful Management: While Run Command is great for one-time installs, State Manager is the preferred tool for ensuring a package stays installed over the long term (compliance).

<img width="1640" height="766" alt="Screenshot 2025-12-27 at 11 01 11 PM" src="https://github.com/user-attachments/assets/480a84f0-813c-4ba6-b929-aa17b0658a26" />

<img width="1641" height="857" alt="Screenshot 2025-12-27 at 11 06 13 PM" src="https://github.com/user-attachments/assets/6a9cd197-57ea-4f67-b14b-0879555d5593" />



📈 Results
Security: Successfully deployed software without opening Port 22 (SSH), reducing the attack surface.

Auditability: Every step of the deployment was logged in Systems Manager, providing a clear audit trail of software versions across the fleet.

Scalability: Demonstrated a process that can manage 10,000 instances as easily as two.
