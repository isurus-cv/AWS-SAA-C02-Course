## 09-Containers

### Intro to Containers

Virtualisation Problems
![image](https://user-images.githubusercontent.com/88237437/158794038-6d3e4540-8133-4c87-8c01-0409706cb9cb.png)

AWS EC2 Host : base
AWS Hypervisor (Nitro)

If you run a virtual machine with 4 GB ram and 40 GB disk,
the OS can comsume 60-70% of the disk and little available
memory.

If all of the OS use the same or similar resources, they are
duplicates. Every restart must manipulate the entire OS.

What you want to do is run applications in their own
isolated enviroments for their applications to run.

![image](https://user-images.githubusercontent.com/88237437/158794132-8bac25b1-0571-4741-ae3d-62fcc8aacf7b.png)

Containers have isolated OS from each other.

#### Image Anatomy

![image](https://user-images.githubusercontent.com/88237437/158794220-c919f2b3-5c6c-406a-bf6d-3e6bf7013fc5.png)

Container is a running image of docker image.

These are stacks of layers and not a monolithic disk image.

Each line of a docker image creates a new filesystem layer.

Images are created from scratch or a base image.

Images contain read only layers, images are layer onto images.

![image](https://user-images.githubusercontent.com/88237437/158794282-75120278-21fc-435d-9460-8d5ca09499d2.png)

Docker container is the same as a docker image, except it
has an additional READ/WRITE layer of the container.

If you have lots of containers with very similar base
structures, they will share the parts that overlap.

The other layers are reused between containers.

#### Container Registry

![image](https://user-images.githubusercontent.com/88237437/158794337-f334f759-8482-46af-89c1-ca98d189444d.png)

Registry or hub of container images.

Dockerfile can create a container image where it gets stored
in the container registry.

Docker hosts can run many containers between one or more images.

#### Container Key Concepts

Dockerfiles are used to build images

Portable and always run as expected. Anywhere there is a compatable host,
it will run exactly as you intended.

Containers are super lightweight, use the host OS for the
heavy lifting, but otherwise are light. File system layers
are shared when possible.

Containers only run the application and enviroment it needs.

Ports need to be **exposed** to allow outside access from
the host and beyond.

Application stacks can be multi container

### Elastic Container Service (ECS) Concepts

Accepts containers and instructions you provide.

ECS allows you to create a cluster. Clusters are where containers run from.

Container images will be located on a registry.
AWS provides ECR (elastic container registry).

Container definition tells ECS where the container image is.

Task definition represents the application as a whole and stores whatever
information is needed to run the application.

Container is just a pointer to where the container is stored and what port is
exposed. The rest is defined at the task definition.

Task definitions store the resources and networking used by the task. It stores
the compatability of how the container runs. It also stores **task role**, an
IAM role that allows the task complete its task. This is the best practice
way to give containers the access needed for other AWS resources.

A lot of tasks will only have one container definition, but a task could
include one or more containers.

Task is not by itself highly available.

ECS service - Service Definition. This defines how many copies of the task
is allowed to run to load balance. You can use a service to provide scalability
and high availability.

Tasks or services get deployed to an ECS cluster.

**Container definition** the image and the ports that will be used.

**Task definition** the task role and security is defined. This is an IAM
role that is assumed. The temporary credentials allow access to AWS products
and services.

**Task role** IAM role which the task assumes.

**Service** how many copies of a task you want to run for scaling and
high availability.

### ECS Cluster Types

ECS Cluster manages

- Scheduling and Orchestration
- Cluster manager
- Placement engine

#### EC2 mode

![image](https://user-images.githubusercontent.com/88237437/158929250-a05e8d4e-ff8e-44e7-be6c-b0fc1efefc5e.png)

ECS cluster is created within a VPC. It benefits from the multiple AZs that
are within that VPC.

You specify an initial size which will drive an **auto scaling group**

ECS using EC2 mode is not a serverless solution, you need to worry about
capacity for your cluster.

The container instances are not delivered as a managed service, they
are managed as normal EC2 instances.

This is good because you can use spot pricing or prepaid EC2 servers.

#### Fargate mode

![image](https://user-images.githubusercontent.com/88237437/158929304-a50e62a7-5fce-445b-bdcf-ee3511f9f4f3.png)

Removes more of the management overhead from ECS, no need to manage EC2.

There is a **fargate shared infrastructure** which allows all customers
to access from the same pool of resources.

Fargate deployment still uses a cluster with a VPC where AZs are specified.

For ECS tasks, they are injected into the VPC. Each task is given an
elastic network interface which has an IP address within the VPC. They then
run like an VPC resource.

You only pay for the container resources you use.

#### EC2 vs ECS(EC2) vs Fargate

If you already are using containers, use **ECS**

**EC2 mode** is good for a large workload with price conscious. This allows for
spot pricing and prepayment.

Large workload but overhead conscious **Fargate**

Small or burst style workloads **Fargate** makes sense

Batch or periodic workloads **Fargate**
