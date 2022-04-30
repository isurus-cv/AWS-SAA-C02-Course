## CloudFront

### Architecture Basics

Without CloudFront
![image](https://user-images.githubusercontent.com/88237437/165739437-9b716e94-25c9-4968-bdaf-94ce06a04df8.png)

Cloudfront is a global object cache, also known as a content delivery network (CDN)

Content is cached in locations close to customers.
If the content is not available on the local cache when requested, ClouldFront
will pull and cache it then.

This provides lower latency and higher throughput for customers.

Can handle static and dynamic content.

**Origin**  the source location of your content - can be S3 Origin or Custom Origin

**Distribution** the configuration unit of CloudFront

**Edge locations** - local infrastructure which hosts a cache of your data.
There are over 200 edge locations. They can be one or more racks in a
third party server system. Cannot deploy EC2 instances in edge locations

**Regional Edge Cache** - larger version of an edge location. Provides
another layer of caching.

![image](https://user-images.githubusercontent.com/88237437/165128005-f3b3793b-1603-4386-b69c-1f27cd76e719.png)

- A user uploads an image to a S3 bucket
- The user configures the distribution and sets the S3 bucket as an origin
- The distribution is given a domain name, eg: http://asdfdsfasd.cloudfront.net. We can directly use this domain name as the url for the object. But we can assign an CNAME like *http://animalsforlife.org* as required
- The distribution will be added to the each Edge location
- When Julie request for the image, she will be directed to the closest edge location. If the image is not found in that edge location (cache miss), then it looks for the reginal edge location. If still the image is not exists in the reginal edge location, it will retrive the image from the origin, then store it in the reginal edge location, forward it to the respective edge location. The edge location will cache it and forward the image back to Julie

HTTPS can be used in CloudFront with the help of AWS Certificate Manager (ACM)

*S3 Origins do not use Reginal Edge Caches, only used by the Custom Origins. Edge locations directly communcate with S3 Origins if the searching object is not available in the Edge location*

**CloudFront is download style operations only, any uploads directly go to the Origins**

Behaviours
![image](https://user-images.githubusercontent.com/88237437/165744582-1c9e5166-32db-40ab-aa8f-74344ee47680.png)

#### Caching Optimisation

Parameters can be passed on the url such as query string parameter.
An example is `?language=en` and `?language=es`

To get the cached copy, you need to use the same query strings. If you do
use them, ensure the strings are in the same order.

### TTL and Invalidations

![image](https://user-images.githubusercontent.com/88237437/165921872-c3270e40-141c-4992-b8f2-a5812999e253.png)

- A user request for an image from the edge location
- The image is not cached in the edge location
- Edge location will request the image from the Origin
- Image is received by the edge location, it caches the image and send back to the requested user


![image](https://user-images.githubusercontent.com/88237437/165922130-81791ddf-8f6f-4b50-b285-66d5e0d630ef.png)

- Now the image has been replaced with a better one
- When a user request the image from the edge location, it returns the cached image
- The cached image is not the latest image stored at the Origin, therefore the user is getting an obsolete image

![image](https://user-images.githubusercontent.com/88237437/165922400-e15d8f4f-e66d-4a61-b144-2108d2b5181f.png)

- At some point, every cached object at the edge location will expire
- After expiary, the object is not removed until a user request for the image from the edge location

![image](https://user-images.githubusercontent.com/88237437/165922664-28e4432e-3691-44ee-910c-70ba5f36bfc8.png)

- When a new user request for the image, the edge location will not immediately return the cached image
- Instead, it forwards the request to the Origin
- The origin compares the versions of the images. If the version of the origin is same as the one in edge location, then it will return 304 Not Modified response
- Edge location will send the cached image to the user

![image](https://user-images.githubusercontent.com/88237437/165923087-e29a78d2-588a-4d03-896a-64e90801c548.png)

- When the version of at the origin is later than the one in edge location, then the origin will return the new image with 200 OK response
- The Edge location will cache the image and forward to the user

The issue here is, even after the object is replaced with a new version, the user two is getting an old object until the cache expires.

#### TTL

- More frequent cache hits ==> lower load to the origin
- Default TTL is 24hrs, which is defined in the behaviour
- We can set **Minimum TTL** and **Maximum TTL** in the behaviour
- It is possible to define TTL value for each of the object that the origin has
- If an object has not assigned a TTL value, the default TTL value is set (24hrs)
- Headers are used to set the TTL values
- ![image](https://user-images.githubusercontent.com/88237437/165924838-0a09e89b-fce7-4e6a-84bd-b9dd987be550.png)
- These header can be defined in these ways: Injected by application or the web server (for custom origins), Set in metadata (S3 origins)

#### Invalidations
- Cache invalidation is performed in a distribution
- The invalidation is applied to all edge locations used by the distribution
- This takes some time
- Invalidation will expire cached object regradless the TTL value
- We can define invalidation patterns. These act as filters to identify what are the objects (in the cache) to be invalidated
- /images/wiskers1.jpg => this will invalidate only this object
- /images/wiskers* => this will invalidate all the objects starts with "wiskers" in the images directory
- /images/* => this will invalidate all the objects inside images directory
- /* => this will invalidate all the objects in all directories
- There is a cost assoicated with the invalidation operation, that does not depend on how many objects to be invalidated
- Since this is costly, the best practise is to use versioned file names eg. wiskers1_v1.jpg, wiskers1_v2.jpg etc when updating the versions. The application must be updated with the new version of the file so that the caching TTL will not effect the users


### AWS Certificate Manager (ACM)

- HTTP lacks encryption and is insecure
- HTTPS uses SSL/TLS layer of encryption added to HTTP
- Data is encrypted in-transit
- Certificates allow servers to prove their identity
- Signed by a trusted authority (CA).
- To be secure, a website generates a certificate and has a CA sign it. The
website then uses that certificate to prove its authenticity.

Create, renew, and deploy certificates with ACM.

Supported AWS services ONLY (CloudFront and ALB, NOT EC2)

If it's not a managed service, ACM doesn't support it.

Cloudfront must have a trusted and signed certificate. It **can't be self signed**.

![image](https://user-images.githubusercontent.com/88237437/165927879-69d36ca2-d4c2-44b7-ad46-9d8406001638.png)

### CloudFront and SSL

- Each cloud front distribution receives a default domain name when it is created (CNAME). eg: https://dfasfdfasdfsdf.cloudfront.net
- We can enable the HTTPS access to the distribution by default as long as you use this address: \*.cloudfront.net
- Most of the times we have to use custom domains for the distibution, eg: cdn.catagram.io
- We can point this custom name to the CF distribution using R53
- We should add a certificate that matches the alternative domain name to the cloudfront ditribution to work this 
- In order to do this, either we have to generate or import a certificate using ACM
- For the regional services, the genarated certificates should be located in the same region as the service located
- For global services, the certificate is created or added in **US-EAST-1**
- There are some options can be set in the Behaviour settings for HTTP and HTTPS
- Allow HTTP or HTTPS, Redirect HTTP to HTTPS and HTTPS only
- There will be two SSL connections: Viewer ==> CF and CF ==> Origin
- In order to work this setup, both the certificates need to be valid public certificates  

### SNI - Server Name Identification
![image](https://user-images.githubusercontent.com/88237437/165945416-4a1899c2-bff2-4821-b701-24ae2717dc73.png)

- A web server has one single IP
- Historycally every SSL enabled website needs its own IP
- Browsers establish the connection with the web servers using TLS but TLS does not have a way to tell the server what website need to be accessed (when multiple sites are hosted in same server)
- HTTP connections can indicate which site to be accessed by providing headers in the request but HTTP connection is made after TLS connection is established
- In order to resolve this issue, an extension to the TLS is added. It is called SNI or Server Name Identification, which can be used to pass which website need to be accssed
- This is resulting in many SSL certificates are using same shared IP
- Older browsers does not support SNI, which will cost more to setup dedicated IP for each distribution in CF for older browser support

![image](https://user-images.githubusercontent.com/88237437/165946972-dfc51c4e-2a2a-4d3a-b432-89130078e5ae.png)

### Origin Types and Architecture

![image](https://user-images.githubusercontent.com/88237437/165948713-a8aa3bfa-3f02-40aa-883c-74932a90f8a0.png)

![image](https://user-images.githubusercontent.com/88237437/165948746-04762a5f-7aac-466e-b483-e0d2028f9b64.png)

![image](https://user-images.githubusercontent.com/88237437/165948782-611e952f-ece7-4c28-923f-1e2dc66e0a8e.png)
Origin or the origin group is set in the behaviour

![image](https://user-images.githubusercontent.com/88237437/165948886-c18356bb-41e1-4836-81dc-3b22c7c2d43e.png)
S3 origins
- Origin access identity can be set to restrict the access to the origin content

![image](https://user-images.githubusercontent.com/88237437/165949165-82d16454-dee5-4cfc-82e4-1ed974ea2b1d.png)
Custom origins

![image](https://user-images.githubusercontent.com/88237437/165949266-9e60d145-11d0-40a5-a891-eacce9074df9.png)
- Custom headers can be used to restrict the access


### Origin Access Identity (OAI)

This identity can be associated with a cloud front distribution.

The edge locations gain this identity.

We then remove the explicit allows and only allow the OAI to use it.

![image](https://user-images.githubusercontent.com/88237437/166098879-6befba3d-db90-4ffe-a3ad-4fc5c50769de.png)

- An OAI is created and configured in the distribution
- The distribution will share the OAI with the edge locations
- When user wants to access the origin via the CF, the edge location will send the OAI to the origin to access the objects
- The direct access to the origin is implicitly denied

Adding OAI to an origin
![image](https://user-images.githubusercontent.com/88237437/166099227-98808bef-da4a-47a4-bc6e-6367b5a66ace.png)

bucket policy
![image](https://user-images.githubusercontent.com/88237437/166099261-d142bc07-00a5-4cca-8d28-21afa2d0e1ff.png)

Best practice is to create one per distribution to manage permissions.

#### Securing Custom Origins

![image](https://user-images.githubusercontent.com/88237437/166098959-496a78f4-f359-44de-bd8e-2e82280b86fd.png)

- The communication between the customer and the edge location (Viewer protocol) is secured by HTTPS
- The communication between the edge location and the origin (Origin protocol) is also secured by HTTPS
- In order to protect the origin from accessing directly, it expect specific headers in the request to verify the request comes via an edge location

The other way is to use a firewall at the origin, and only allows the communication from the cloudfront IP range
![image](https://user-images.githubusercontent.com/88237437/166099108-3c0d0958-89a9-4ad6-b34f-ac662e9757b1.png)

### Lambda@Edge

![image](https://user-images.githubusercontent.com/88237437/166099330-60af03c7-47a2-4a0e-8c21-a328a64f3a9a.png)

![image](https://user-images.githubusercontent.com/88237437/166099423-39ca1bd4-7b4b-4bcc-ae2d-e72d733f9a0e.png)

Use cases
![image](https://user-images.githubusercontent.com/88237437/166099562-994d96df-8869-4dd7-aaf7-307fc048f2c9.png)

### AWS Global Accelerator

The problem
![image](https://user-images.githubusercontent.com/88237437/166099952-0dc5a8dd-99be-496f-9ada-424fb18fd608.png)

Starts with 2 **anycast** IP address
1.2.3.4 & 4.3.2.1
![image](https://user-images.githubusercontent.com/88237437/166099967-b92c64a2-9db1-4b38-9fb8-8f9e50e3150b.png)

Anycast IP's allow a single IP to be in multiple locations.
Routing moves traffic to closest location.

Traffic initially uses public internet and enters a global
accelerator edge location.

#### Key concepts

Move the AWS network closer to customers.

Global Accelerator moves the AWS network as close as possible.

Connections enter at edge, using anycast IPs. Transit over AWS backbone to 1+
locations.

Global Accelerator is a network product. Can use TCP, UDP.

Caching is mostly CloudFront. TCP and UDP is mostly Global Accelerator
