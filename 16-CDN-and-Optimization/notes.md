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
- The distribution is given a domain name, eg: http://asdfdsfasd.cloudfront.net. We can directly use this domain name as the url for the object. But we can assign an alias like *http://animalsforlife.org* as required
- The distribution will be added to the each Edge location
- When Julie request for the image, she will be directed to the closest edge location. If the image is not found in that edge location (cache miss), then it looks for the reginal edge location. If still the image is not exists in the reginal edge location, it will retrive the image from the origin, then store it in the reginal edge location, forward it to the respective edge location. The edge location will cache it and forward the image back to Julie

HTTPS can be used in CloudFront with the help of AWS Certificate Manager (ACM)

*S3 Origins do not use Reginal Edge Caches, only used by the Custom Origins. Edge locations directly communcate with S3 Origins if the searching object is not available in the Edge location*

**CloudFront is download style operations only, any uploads directly go to the Origins**

![Uploading image.png…]()

#### Caching Optimisation

Parameters can be passed on the url such as query string parameter.
An example is `?language=en` and `?language=es`

To get the cached copy, you need to use the same query strings. If you do
use them, ensure the strings are in the same order.

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

Cloudfront must have a trusted and signed certificate. It can't be self signed.

### Origin Access Identity (OAI)

This identity can be associated with a cloud front distribution.

The edge locations gain this identity.

We then remove the explicit allows and only allow the OAI to use it.

Best practice is to create one per distribution to manage permissions.

### AWS Global Accelerator

Starts with 2 **anycast** IP address
1.2.3.4 & 4.3.2.1

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
