## Hybrid Enviroment and Migration

### Broder Gateway Protocol (BGP)

![image](https://user-images.githubusercontent.com/88237437/165965301-77331637-7cb9-4512-928a-dc7b87acce30.png)

![image](https://user-images.githubusercontent.com/88237437/165965337-c8b9d5f5-31a6-4a9e-bfe2-d6b311a01486.png)


### AWS Site-to-Site VPN

A logical connection between a VPC and on-premise network encrypted
using IPSec, running over the public internet.

This can be full High Availability if you design it correctly

Quick to provision, less than an hour.

VPNs connect VPCs and on-prem

Virtual Private Gateway (VGW) is the target on one or more route tables

Customer Gateway (CGW)

- logical configuration on AWS
- also the thing that configuration mentions

VPN connection itself stores the config and connects the VGW and CGW

![image](https://user-images.githubusercontent.com/88237437/165967310-ce773981-0cb0-49b1-89a9-9169e5d5ca7f.png)

There is a single point of failure in this architecture, that is the Customer Gateway
![image](https://user-images.githubusercontent.com/88237437/165967460-4665dc1a-1b0d-4af6-9fa8-361b5824963c.png)

![image](https://user-images.githubusercontent.com/88237437/165967861-77e946b7-376f-4d13-9fcc-010e1c2156b6.png)
We introduce two Customer gateways which are using two different routers (uses two different internet connections) in the customer side to make the customer side higly available

Static vs Dynamic VPN
![image](https://user-images.githubusercontent.com/88237437/165968853-494a65d1-2ef0-4ee0-ac62-ba06fde632f3.png)

#### Considerations

Speed Cap on VPN 1.25Gbps

Latency Considerations - this is inconsistent because it uses the public
internet.

Cost - AWS hourly, GB out cost, data cap

Setup of hours or less

Great as a backup especially for Direct Connect (DX)

### AWS Direct Connect (DX)

This is a 1 Gpbs or 10 Gbps Network Port into AWS

This is at a DX Location (1000-Base-LX or 10GBASE-LR)

This is a cross connect to your customer router (requires VLANS/BGP)

You can connect to a partner router if extending to your location.

The port needs to be arranged to connect somewhere else and connect to
your hardware.

This is a single fiber optic cable from the DX port to your network.

VIFS are multiple virtual interfaces (VIFS) over one DX

- Private VIF (VPC)
- Public VIF (Public Zone Services)

Has one physical cable with no high availability and no encryption.

Can take weeks or month for physica cable to be installed.

**Public VIF** is only public services, not public internet.

**Private VIF** is one VPC

![image](https://user-images.githubusercontent.com/88237437/166101943-8f8be3b0-c331-45e8-9e20-e55f30c8120b.png)

DX Port Provisioning is likely quick, the cross-connect takes longer.

Generally use a VPN first then bring a DX in and leave VPN as backup.

40 Gbps with aggregation

It does not use public internet and provides consistently low latency.

DX provides NO ENCRYPTION and needs to be managed on a per application basis.

![image](https://user-images.githubusercontent.com/88237437/166102001-468a1b2a-6977-4855-9ffb-c356bfebc21c.png)
Encryption can be added by using the site to site VPN on top of the DX

#### DX Resilliancy

##### Evolution 1: Not Highly Available
![image](https://user-images.githubusercontent.com/88237437/166101762-c95e7dc0-42f8-4d1a-be24-f0b25f1ddc1a.png)

##### Evolution 2: Somewhat HA
![image](https://user-images.githubusercontent.com/88237437/166101796-9e397a7e-025a-4cd0-ba1c-9256d92d87fe.png)

##### Evolution 3: Better HA
![image](https://user-images.githubusercontent.com/88237437/166101813-81c14c59-63ca-43c3-9b1a-b74f8b4463e9.png)

##### Evolution 4: Extreme HA
![image](https://user-images.githubusercontent.com/88237437/166101826-64edec1e-2d51-48be-80d4-62a257993226.png)


### AWS Transit Gateway (TGW)

Network transit hub to connect VPCs to on premises networks

Significantly reduces network complexity.

There is a single network object which makes it HA and scalable.

Attachment to other network types.

VPC attachments are configured with a subnet in each AZ where service
is required.

You can use these for cross-region peering attachment.

Can share between accounts using AWS RAM

### Storage Gateway

Hybrid Storage Virtual Application (On-premise)

Scenarios

Extend storange of File and Volume Storage into AWS.
Keep volume storage backups into AWS.
Tape backups into AWS. Can act as emulation layer.

Migration of extisting infrastructure into AWS slowly.

- Tape Gateway (VTL) Mode
  - Virtual Tapes are stored on S3

- File Mode : SMB and NFS
  - File Storage Backed by S3 Objects

- Volume Mode (Gateway Stored)
  - Block Storage backed by S3 and EBS
  - Great for disaster recovery
  - Data is kept locally
  - Awesome for migrations

- Volume Mode (Cache Mode)
  - Data as added to gateway is not stored locally.
  - Backup to EBS Snapshots
  - Primarily stored on AWS
  - Great for limited local storage capacity.

### Snowball / Edge / Snowmobile

Move large amounts of data IN and OUT of AWS

Physical storage the size of a suitcase or truck.

Ordered from AWS, use, then return.

#### Snowball

Anything on Snowball uses KMS
50TB or 80TB Capacity
1 Gbps or 10 Gbps
This makes sense from 10 TB to 10 TB and over many premises.
This only includes storage

#### Snowball Edge

Both storage and compute
Larger capacity vs snowball
10 Gbps or up to 100 Gbps

Storage optimized (with EC2) includes 1TB SSD
Compute optimized
Compute with GPU as above with GPU

These are great for remote sites when ingestion is needed

#### Snowmobile

Portable data center within a shipping container on a truck.

This is a special order and is not available in high volume.
Ideal for single location where 10 PB+ is required.

Up to 100 PB per snowmobile.

This is not economical for multi-site for sub 10 PB

### AWS Directory Service

This is a managed service with lots of use cases.

Stores objects, users, groups, computers, servers, file Shares with
a structure.

Multiple trees can be grouped into a forest.

Commonly used in Windows Environments.

Sign in to multiple devices with the same username/password provides
central management for assets.

#### AWS managed implementation

Runs within a VPC as a private service.

Provides HA by deploying into multiple AZs.

Certain services in AWS need a directory, Amazon Workspaces.

To join EC2 instances to a domain you need a directory.

Can be isolated or integrated with existing on-prem system.

Could act as a proxy back to on-premises.

#### Picking the Modes

Simple AD should be default

Microsoft AD is anything with Windows or if it needs a trust relationship
with on-prem. This is not an emulation.

AD Connector - Use AWS services without storing any directory info in the
cloud, it proxies to your on-prem directory.

### AWS DataSync

Data transfer service TO and FROM AWS.

This is used for migrations or for large amounts of data processing transfers.

Designed to work at huge scales. Each agent can handle 10 GB and each job
can handle 50 million files.

This keeps metadata.

Has built in data validation to ensure the data matches.

Each agent is about 100 TB per day.

Can use bandwidth limiters to avoid customer impact

Has incrememetal and scheduled transfer options

Compression and encryption is also supported

It does data validation and automatic recovery from transit errors.

AWS service integration with S3, EFS, FSx for Windows servers.

Pay as you use product.

The data is encrypted in transit and all of the data transfer in parts.

#### Components

Task is a job within datasync and defines what is going from where to where

Agent is software to read and write to on prem such as NFS or SMB

Location is the FROM and TO

### FSx for Windows File Server

Fully managed native windows file servers/shares
Designed for integration with windows environments

Integrates with Directory Service or Self-Managed AD

Single or Multi-AZ within a VPC.

Can perform on-demand and scheduled backups.

File systems can be access using VPC, Peering, VPN, Direct Connect. Native
windows filesystem or Directory Services.

#### Words to look for

VSS - User Driven Restores
Native file system accesible over SMB

Windows permissions model

Product supports DFS, scale out file share.

Managed - no file server admin

Integrates with DS and your own directory.
