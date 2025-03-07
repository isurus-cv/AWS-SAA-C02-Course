## Security, Deployment, Operations

### AWS Secrets Manager

It does share functionality with parameter store.

This is designed specifically for secrets, passwords, API keys.

Usable via Console, CLI, API, or SDK

Supports the automatic rotation using Lambda.

It also directly integrates with RDS and other AWS products. If lambda
is invoked and changes a secret, the password can automatically
change in RDS.

This is great for rotating secrets and especially with RDS.

#### Example

The Secrets Manager SDK can be used to retrieve credentials. It can use IAM
role or access keys to retrieve the secrets.

Once the application has these secrets, it can access the database.

Periodically, the lambda function can rotate the secrets. The Lambda uses
IAM roles to do this.

The authenticaion in RDS is also rotated automatically.

Secrets are secured via KMS which ensures role seperation.

![image](https://user-images.githubusercontent.com/88237437/165134049-2bd27755-8937-4b3d-9888-31dc4ae898de.png)

**In the exam: look for keywords - Secrets, Rotation and Integration with specific Products (RDS)**

### AWS Shield and WAF (Web Application Firewall)

Provides against DDoS attacks with AWS resources. This is a denial of
service attack. Normally not possible to block them by using individual
IP addresses. Without detailed anaylsis, the traffic looks like normal
requests to your website.

Shield Standard - Free with Route53 and CloudFront as default
Provides layer 3 and layer 4 protection against DDoS attacks.

Shield advanced - $3000 per month

- Includes EC2, ELB, CloudFront, Global Acceleration and R53
- Provides access to DDoS advanced response team and financial insurance
against increased costs.

#### WAF (web application firewall)

Layer 7 firewall (HTTP/s) firewall

Protects against these complex layer 7 attacks like:

SQL injections, cross-site scripting, geo blocks, rate awareness.

Main entity provided is a WEBACL that is integrated with Load Balancers,
API gateways, and cloudfront.

The rules are added to WEBACL and evaluated when traffic arrives.

#### Example of Architecture

Shield standard automatically looks at the data before any data reaches
past Route53.

The user is directed to the closest CloudFront location. Again, shield
standard looks at the data again before it moves on.

WAF Rules are defined and included in a WEBACL which is associated to a
cloud front distribution and deployed to the edge.

Shield advanced can then intercept traffic when it reaches the load balancer.

Once the data reaches the VPC, it has been filtered at Layer 3, 4, and 7
already.

Layer 7 filtering is only provided by WAF.

![image](https://user-images.githubusercontent.com/88237437/166108133-f1f0ce4e-a301-4a68-a069-a634a4c21915.png)

### CloudHSM

An applicance which creates and managed cryptographic material such as keys.

KMS is a shared service with other AWS accounts.

Although the permissions are strict, AWS still does manage the hardware for KMS.

You can run your own HSM on premise. Cloud HSM is a true "single tenant"
hardware security module (HSM)

AWS provisioned, but it is impossible for them to help. There is no way
to recover data from them if access is lost.

Fully FIPS 140-2 Level 3 (KSM is L2 overall, but some is L3)

IF you require level 3 overall, you MUST use CloudHSM.

KSM all actions are performed with AWS CLI and IAM roles.

HSM uses

- PKCS#11
- Java Cryptography Extensions (JCE)
- Microsoft CryptoNG (CNG) libraries

KMS can use CloudHSM as a custom key store, CloudHSM integrates with KMS.

![image](https://user-images.githubusercontent.com/88237437/166145919-3c9052da-f3f7-4b2c-a53e-bc6261fae191.png)

HSM is not highly available and runs within one AZ. You need at least
two HSMs and one in each AZ you use. Once HSM is in a cluster, they
replicate all policies in sync automatically.

HSM needs an endpoint in the subnet of the VPC to allow resources access
to the cluster.

AWS has no access to the HSM appliances which store the keys.

#### Cloud HSM Use Cases

No native AWS integration with AWS products. You can't use S3 SSE with
CloudHSM.

Can be used to offload the SSL/TLS processing for the webservers. CloudHSM
is much more efficent to do these encryption processes.

Oracle Databases can use CloudHSM to enable transparent data encryption (TDE)

This can be used to protect the private keys for an issuing
certificate authority.

Anything that needs to interact with non AWS products.

### AWS Config
Auditing only, cannot prevent the actions (like a CCTV, it can record all the activities but cannot prevent any disasterus activities)
![image](https://user-images.githubusercontent.com/88237437/166145322-93eec98d-2f0d-4d44-a573-94fedac6feac.png)

![image](https://user-images.githubusercontent.com/88237437/166145304-98dd19ce-5516-42c6-98b6-517ef538177d.png)

### Amazon Macie
![image](https://user-images.githubusercontent.com/88237437/166145532-63aaa060-6c1f-4c35-a6c4-c9f55fc66cd6.png)

![image](https://user-images.githubusercontent.com/88237437/166145598-6ee092e4-ba1c-450a-be3e-2b3f5b079a9f.png)

![image](https://user-images.githubusercontent.com/88237437/166145702-94fe7e03-bf74-4a7a-9869-cfff4de4faf9.png)

![image](https://user-images.githubusercontent.com/88237437/166145865-7d80519f-1438-40b5-ba4e-539d88dfd138.png)

