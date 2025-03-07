## Serverless and AppServices

### Architecture Evolution

Youtube

Once a video is uploaded to Youtube, it creates different versions in
different quality levels. Historically the most popular way to make this work
was a monolithic architecture.

#### Monolithic

- It fails together. One error will bring the whole system down.
- Scales together. Everything expects to be running on the same compute hardware
- Bill together. All components are always running and always incurring charges.

This is the least cost effective way to architect systems.
![image](https://user-images.githubusercontent.com/88237437/159154856-4c4a4a05-6907-490b-b789-53d48a59fa16.png)


#### Tiered

The different components can be on the same server or different servers.

The components are coupled together because the endpoints connect together.

You can increase the size of the server that is running each application tier.

We can utilize load balancers in between tiers to add capacity.

The tiers are still coupled, the upload tier **expects** and requires
processing to respond. If the Processing fails completely, the upload tier
will fail because it does not recieve a response.

If there is a backload in one tier, it will impact the other tiers and the
customer experience.

Even if there is no job to be processed, the middle tier will need to be
running because otherwise it would fail.

![image](https://user-images.githubusercontent.com/88237437/159154861-bcc1ae36-2798-4f99-ad6c-d3cb4adc8b8e.png)


#### Evolving with Queues

This is a system that accepts messages off a queue. Often times
they are **FIFO** (first in, first out)

Instead of passing data into the processing tier. It will store this in an S3
bucket as well as detailing the information into the queue. This now
moves towards the next slot in the queue.

The upload tier doesn't expect an immediate answer from the processing tier.

It sends an asycn message. While this is happening, the upload can add more
messages to the queue.

The queue will have an autoscaling group to increase capacity at the end and
process appropriately.

The queue has the location of the S3 bucket as well as the location
of the information.

The autoscaling group will only bring up servers as their needed.

![image](https://user-images.githubusercontent.com/88237437/159154872-38e21e2d-28a9-44d4-930a-d8cbb29916f2.png)

#### Microservices Architecture

![image](https://user-images.githubusercontent.com/88237437/159154902-d3a72ad3-7d5f-45c3-9a93-a183bef9a762.png)

#### Event Driven Architecture

Event producers - interact with customers or systems monitoring components.
They produce events in reaction to something.

Event consumers - pieces of software waiting for events to occur.

Services can be producers and consumers at once.

In either case, there are no resources waiting around to be used.

Event router is needed for event driven architecture that also manages
an event bus.

![image](https://user-images.githubusercontent.com/88237437/159154912-672f55f5-d59b-4b5e-bc1b-253e87932741.png)

#### Highlights

- No constant running or waiting for things
- Producer generate events when something happens
- Clicks, events, errors, actions
- Events are delivered to consumers of events
- Actions are taken and the system returns to waiting

Mature event driven architecture only consumes resources while handling
events.

### AWS Lambda

- Function-as-a-service (FaaS)
- Event driven invocation (execution)
- **Lambda function** piece of code in one language
- Lambda functions use a **runtime** (e.g. Python 3.6)
- Runs in a **runtime environment**
- You are billed only for the duration a function runs. There is no charge
for having lambda functions waiting and ready to go.
- The environment has direct memory allocation. There is no direct vCPU allocation, it happens behined the scenes indirectly.
- A key part of **Serverless** architecture in AWS

#### Lambda Architecture

Best practice is to make it very small and very specialized.
The runtime enviroment will match the language the script is written in.
The runtime enviroment is lime a mini container provided with resources.

Lambda functions can be given an IAM role, **execution role**. Whenever that
function executes, the code inside has access to permissions.

This can be **event-driven** or **manual** invocation many ways.

Each time you invoke a lambda function, the enviroment is new and clean

These are by default public services and can access any websites. By default
they cannot access private VPC resources.

Once they are configured, they can only access resources within a VPC.

Should always use AWS services for input and output.

Lambda functions can run up to 15 minutes. That is the max limit.

![image](https://user-images.githubusercontent.com/88237437/159156410-abf05232-85ac-48cb-acf2-096ae9699b4e.png)

#### Key Considerations

- Currently 15 min execution limit
- Assume each execution gets a new runtime environment
- Use the execution role which is assumed when needed
- Always load data from other services from public API's or S3
- Store data to other services (e.g. S3)
- 1M free requests and 400,000 GB-seconds of compute per month

#### Common Uses

- Serverless Applications (S3, API Gateway, Lambda)
- File Processing (S3, S3 Events, Lmanda) - Image resize, waterark etc.
- Database Triggers (DynamoDB, Streams, Lambda)
- Serverless CRON (EventBridge/CWEvents + Lambda) - Schedule to run a certain script
- Realtime Stream Data Processing (Kinesis + Lambda)

#### Public Lambda and Private Lambda

Public Lambda
![image](https://user-images.githubusercontent.com/88237437/159156630-e784e1fd-0fe7-4618-a3de-bc130bf6e412.png)

Private Lambda
![image](https://user-images.githubusercontent.com/88237437/159156816-0857d0c9-be6b-4193-bae3-a0a5eab34f0e.png)

By Default the Lambda function has access to all the resources within the VPC without any restrications

In order to access public AWS services or services available in the internet
- Access the public AWS services via VPC endpoints
- Setup a NAT Gateway inside a Public subnet, connect the NATGW to an internet GW, then connect to either AWS public services or internet based services

Communication of VPC based Lambda with the services located in another VPC

Old way
![image](https://user-images.githubusercontent.com/88237437/159156926-ba105d36-f7c7-4c3f-9590-7f630a49b6ee.png)

New way
![image](https://user-images.githubusercontent.com/88237437/159156978-c7e9bb17-0d5f-47ab-a8fb-52ad23ce988b.png)

#### Security
![image](https://user-images.githubusercontent.com/88237437/159157105-d702d839-56eb-4878-ade6-fc5c43c3f9f3.png)

#### Logging
![image](https://user-images.githubusercontent.com/88237437/159157160-b10ad799-51c7-4255-abbe-f9e45835bc5c.png)

In a scenario we are diagnosing a failure of lambda execution and no trace found in the CloudWatch, main possible reason would be the CloudWatch is not given the permission (via Execution Role) to access the lamnda.

#### Invocation
1. Synchronous
2. Asynchronos
3. Event Source Mappings

Synchronous
![image](https://user-images.githubusercontent.com/88237437/159157328-6071ca5a-21ca-4202-b3a9-a07e53f22e8c.png)

Asynchronous
![image](https://user-images.githubusercontent.com/88237437/159157454-fed2ea64-4b87-4a3b-9a24-26047b9ac9ba.png)

Event Source Mapping
![image](https://user-images.githubusercontent.com/88237437/159157565-84972ec4-c646-4c8c-9ef3-cbc6c3e0402e.png)

#### Versions
![image](https://user-images.githubusercontent.com/88237437/159157609-4c5285d5-7866-456e-9e1e-1ac7f2dccb97.png)

#### Lambda Startup
![image](https://user-images.githubusercontent.com/88237437/159157746-f42ef315-40e0-405b-be22-5d0ef595a68b.png) 

### CloudWatch Events and EventBridge

Delivers near real time stream of system events that describe changes in AWS
products and services. EventsBridge will replace CW Events.

EventsBridge can also handle events from third parties. Both share the same
underlying architecture. AWS is now encouraging a migration to EB.

#### Key Concepts

They can observe if X happens at Y time(s), do Z. This is basically
CW Events V2.

Both systems have a default Event bus for a particular AWS account.

In CW Events, there is only one bus (implicit), this is not exposed.
EventBridge can have additional event buses. These can be interacted with
in the same way as the default bus.

Rules match incoming events or schedules. The rule matches an event and routes
that event to one or more targets as you define on that rule.

Architecturally at the heart of event bridge is the default event bus.

The default event bus is moving packets of JSON data.

![image](https://user-images.githubusercontent.com/88237437/159160866-52a1e3c9-aee4-49b3-8e78-b659ec1127d1.png)

#### DEMO

Lambda execution role - create and update CW logs & EC2 start/stop

![image](https://user-images.githubusercontent.com/88237437/159160971-24bcda51-ecbe-4547-b044-1b9613170767.png)

There will be 3 lambda functions created
1. Stop instances
2. Start instances
3. Start an instance when it is stopped - EventBridge rule is configured to create an event when the instance is stopped
4. Stop instance on scheduled time

### Serverless

This is not one single thing, you manage few if any servers.
Applications are a collection of small and specialized functions.

These functions are stateless and run in ephemeral environments.
Each time they run, they obtain the data they need each time.

Generally, everything is event driven. While not being used, there should
be little to no cost due to compute not being used.

Should use managed services when possible.

#### Example of Serverless

She browses to a static website that is running the uploader. The JS runs
directly from the web browser.

We use a third party auth provider, google in this case. Authenticate via
**token**.

AWS cannot use tokens provided by third parties. In this case another
service **Cognito** is called. This swaps the third party token for AWS
credentials.

It uses these temporary credentials to upload a video to S3 bucket.

The bucket will generate an event once it has completed the upload.

The event will trigger a lambda to transcode the video as needed. The
transcoder will get the original S3 bucket video location and will use
this for its workload.

These will be added to a new transcode bucket and will put an entry into
DynamoDB.

The user can then interact with another Lambda which will allow her to
pull the media from the transcode bucket using the dynamoDB entry.

![image](https://user-images.githubusercontent.com/88237437/159162204-4cb19286-bc37-4697-826a-b982bf02ddd0.png)

### Simple Notification Service (SNS)

HA, Durable, and Secure service.

This is a public service which needs access to the public endpoint. This
allows anyone from the public internet to access it.

Messages are under 256KB in size.

SNS topics are the base entity of SNS.

A publisher sends messages to a topic. Topics have subscribers which recieve
messages.

You can create topics inside the SNS.

By default all topics will recieve the message, you can put filters on
those lines to make sure they don't trigger additional lambdas.

You can use fanout to process different flows from SQS

![image](https://user-images.githubusercontent.com/88237437/159162475-e48b2dda-ccb3-478b-b1e2-5de13005101f.png)

Offers:

- Delivery Status including HTTP, Lambda, SQS
- Delivery retries - Reliable Delivery
- HA and Scalable (Regional)
- SSE (server side encryption)
- Topics can be used cross-account via Topic Policy

### AWS Step Functions

There are many problems with lambdas limitations that can be solved with
a state machine.

This is a serverless workflow

- Start
- States
- End

States are **things** which occur

Maximum duration is 1 year

Standard workflow and express. At a high level, standard is the default
and has a 1 year workflow. Express is for IOT and highly transactional
such as IoT.

Started via API Gateway, IOT Rules, EventBridge, Lambda.

Amazon States Languate (ASL) - JSON template

These use IAM Roles for permissions.

#### States

- Succeed & Fail : Will wait until either is achieved
- Wait : will wait until specific date and time or period of time
- Choice : different path is determined based on an input
- Parallel : will create parallel branches based on a choice
- Map : accepts a list of things
- Task : Single unit of work (lambda, batch, dynamoDB)

### API Gateway

Application Programming Interface (API)

This is a way that applications or services can communicate with each other.

Endpoints are used to access services. Each service has its own endpoint
in its own region.

When you request AWS stops an EC2 instance, the message is set to the API
in that region for that resource.

APIs also perform authentication using passwords or keys. API authorizes
each service and needs your permissions verified each time.

#### Authentication

#### Authorization

API gateway is an AWS managed service that provides managed API endpoints.
Allows you to create, publish, monitor, and secure APIs as a service.

Billed based on the number of API calls as well as data transfered.

This can be used for serverless architecture to provide an entry point
for that design.

This is great during an architecture evolution.

Step 1:
Create a managed API and point at the existing monolithic application.

Step 2:
Using API gateway allows the buisness to evolve along the way slowly.
This might move some of the data to fargate and aurora architecture.

Step 3:
Move to a full serverless architecture with DynamoDB


### Simple Queue Service (SQS)

Provides managed message queues, fully managed, highly available.

Replication happens within a region by default.

- Standard : 
- FIFO queue : 

Messages up to 256KB in size. These should link to larger sets of data.

Polling is checking for any messages on the queue. When a client polls
and recieves messages, they are hidden due to **visibility timeout**.

This is the amount of time that a client can wait to work on the messages.

If a client recieves messages on the queue and finishes on that workload
it can delete the message. If the client doesn't delete the message, then
it will reappear on the queue. The queue will put the message back in
and make sure a different client can retry that workload.

**Dead-letter queue** if a message is recieved multiple times but is unable
to be finished, this puts it into a different workload to try and fix
the corruption.

ASG can scale and lambdas can be invoked based on queue length.

#### Highlights

Two types of queue

Standard - multi-lane HW
guarantee the order and at least once delivery.

FIFO - single lane road with no way to overtake
guarantee the order and at exactly once delivery
3,000 messages per second with batching or up to 300 messages second without

Billed on **requests** not messages. A request is a single request to SQS
One request can send 1 - 10 messages up to 64KB total.

Requests can return 0 messages. The more frequently you poll a SQS Queue,
the less effective it is.

Two ways to poll

- short (immediate) : uses 1 request and can return 0 or more messages. If the
queue is empty, it will return 0 and try again. This hurts queues that stay
short

- long (waitTimeSeconds) : it will wait for up to 20 seconds for messages
to arrive on the queue. It will sit and wait if none currently exist.

Messages can live on SQS Queue for up to 15 days. They offer KMS encryption
at rest.

Access is based on identity policies or a queue policy.

### Kinesis

This is a scalable streaming service. It is designed to inject data from
lots of devices or lots of applications.

Producers send data into a Kinesis Stream.

The stream can scale from low to near infinite data rates.

Highly available public service by design.

Streams store a 24-hour moving window of data. Can be increased to 7 days.
Data that is 24 hours and a second more is replaced by new data entering
the stream.

Kinesis includes the store within it for the amount of data
that can be ingested during a 24 hour period. However much you ingest during
24 hours, that's included.

Multiple consumers can access data from that moving window.
One might look at data points once per hour while another looks at data in
real time.

Each shard can have 1MB/s for ingestion and 2MB/s consumption.

**Kinesis data records (1MB)** are stored accross shards and are the blocks
of data for a stream.

**Kinesis Firehose** connects to a Kinesis stream. It can move the data
from a stream onto S3 or another service.

### SQS vs Kinesis

Is this about the ingestion of data or is it about the worker pools.

Large throughput or large numbers of devices, it is likely Kinesis.

SQS has 1 thing sending messages to the queue. One consumption group from
that tier.

Allow for async communications where the sender and reciever don't care
about what the other is doing. Once the message is processed, it is deleted.

Kinesis is desiged for huge scale ingestion with multiple consumers. Rolling
window for multiple consumers.

Designed for data ingestion, analytics, monitoring, app clicks.
