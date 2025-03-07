## ELASTIC COMPUTE CLOUD (EC2) BASICS

### Virtualization 101

EC2 provides Infrastructure as a Service (IAAS Product)

Servers are configured in three sections without virtualization

- CPU hardware
- Kernel
  - Operating system
  - Runs in Privileged mode and can interact with the hardware directly
- User Mode
  - Runs applications
  - Can make a **system call** to the Kernel to interact with the hardware
  - If an app tries to interact with the hardware without a system call, it
  will cause a system error and can crash the server or at minimum the app
![image](https://user-images.githubusercontent.com/88237437/157149408-feb1fe70-2a36-4579-8663-4adbd4d1b970.png)
 

#### Emulated Virtualization - Software Virtualization

A host OS operated on the hardware and it included a hypervisor
The software ran in priviledged mode and had full access to the hardware on the
server.

Guest operating systems were wrapped in a virtual machine and had devices
mapped into their OS to emulate real hardware. Drivers such as graphics cards
were all software emulated to allow the process to run properly.

The guest OS still believed they were running on real hardware and tried
to take control of the hardware. The areas were not real and only allocated
space to them for the moment.

The hypervisor performs **binary translation**. Any calls attempted to make
are intercepted and translated in software on the way. The guest OS needs no
modification, but it slows the guest OS down a lot.

#### Para-Virtualization

Guest OS are still running in containers, except they do not use slow
binary translation. This only works on a small subset of OS that can be
modified. The OS is modified to change the **system calls** to **user calls**.
Instead of calling on the hardware, they call on the hypervisor called
hypercalls. Areas of the OS call the HV instead of the hardware. They need
to be modified for the particular vendor peforming the para-virtualization.

The OS becomes virtualization aware and got faster, but this was still a
software trick.

#### Hardware Assisted Virtualization

The physical hardware itself is virtualization aware. The CPU has specific
functions so the hypervisor can come in and support. When guest OS try to run
priviledged instructions, they are trapped by the CPU and do not halt
the process. They are redirected to the hypervisor from the hardware.

What matters for a virtual machine is the input and output operations such
as network transfer and disk IO. The problem is multiple OS try to access
the same piece of hardware but they get caught up on sharing.

#### SR-IOV (Singe Route IO virtualization)

Allows a network or any addon card to present itself as many mini cards.
As far as the hardware is concerned, they are real dedicated cards for their
use. No translation needs to be done by the hypervisor. The physical card
handles their own process internally. In EC2 this feature is called
**enchanced networking**. This means less CPU usage for the guest.

### EC2 Architecture and Resilience

- EC2 instances are virtual machines (OS+Resources (vCPU, Memory, Local Storage, NW storage, GPU etc.))
- Run on EC2 hosts
- Shared hosts or dedicated hosts
  - Every customer is isolated even on the same shared hardware
  - Dedicated hosts pay for entire host, don't pay for instances
- AZ resilient service. They run within only one AZ system. If AZ fails then the host fails and as a result of that the instances will be failled

Example AZ with an EC2 host

![image](https://user-images.githubusercontent.com/88237437/157837293-82a5a460-09e1-43bd-b429-688ff9baaf03.png)

- Run within single AZ
- Local hardware such as CPU and memory
- Also have instance store
  - Temporary
  - If instance moves hosts, the storage is lost
- Networking
  - Storage networking
  - Data networking

When instances are provisioned within a specific subnet within a VPC
A primary elastic network interface is provisioned in a subnet which
maps to the physical hardware on the EC2 host. Subnets are also within
one specific AZ. Instances can have multiple network interfaces, even within
different subnets so long as they're within the same AZ.

EC2 can make use of remote storage, Elastic Block Store (EBS).

EBS also runs within a specific AZ. You can't access them cross zone.

EBS allows you to allocate volumes of persistent storage to instances
within the same AZ.

An instance runs on a specific host. If you restart the instance
it will stay on that host.

Instances stay on the host until either

- The host fails or is taken down by AWS
- The instance is stopped and then started, different than restarted.

The instance will be relocated to another host in the same AZ. Instances
cannot move to different AZs. Everything about their hardware is locked within
one specific AZ.

A migration is taking a copy of an instance and moving it to a different AZ.

In general instances of the same type and generation will occupy the same host.
The only difference will generally be their size.

#### EC2 Strengths

Traditional OS+Application compute need. A specific OS with a specific hardware
setup.

Long running compute needs. Many other AWS services have run time limits.

Server style applications

- things waiting for network response
- burst or stead-load
- monolithic application stack
  - middle ware or specific run time components
- migrating application workloads or disaster recovery
  - existing applications running on a server and a backup system to intervene

### EC2 Instance Types

Raw CPU, memory, local storage capacity, and type
Resource ratios: Some might have more CPU while others have more storage.
Storage and Data network Bandwidth

Influences the architecture and vendor.
AMD CPU vs Intel CPU

Influcences features and capabilities with that instance

#### EC2 Categories

- General Purpose - default steady state workloads with even resources

- Compute Optimized - Media processing, scientific modeling and gaming

- Memory Optimized - Processing large in-memory datasets

- Accelerated Computing - Hardware GPU, FPGAs

- Storage Optimized - Large amounts of super fast local storage. Massive amounts
of IO per second. Elastic search and analytic workloads.
![image](https://user-images.githubusercontent.com/88237437/158585107-aab6f620-d977-4c96-b9be-ab16de2c32c3.png)


#### Naming Scheme

![image](https://user-images.githubusercontent.com/88237437/157857125-5f8ba310-250f-4d9a-b074-fbc2406d8cec.png)


R5dn.8xlarge - whole thing is the instance type. When in doubt give the
full instance type

- 1st char: Instance family. No expectation to remember all the details
- 2nd char: Instance generation. Generally always select the newest generation.
- char after period: Instance size. Memory and CPU considerations.
  - often easier to scale system with a larger number of smaller instance sizes
- 3rd char - before period: additional capabilities
  - a: amd cpu
  - d: nvme storage
  - n: network optimized
  - e: extra capacity for ram or storage

### Storage Refresher

#### Key Terms

**Direct** local attached storage - these are physical disks on EC2
If the disk lost or hardware fails or instance moved between hosts, the storage will be lost. 

**Network** attached storage - volumes delivered over the network (EBS)
Highly resillient

**Ephemeral** storage - temporary storage. Instant store attached to EC2 host

**Persistent** storage - permanent storage that lives on past the lifetime
of the instance. EBS is persistent storage.

#### Three types of storage

**Block Storage** - volume presented to the OS as a collection of blocks. No
structure beyond that. These are mountable and bootable. The OS will
create a file system on top of this, NTFS or EXT3 and then it mounts
it as a drive or a root volume on Linux. Spinning hard disks or SSD. This
could also be delivered by a physical volume. Has no built in structure.
You can mount an EBS volume or boot off an EBS volume.
![image](https://user-images.githubusercontent.com/88237437/158586320-06d00a78-34b6-46e1-8ba9-b76208e8c114.png)


**File Storage** - Presented as a file share with a structure. You access the
files by traversing the storage. You cannot boot from storage, but you
can mount it.


**Object Storage** - It is a flat collection of objects. An object can be anything
with or without attached metadata. To retrieve the object, you need to provide
the key and then the value will be returned. This is not mountable or
bootable. It scales very well and can have simultanious access.

**Usage**:

Block Storage - bootable storage, high performance inside OS

File Storage - share file system across multiple servers or clients

Object Storage - large access to read/write object data at scale

#### Storage Performance

IO Block Size - size of the wheels. This determines how to split up the data.
IOPS - speed of an engine rev (RPM). How many reads or writes a storage
system can accomidate in a second.
Throughput - end speed of the race car. This can be influenced by the network
speed for network storage. Expressed in MB/s (megabyte per second).

![image](https://user-images.githubusercontent.com/88237437/158588008-ed78c7d6-0a90-4769-8a8b-f5eab55e26ac.png)

This isn't the only part of the chain, but it is a simplification.

A system might have a throughput cap. The IOPS might decrease as the block
size increases.

### Elastic Block Store (EBS)

Allocate block storage **volumes** to instances.
Volumes are isolated to one AZ.

- The data is highly available and resilient for that AZ.
- All of the data is replicated within that AZ. The entire AZ must have
a major fault to go down.
- Different physical storage types available (SSD/HDD)
- Varying level of performance (IOPS, T-put)
- Billed as GB/month.
  - If you provision a 1TB for an entire month, you're billed as such.
  - If you have half of the data, you are billed for half of the month.

![image](https://user-images.githubusercontent.com/88237437/158589525-b525fcce-7bdd-4837-9427-435a248a953c.png)

Architecture
![image](https://user-images.githubusercontent.com/88237437/158589896-3558281d-c716-4330-9edd-9e9bab14da9a.png)

### EBS Volume Types

Four types of Volumes:

- General purpose SSD (gp2)
- Provisioned IOPS SSD (io1)
- T-put optimized HDD (st1)
- Cold HDD (sc1)

Each volume has a dominant performance attribute

SSD based focus on maximum IOPS such as a database
HDD based focus on throughput for logs or media storage.

#### General Purpose SSD (gp2)

![image](https://user-images.githubusercontent.com/88237437/158726723-b6de0df4-f9b1-49aa-a349-3bb597c7fb81.png)

Uses a performance bucket architecture based on the iops it can deliever.

![image](https://user-images.githubusercontent.com/88237437/158727867-d9d57b24-3380-4697-bc3e-d14bde53230d.png)

The GP2 starts with 5,400,000 IOPS allocated. It is available instantly.

You can consume the capacity quickly or slowly over the life of the volume.

The capacity is filled back based upon the volume size.

Min of 100 iops added back to the bucket per second.
Above that, there are 3 IOPS/GiB of volume size. The max is 16,000 IOPS.

This is the **baseline performance**

This should be the default for boot volumes and some data volumes.

Can only be attached to one volume at a time.
![image](https://user-images.githubusercontent.com/88237437/158729136-c5a755d7-4c45-48f1-ad51-be44be29666e.png)


GP3
![image](https://user-images.githubusercontent.com/88237437/158727962-5117fc1f-601a-4a59-a369-dacd59b8ad84.png)


#### Provisioned IOPS SSD (io1)

![image](https://user-images.githubusercontent.com/88237437/158752132-e8040cbf-dfa4-4a10-b63e-96e78b42bda0.png)
 
50:1 IOPS to GiB Ratio

You pay for capacity and the IOPs set on the volume. This is good if your
volume size is small but need a lot of IOPS.

64,000 is the max IOPS per volume assuming 16 KiB I/O.

Good for latency sensitive workloads such as mongoDB.

Multi-attach allows them to attach to multiple EC2 instances at once.

If it mentions high IOPS, or mentioning latency. Small volume sizes.

#### HDD Volume Types

![image](https://user-images.githubusercontent.com/88237437/158752960-eb548e0b-3e85-4b9c-af65-8834db276ff2.png)

st1 - throughput
sc1 - cold

Both have 4 shared characteristics

- great value
- great throughput
- 500 GiB - 16 TiB
- Neither can be used for instance boot volumes. Cannot boot from them.

Good for price conscious.
Great for high throughput vs IOPs.

Good for streaming data on a hard disk. Media conversion with large amounts of
storage.

Frequently accessed high throughput intensive workload

- log processing
- data warehouses

The access patterns should be sequential

##### st1

Starts at 1 TiB of credit per TiB of volume size.

40 MB/s baseline per TiB
Burst of 250 MB/s per TiB
Max t-put of 500 MB/s

##### sc1

Designed for less frequently accessed data

Fills slower:

12 MB/s baseline per TiB
Burst of 80 MB/s per TiB
Max t-put of 250 MB/s

There is a massive inefficency for small reads and writes. These are based on
large block sizes of data in a sequential way

#### Exam Power Up

Volumes are created in an AZ, isolated in that AZ
If an AZ fails, the volume is impacted.
Highly available and resilient in that AZ. The only reason for failure is
if the whole AZ fails.
Generally one volume to one instance, but multi-attach occurs.
Has a GB/m fee regardless of instance state.
EBS maxes at 80k IOPS per instance and 64k vol (io1)
Max 2375 MB/s per instance, 1000 MiB/s (vol) (io1)

### EC2 Instance Store

Local physical storage that instances can utilize attached to an instance.

**block storage** devices. They're just like EBS but they're local.

The volumes are physically connected to one EC2 host. They are isolated to
that one specific host.

![image](https://user-images.githubusercontent.com/88237437/158753815-5e32e173-2641-4a44-acc5-574451943f75.png)

Instances on that host can access them.

Highest storage performance in AWS.

They are included in instance price, use it or lose it.

They can be attached ONLY at launch. Cannot be attached later.

Architecturally, each instance has a collection of ephemeral volumes that are
locked to that specific host. Even though an instance is been alocated a
specific number of volumes, the data is locked to the host.

![image](https://user-images.githubusercontent.com/88237437/158753937-fb27dce0-8a65-47a0-9778-002afad10e09.png)

Instances can move between hosts for many reasons:

- If an instance is stopped and started, that migrates hosts.
- If a host undergoes AWS maintenance, it will be wiped.
- If you change the type of an instance, these will be lost.
- If a physical hardware fails, then the data is gone.

The number, size, and performance of instance store volumes vary based on the
type of instance used. Some instances do not have any instance store volumes
at all.

#### Instance Store Performance

This is much higher than EBS can provide. These volumes
perform at much higher volumes than EBS.

![image](https://user-images.githubusercontent.com/88237437/158754295-13db7f54-788b-4fdd-a934-e759ee5aea1f.png)


#### Exam Powerup

- Instance store volumes are local to EC2 host.
- Can only be added at **launch time**. Cannot be added later.
- Any data on instance store data is lost if it gets moved, resized or hardware failure.
- Highest data performance in all of AWS.
- You pay for it anyway, it's included in the price.
- **TEMPORARY**

### EBS vs Instance Store

If the read/write can be handled by EBS, that should be default.

![image](https://user-images.githubusercontent.com/88237437/158755157-646680dc-bc5c-4832-b078-269c5f36312e.png)

When to use EBS

- Highly available and reliable in an AZ. Can self correct against HW issues.
- Persist independently from EC2 instances.
  - Can be removed or reattached.
  - You can terminated instance and keep the data.
- Multi-attach feature of **io1**
  - Can create a multi shared volume.
- Region resilient backups.
- Require up to 64,000 IOPS and 1,000 MiB/s per volume
- Require up to 80,000 IOPS and 2,375 MB/s per instance

When to use Instance Store

- Great value, they're included in the cost of an instance.
- More than 80,000 IOPS and 2,375 MB/s
- If you need temporary storage, or can handle volativity.
- Stateless services, where the server holds nothing of value.
- Rigid lifecycle link between storage and the instance.
  - This ensures the data is erased when the instance goes down.

![image](https://user-images.githubusercontent.com/88237437/158755708-2f517cb0-cada-4b17-8416-80a36440ee5b.png)

### Snapshots, restore, and fast snapshot restore

EBS Snapshots

Efficent way to backup EBS volumes to S3. Protect data against AZ issues.
Can be used to migrate data between hosts.

The data becomes region resilient.

Snapshots are incremental volume copies to S3.

The first is a **full copy** of `data` on the volume. This can take some time.

EBS won't be impacted, but will take time in the background.

Future snaps are incremental, consume less space and are quicker to perform.

If you delete an incremental snapshot, it moves data to ensure subsequent
snapshots will work properly.

Volumes can be created (restored) from snapshots. Snapshots can be used to
move EBS volumes between AZs. Snapshots can be used to migrate data between
volumes.

![image](https://user-images.githubusercontent.com/88237437/158756707-5cd1ee44-ec41-484f-bf0a-4bfba1af7b3f.png)

#### Snapshot and volume performance

When creating a new EBS volume without a snapshot, the performance is
available immediately.

If you restore a snapshot, it does it lazily.

If you restore a volume, it will transfer it slowly in the background.

If you attempt to read data that hasn't been restored yet, it will
pull it from S3, but this will achieve lower levels of performance than reading
from EBS directly.

You can force a read of all data immediately. This is something you would do
right when using a volume.

You could also use Fast Snapshot Restore (FSR) - Immediate restore

You can have up to 50 snaps per region. This is set on the snap and AZ.

FSR is not free and can get expensive with lost of different snapshots.

You can force the same response by reading every block manually using DD or
another tool in the OS.

#### Snapshot Consumption and Billing

They are billed using a GB/month metric.

20 GB stored for half a month, represents 10 GB-month.

This is used data, not allocated data. If you have a 40 GB volume but only
use 10 GB, you will only be charged for the allocated data. This is not true
for EBS itself.

The data is incrementally stored which means doing a snapshot every 5 minutes
will not necessarily increase the charge as opposed to doing one every hour.

![image](https://user-images.githubusercontent.com/88237437/158757560-bdc2e30a-364b-416a-9f3f-5fcb185e99ac.png)

#### EBS Encryption

Provides at rest encryption for block volumes and snapshopts.

When you don't have EBS encryption, the volume is not encrypted.
The physical hardware itself may be performing at rest encryption, but
that is a seperate thing.

When you set up an EBS volume initially, EBS uses KMS and a customer master key.
This can be the ebs default (CMK) which is refered to as `aws/ebs` or it
could be a customer managed CMK which you manage yourself.

That key is used by EBS when an encrypted volume is created. The CMK
generates an encrypted data encryption key which is stored with the volume with
on the physical disk. This key can only be encrypted by KMS when a role with
the proper permissions makes the request.

When the volume is first used, EBS asks CMS to decrypt the key and stores
the decrypted key in memory on the EC2 host while it's being used. At all
other times it's stored on the volume in encrypted form.

When the EC2 instance is using the encrypted volume, it can use the
decrypted data encryption key to move data on and off the volume. It is used
for all cryptographic operations when data is being used to and from the
volume.

![image](https://user-images.githubusercontent.com/88237437/158780020-467a8c99-206c-4be9-a5c3-bb3b1188cbdc.png)


When data is stored at rest, it is stored as **Ciphertext**.

If the EBS volume is ever moved, the key is discarded.

If a snapshot is made of an encrypted EBS volume, the same data encryption
key is used for that snapshot. Anything made from this snapshot is also
encrypted in the same way.

![image](https://user-images.githubusercontent.com/88237437/158780198-15a23252-f232-4370-8227-7f457ccad9e0.png)


Everytime you create a new EBS volume from scratch, it creates a new
data encryption key.

##### EBS Exam Power Up

- AWS accounts can be set to encrypt EBS volumes by default.
- It will use the default CMK unless a different one is chosen.
- Each volume uses 1 unique DEK (data encryption key)
- Snapshots and future volume use the same DEK
- Can't change a volume to NOT be encrypted. You could mount an unencrypted volume and copy things over but you can't change the origina volume.
- The OS isn't aware of the encryption, there is no performance loss. The data uses AES256
- If an exam question does not use AES256, or it suggests you need an OS to encrypt or hold the keys, then you need to perform full disk encryption at the operating system level.
- You can perform full disk encryption on an unencrypted or encrypted EBS volume.

### EC2 Network Interfaces, Instance IPs and DNS

An EC2 instance starts with at least one ENI - elastic network interface.

An instance may have ENIs in seperate subnets, but everything must be
within one AZ.

When you launch an instance with Security Groups, they are on the
network interface and not the instance.

#### Elastic Network Interface

Has these properties

- MAC address
- Primary IPv4 private address
  - From the range of the subnet the ENI is within.
  - 10.16.0.10 will be static and not change for the lifetime of the instance
  - Given a DNS name that is associated with the address
    - ip-10-16-0-10.ec2.internal
    - only resolvable inside the VPC and always points to private IP address
- 0 or more secondary private IP addresses

- 0 or 1 public IPv4 address given two ways
  - Instance must manually be set to recieve an IPv4 addr
  - Default settings into a subnet which automatically allocates an IPv4
  - Dynamic IP that is not fixed
  - If you stop an instance the address is deallocated.
  - When you start up again, it is given a brand new IPv4 address
  - Restarting the instance will not change the IP address
  - Changing between EC2 hosts will change the address
  - They are allocated a public DNS name.
  - Public DNS name will resolve to the primary
  private IPv4 address of the instance
  - Outside of the VPC, the DNS will resolve to the public IP address.
  - Allows one single DNS name for an instance, and allows traffic to resolve
  to an internal address inside the VPC and the public will resolve to a public
  IP address.

- 1 elastic IP per private IPv4 address
  - Can have 1 public elastic interface per private IP address on this interface
  - Allocated to your AWS account
  - Can associate with a private IP on the primary interface or
  on the secondary interface.
  - If you are using a public IPv4 and assign an elastic IP, the original IPv4
  address will be lost. There is no way to recover the original address.

- 0 or more IPv6 address on the interface
  - These are by default public addresses
- Security groups
  - applied to network interfaces
  - will impact all IP addresses on that interface
  - if you need different IP addresses impacted by different security
  groups, then you need to make multiple interfaces and apply different
  security groups to those interfaces
- Source / destination checks
  - if traffic is on the interface, it will be discarded if it is not
  from going to or coming from one of the IP addresses

Secondary interfaces function in all the same ways as primary interfaces except
you can detach interfaces and move them to other EC2 instances.

#### Exam Power Ups

Legacy software is licensed using a mac address.
If you provision a secondary ENI to a specific license, you can move
around the license to different EC2 instances.

Multi homed (subnets) management and data.

Different security groups are attached to different interfaces.

The OS doesn't see the IPv4 public address.

You always configure the private IPv4 private address on the interface.

Never configure an OS with a public IPv4 address.

IPv4 Public IPs are Dynamic, starting and stopping will kill it

Public DNS for a given instance will resolve to the primary private IP
address in a VPC. If you have instance to instance communication within
the VPC, it will never leave the VPC. It does not need to touch the internet
gateway.

### Amazon Machine Image (AMI)

Images of EC2.

AMI's can be used to launch EC2 instance.

- When you launch an EC2 instance, you are using an Amazon provided AMI.
- Can be Amazon or community provided
- Marketplace (can include commercial software)
  - Will charge you for the instance cost and an extra cost for the AMI
- Regional, unique ID
  - ami-`random set of chars`
- Controls permissions
  - Default only your account can use it
  - Can be set to be public
  - Can have specific AWS accounts on the AMI
- Can create an AMI from an existing EC2 instance to capture the current config

#### AMI Lifecycle

![image](https://user-images.githubusercontent.com/88237437/159104990-220fb123-e8cf-4bb7-a6b0-44adeaa45223.png)

Launch

EBS volumes are attached to EC2 devices using block IDs

- BOOT /dev/xvda
- DATA /dev/xvdf

Configure

- Can customize the instance from applications or volume sizes

Create Image

- Once it has been customized, an AMI can be created from that
- AMI contains:
  - Permissions: who can use it
  - EBS snapshots are created from attached volumes
    - Block device mapping links the snapshot IDs and a device ID for
    each snapshot.

Launch

When launching an instance, the snapshots are used to create new EBS
volumes in the availability zone of the EC2 instance and contain the same
block device mapping.

#### Exam Powerups

AMI can only be used in one region
AMI Baking: creating an AMI from a configuration instance.
An AMI cannot be edited. If you need to update an AMI, launch an instance,
make changes, then make new AMI
Can be copied between regions
Remember permissions by default are your account only
Billing is for the storage capacity for the EBS snapshots the AMI references

### EC2 Pricing Models

#### On-Demand Instances

- Hourly rate based on OS, size, options, etc
- Billed in seconds (60s min) or hourly
  - Depends on the OS
- Default pricing model
- No long-term commitments or upfront payments
- New or uncertain application requirements
- Short-term, spiky, or unpredictable workloads which can't tolerate
any disruption.

![image](https://user-images.githubusercontent.com/88237437/159107745-7dee6ba5-810d-40bd-b8f7-feb1909641f7.png)


#### Spot Instances

Up to 90% off on-demand
Depends on the spare capacity
You can set a maximum hourly rate in a certain AZ in a certain region.
If the max size you set is above the spot price, you pay for the instance.
As the spot price increases, you'll keep paying until the price increases.
Once this price increases too much, it will terminate the instance.
Great for data analytics when the process can occur later.

![image](https://user-images.githubusercontent.com/88237437/159107867-f46b904d-a0ae-4073-9c0d-5105e4365273.png)


#### Reserved Instance

Up to 75% off on-demand
The trade off is commitment.
You're buying capacity in advance for 1 or 3 years.
Flexibility on how to pay

- All up front
- Partial upfront
- No upfront

![image](https://user-images.githubusercontent.com/88237437/159108015-39676eae-28fe-4890-818e-b093220b250a.png)

Best discounts are for 3 years all up front
Reserved in region, or AZ with capacity reservation
Reserved takes priority for AZ capacity.
Can perform scheduled reservation when you can commit to specific time windows.

If you have a known stead state usage, email usage, domain server.
Cheapest option with no tolerance for distruption.

Other types of reserved instances

- Scheduled Reserved Instances
- ![image](https://user-images.githubusercontent.com/88237437/159108415-c02d5d9b-23fc-41a7-a4fd-1786537eae0e.png)

Capacity Reservations
![image](https://user-images.githubusercontent.com/88237437/159108517-eaf9c390-8762-4b41-80ee-03a211878eba.png)

EC2 Savings Plan
![image](https://user-images.githubusercontent.com/88237437/159108620-5c417176-daf2-4dd0-a890-1e9ba7353cc4.png)

#### Dedicated Hosts
![image](https://user-images.githubusercontent.com/88237437/159108077-fdabbbab-fed2-4e17-b103-491719e64fed.png)

Used for strict licencing requirments

#### Dedicated Instances
![image](https://user-images.githubusercontent.com/88237437/159108239-5acd6d67-0368-45c1-8a9f-7d01a4baa332.png)

Instances are running in dedicated hardware.

Used when there is a requirement to use dedicated hardware.

### Instance Status Checks and Autorecovery

Every instance has two high level status checks
![image](https://user-images.githubusercontent.com/88237437/159108732-29ba2212-00a8-43c7-a60f-50217146a21c.png)


#### System Status Checks

Failure of this check could indicate software or hardware problems of the EC2
service or the host.

#### Instance Status Checks

This is specific to the file system or has a corrupted Kernel

Assuming you haven't launched an instance, this is a problem and needs to be
fixed.

#### Create Status Check Alarm

This feature has four options

- Recover this instance: can be a number of steps depending on the failure
- Stop this instance
- Terminate this instance: useful in a cluster
- Reboot this instance:

### Shutdown, Terminate & Termination Protection

Enable termination protection
![image](https://user-images.githubusercontent.com/88237437/159108903-60a9793e-3a79-4f57-a6b8-00b7aef067e1.png)

The user should have **disableApiTermination** permission to disable the termination protection

Shutdown Behavior
![image](https://user-images.githubusercontent.com/88237437/159109012-b17bfc52-359e-4770-ae35-fd80002a4b98.png)

### Horizontal and Vertical Scaling

#### Vertical Scaling

![image](https://user-images.githubusercontent.com/88237437/159109110-be8ee6a5-d08e-4cb1-8d4a-7c8a7c271691.png)

As customer load increases, the server may need to grow to handle more data.
The server can increase in capacity, but this will require a reboot.

Often times vertical scaling can only occur during planned outages.

Larger instances also carry a $ premium compared to smaller instances.

There is an upper cap on performance - instance size.

No application modification is needed.

Works for all applications, even monoliths (all code in one app)

#### Horizontal Scaling

![image](https://user-images.githubusercontent.com/88237437/159109169-0cb7a07e-5617-4f3a-9adc-89c02ca0da6b.png)

As the customer load increases, this adds additional capacity.
Instead of one running copy of an application, you can have multiple versions
running on each server.
This requires a **load balancer**.

![image](https://user-images.githubusercontent.com/88237437/159109188-a03bc2ca-a042-4c42-b8b4-d0d9f276adcc.png)

When customers try to access an application, the load balancer ensures the
servers get equal parts of the load.

Sessions are everything.
With horizontal scaling you can shift between instances equally.
This requires either application support or off-host sessions.
This means the servers are **stateless**, the app stores session data elsewhere.

No distruption while scaling up or down.

No real limits to scaling.

Uses smaller instances so you pay less, allows for better granulairity.

### Instance Metadata

EC2 service provides data to instances
Accessible inside all instances

Memorize **<http://169.254.169.254/latest/meta-data/>

Meta-data contains:

- enviroment the instance is in
- networking is the big reason why
- authentication information
  - instances can be used to gain access on other services
- user-data
- NO AUTHENTICATION or ENCRYPTED
  - Anyone who can gain access and it can and will get exposed
  - Can be restricted by local firewall

Example:
Get public IPv4 Address
![image](https://user-images.githubusercontent.com/88237437/159109938-1dc89d4d-8c98-4c23-b87d-537edb233562.png)

Get public host name
![image](https://user-images.githubusercontent.com/88237437/159109952-d6f0d381-7e04-4da2-b5e1-c19524554f5b.png)

ec2-metadata tool
![image](https://user-images.githubusercontent.com/88237437/159109970-4766a58d-d7c7-4bd9-a12d-12aa71745318.png)

