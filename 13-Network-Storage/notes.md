## Network Storage

### EFS Architecture

EFS moves the instances closer to a stateless.

EFS is an implementation of NFSv4

EFS Filesystems can be mounted in Linux.

Using Amazon EFS with Microsoft Windows–based Amazon EC2 instances is not supported.

Media can be shared between many EC2 instances.

This is a private service, access is via mount targets inside a VPC.

You can access from on-premises with VPN or DX so long as access is configured.

#### Elastic File System Explained

EFS runs inside a VPC.
EFS includes POSIX permissions. These are made available via mount targets.

For a fully highly available system you need a mount target for each AZ the
system runs in.

You can use hybrid networking to connect to the same mount targets.

![image](https://user-images.githubusercontent.com/88237437/159148835-1fc52854-85ef-4f25-800d-0aaa2a6d807d.png)

#### EFS Exam Powerup

EFS is Linux Only

Two performance modes, **general purpose** (ideal for latency sensitive use cases, web servers, CDNs, home directories, general file serving when using linux instances) and **max I/O performance** mode (high latency - this is a trade-off) - (ideal for highly paralell applications - big data, media processing, sceintific analysis etc.).

General purpose = default for 99.9% of uses

**Bursting** and **provisioned** throughput modes.
- Bursting mode work like GP2 volumes in EBS. More data you store, you get more throuhput
- Provisioned mode - handles throughput requirements separately from the size. This is like IO1 volumes in EBS

Two storage classes available, **standard** (default, this is used to store frequent access files) and **infrequent access** (lower cost).

Lifecycle policies can move data between EFS (just like S3)

### DEMO

Creating a an EFS 

File system settings
![image](https://user-images.githubusercontent.com/88237437/159149181-7f64da62-7da3-4eac-a549-e32c7cf78dcf.png)

Network Access
![image](https://user-images.githubusercontent.com/88237437/159149234-a3fd8f23-85e3-498e-b91c-2d7e7bd1621a.png)

File system policy options
![image](https://user-images.githubusercontent.com/88237437/159149249-b0e3257a-0738-4bfd-8720-23f8f572bdb8.png)





