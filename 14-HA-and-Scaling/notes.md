## HA and Scaling

### Reginal and Global AWS Architecture

![image](https://user-images.githubusercontent.com/88237437/159150260-24207173-6a65-4eda-b081-a7c046ddb2b8.png)

### Load Balancing Fundamentals

#### ELB Evalution

Three types of load balancers available in AWS
- Classic Load Balancer (CLB)
- Application Load Balancer (ALB)
- Network Load Balancers (NLB)

Two versions, v1 and v2. Avoid v1 and proceed with v2

Classic Load Balancer (CLB) is v1 product - Introduced in 2009

CLB supports HTTP and HTTPs but it is not really a layer 7 LB. CLB support only one SSL per CLB.

Application Load Balancer (ALB) is a v2 product - supports HTTP, HTTPs and WebSocket protocols

Network Load Balancer (NLB) is also a v2 product - supports TCP, TLS and UDP protocols. This is the type of LB we would pick if the networking protocol is not HTTP or HTTPs. e.g.: Email servers, SSH server, Games which would use custom protocols

In general, v2 are faster, cheaper, support target groups and rules

#### ELB Architecture

The job of LB is to accept the connections from the customers and distribute those connection to any registered backend computes. It means the user is abstracted away from the physical infrastructure. The amount of infrastructre can increase or decrease without effecting the customer. Since the physical infrastructre is hidden, it can fail and can be reparied without affecting the customer. 

![image](https://user-images.githubusercontent.com/88237437/159150802-957af3f5-16b0-4562-9076-e6652af5547d.png)

- We have two AZs, AZ A and AZ B
- In those AZs we have public and private subnets
- ![image](https://user-images.githubusercontent.com/88237437/159150843-c414e542-4343-4463-b147-6eb4d70ea2b5.png)
- User Bob wants to access AWS services. We place LBs and Bob connect to those LBs
- ![image](https://user-images.githubusercontent.com/88237437/159150888-b4ed088c-0e8a-4a60-9a75-1ad52f156785.png)
- ALB supports not only EC2 but other AWS compute services as well
- When we define a LB we have to consider it supports IPv4 only or dual stack (IPv4 and IPv6)
- When provisioning an LB, we have to pick which are the AZs and only one subnet in each of those AZs
- ELB will deploy a node in each subnet
- ![image](https://user-images.githubusercontent.com/88237437/159151073-da2a7970-d8e1-4f09-908b-51868c4fdee5.png)
- When ELB is created it is created with single DNS record. This resolves to all the ELB Nodes configured with the LB
- ![image](https://user-images.githubusercontent.com/88237437/159151271-ff1a8ec2-6c7e-4616-b9bd-dc3d61b4fd68.png)
- These nodes are highly available. If one nodes fails it replaced. If the traffic/load increases, addtional nodes will be created inside subnets configured to the LB
- When creating a LB it is important to decide whether it is internet facing or internal. This controlls the IP addressing of the LB nodes.
- If the LB is internet facing - Nodes have both Public and Private IPs
- If the LB is internal - Nodes only have Private IPs
- Nodes are configured with listeners which accept traffic on a port and protocol and communicate with targets on a port and protocol
- Listeners (belong to an internet facing LB) can be configured to access both public and **private** EC2 instances
- ![image](https://user-images.githubusercontent.com/88237437/159151508-2637b96a-bacd-4a45-9eb3-df43165ad09e.png)
- LBs need 8 or more free IP addresses per subnet. This means we can use /28 subnet (contains 16 IPs, out of that 5 is allocated for other things, in result there will be 11 IPs can be used for LB). But AWS suggest to use /27 subnet due to scalability reasons.
- Internal LBs are generally use to separate the application tiers
- ![image](https://user-images.githubusercontent.com/88237437/159151720-e8ac516b-03e4-44e8-b8ad-409e5cd38be0.png)

![image](https://user-images.githubusercontent.com/88237437/159151809-05fea750-ee76-444a-b3b7-79b24c4c6fed.png)

LBs allow each tier to sacle indepndently

**Cross Zone LB**

Each EC2 instance in AZ A gets 12.5% load where as the singe instance located in AZ B gets 50% of the load.
![image](https://user-images.githubusercontent.com/88237437/159151950-8cdfe887-eae3-4571-bde0-b5c3eca713a3.png)

Cross-Zone LB distributes the load
![image](https://user-images.githubusercontent.com/88237437/159151995-4d8fd35d-4637-4ab8-807e-9b010c5e89d6.png)

*For ALB this feature is enable by default*

#### Others

Without load balancing, it is difficult to scale.

The user connects to a load balancer that is set to listens on port 80 and 443

Within AWS, the ports the load balancer will listen to is called
the **listener**.

The user is connected to the load balancer and not the actual server.

Behind the load balancer, there is an application server. At a high
level when Bob connects to the load balancer, it distributes that to
servers on the application server.

As long as 1+ servers are available, the LB is operational. Clients
shouldn't see errors.

#### LB Exam Powerup

Clients connect to the **listener** of the load balancer

The load balancer connects to one or more **targets** or servers

Listener connection - one connection between the client and listener
Backend connection - one connection between load balancer and backend instance

Client is abstracted away from individual servers

Used for high availability, fault tolerance, and scaling

### Application Load Balancer (ALB)

ALB is a layer 7 - it is capable of inspecting data that passes through
it and can understand the application layer

It can take action based on things from that protocol

These are scalable and highly available.

Internet facing or Internal

Internal load balancer is used for inside a VPC only

Listens on the outside and sends to target groups

#### Cross zone load balancing

Each node that is part of the load balancer is able to distribute load
even if its not in the same AZ. It is the reason we can achieve a balanced
distribution of connections behind a load balancer.

It can also provide health checks on the target servers.

If all instances are shown as healthy, it can distribute evenly.

ALB can support a wide array of targets. An individual target can be a
member of multiple groups.

#### ALB Exam Powerup

**Targets** are lambda functions or EC2 instances that are directed towards.

Rules are path based or host based.

Support EC2, EKS, Lambda, HTTPS, HTTP/2 and websockets

ALB can use SNI for multiple SSL certs - host based rules

AWS does not suggest using Classic Load Balancer (CLB), these are legacy.

### Launch Configuration and Templates

LC and LT key concepts. They are documents which allow you to config an EC2
instance in advance. You can configure userdata and IAM role along with
networking and security groups.

Launch templates
- Provide T2/T3 Unlimited, placement groups with more.  
- Newer and recommended to use over launch configurations, they include the latest features and improvements.
- Supports versioning of templates.
- Can be used to save time when provisioning EC2 instances from the console UI / CLI.

If you need to adjust a configuration, you must make a new one and launch it.

### Auto Scaling Groups

Automatic scaling and self-healing for EC2

They make use of Launch Templates to know what to provision.

They use one version or one configuration they're assigned with.

Minimum, Desired, and Maximum Size.

Provision or Terminate instances to keep at the desired level

Scaling Policies automate based on metrics or a schedule

Auto Scaling Groups will try to keep the AZs equal with the number of EC2
instances.

#### Scaling Policies

Manual Scaling - manually adjust the desired capacity

Scheduled Scaling - time based adjustments

Dynamic Scaling

- Simple : If CPU is above 50%, add one to capacity
- Stepped : If CPU usage is above 50%, add one, if above 80% add three
- Target : Desired aggregate CPU = 40%, ASG will achieve this

Cooldown Period - How long to wait at the end of a scaling action before scaling again.

Always use cool downs to avoid rapid scaling.

AGS can use the load balancer health checks rather than EC2.

Autoscaling Groups are free  

Think about more, smaller instances to allow granularity

You should use ALB with autoscaling groups.

ASG defines when and where, Launch Template defines what.

### Network Load Balancer (NLB)

Part of AWS Version 2 series of load balances.

NLB's are Layer 4, only understand TCP and UDP.

Can't understand or interpret HTTP or HTTPs, for these reason they are much
faster in latency. You should default to NLB if http is not used.

There is nothing stopping NLB from load balancing on HTTP just by data.

Rapid scaling - **millions of requests per second**

Only member of the load balancing family that can be provided a static IP.
There is 1 interface per AZ. Can also use Elastic IPs (whitelisting)

Can do SSL pass through.

NLB can load balance non HTTP/S applications, doesn't care about anything
above TCP/UDP. This means it can handle load balancing for FTP or things
that aren't HTTP or HTTPS.

### SSL Offload and Session Stickiness

#### Bridging - Default mode

One or more clients makes one or more connections to a load balancer.
The load balancer is configured so the **listener** uses HTTPS, SSL connections
occur between the client and the load balancer.

The load balancer then needs an SSL certificate that matches the domain name
of the application.

If you need to be careful of where your certificates are stored, you may
have a problem with this system.

ELB initiates a new SSL connection to backend instances with a removed
HTTPS certificate. This can take actions based on the content of the HTTP.

It needs to decrypt any data that is being encrypted by the client.

The EC2 will need matching SSL certificates.

Needs the compute for the cryptographic operations. Every EC2 instance must
peform these cryptographic operations.

#### Pass-through

The client connects, but the load balancer passes the connection along without
decrypting the data at all. The instances still need the SSL certificates,
but the load balancer does not.

The load balancer is configured for TCP, it can see the source or destinations,
but it never touches the encrypted connection. The certificate never
needs to be seen by AWS.

Negative is you don't get any load balancing based on the HTTP part
because that is never exposed to the load balancer.

#### Offload

Clients connect to the load balancer using HTTPS and are terminated on the
load balancer. The LB needs an SSL certificate to decrypt the data, but
on the backend the data is sent via HTTP. While there is a certificate
required on the load balancer, this is not needed on the LB.

Data is in plaintext form across AWS's network. Not a problem for most.

#### Connection Stickiness

If there is no stickiness, each time the customer logs on they will have
a stateless experience. If the state is stored on a particular server,
sessions can't be load balanced across multiple servers.

Session Stickiness is an option. If enabled, the first time a user makes a
request, the load balancer generates a cookie called AWSALB. A valid duration
is between one second and seven days. For this time, sessions will be sent to
the same backend instance. This will happen until:

- If we have a server failure, then the user will be moved to a different
server.
- The cookie could expire, the whole process will repeat and will recieve a
new cookie and the process will start again.

This could cause backend unevenness
