# Overview

Welcome to the "101" section of SCOD. The goal of this section is to create a learning resource for individuals who want to learn about emerging topics in a cloud native world where containers are the focal point. The content is slightly biased towards storage.

## Mission Statement
We aim to provide a learning resource collection that is generic enough to comprehend nuances in the different solutions and paradigms. Hewlett Packard Enterprise Products are highly likely referenced in some examples and resources. We can therefore not claim vendor neutrality nor a Switzerland opinion. External resources are the primary learning assets used to frame certain topics.

Let's start the learning journey.

[TOC]

## Cloud Native Computing

The term "cloud native" stems from a software development model where resources are consumed as services. Compute, network and storage consumed through APIs, CLIs and web administration interfaces. Consumption is often modeled around paying only for what is being used.

The applications deployed into Cloud Native Computing environments are often divided into small chunks that are operated independently, referred to as microservices. On the uprising is a broader adoption of a concept called serverless where your application runs only when called and is billed in milliseconds.

Many public cloud vendors provide many already cloud native applications as services on their respective clouds. An example would be to consume a SQL database as a service rather than deploying and managing it by yourself.

### Key Attributes

These are some of the key elements of Cloud Native Computing.

- Resources are provisioned through complete self-service.
- API first strategies to promote interoperability and collaboration.
- Separation of concerns in microservice architectures.
- High degree of automation of resource provisioning and deprovisioning.
- Modern languges and frameworks.
- Infrastructure-as-a-Service (IAAS)

### Learning Resources

Curated list of learning resources for Cloud Native Computing.

- **Webinar:** [What is Cloud Native and Why Does It Exist?](https://www.cncf.io/webinars/what-is-cloud-native-and-why-does-it-exist/)<br />
  A webinar by WeaveWorks endorsed by the [CNCF](https://landscape.cncf.io/) (Cloud Native Computing Foundation).
- **Market Overview:** [CNCF Cloud Native Interactive Landscape](https://landscape.cncf.io/)<br />
  Many applications and vendors claim to be cloud native. This map is compiled by the CNCF.
- **Reference:** [12factor.net](https://12factor.net/)<br />
  A design pattern for microservice architectures.
- **Blog:** [The rise of cloud native programming languages](https://hackernoon.com/the-rise-of-cloud-native-programming-languages-211a5081f1b2)<br />
  A blog post that outlines the journey from bare-metal beyond serverless.
- **Blog:** [10 Key Attributes of Cloud-native Applications](https://thenewstack.io/10-key-attributes-of-cloud-native-applications/)<br />
  A blog post from [thenewstack.io](https://thenewstack.io)

### Practical Exercises

How to get hands-on experience of Cloud Native Computing.

- Sign-up on any of the public clouds. 
    - Provision an instance and get remote access to the host OS of the instance. 
    - Deploy an "as-a-service" of an application technology you're familiar with.
    - Connect a client from your instance to your provisioned service.
    - Deploy either web server or Layer-4 load-balancer to give external access to your client application.


## Cloud Native Tooling

Tools to interact with infrastructure and applications come in many shapes and forms. A common pattern is to learn by visually creating and deleting resources to understand an end-state. Once a pattern has been established, either APIs, 3rd party or a custom CLI is used to manage the life-cycle of the deployment in a declarative manner by manipulating RESTful APIs. Also known as Infrastructure-as-Code.

### Key Attributes

These are some of the key elements of Cloud Native Computing Tooling.

- State stored in a Source Code Control System (SCCS).
- Changes made to state are peer reviewed and automatically tested in non-production environments before being merged and deployed.
- Industry standard IT automation tools are often used to implement changes. Ansible, Puppet, Salt and Chef are example tools.
- Public clouds often provide CLIs to manage resources. These are great to prepare, inspect and test deployments with.
- Configuration and deployment files are often written in a human and machine readable format, such as JSON, YAML or TOML.

### Learning Resources

Curated list of learning resources for Cloud Native Computing Tooling.

- **Blog:** [Imperative vs Declarative](https://dev.to/stereobooster/imperative-vs-declarative-1f09)<br />
  A blog that highlights the fundamental differences between the two.
- **Reference:** [json.org](https://www.json.org/)<br />
  Definitive guide on JavaScript Object Notation (JSON) data structures.
- **Reference:** [YAML Syntax](https://docs.ansible.com/ansible/latest/reference_appendices/YAMLSyntax.html)<br />
  Simple guide for YAML Ain't Markup Language (YAML). 
- **Reference:** [RESTful API Tutorial](https://restfulapi.net/)<br />
  Learn the design principles of REpresentational State Transfer (REST).
- **Screencast:** [Super-basic Introduction to Ansible](https://www.youtube.com/watch?v=xew7CMkL7jY)<br />
  The simplest of Ansible tutorials starting with nothing.

### Practical Exercises

How to get hands-on experience of Cloud Native Computing Tooling.

- Sign-up on AWS.
    - Install the [AWS CLI](https://aws.amazon.com/cli/) and [Ansible](https://www.ansible.com/resources/get-started) in a Linux instance.
    - Configure [BOTO](https://boto3.amazonaws.com/v1/documentation/api/latest/guide/quickstart.html#configuration).
    - Use the Ansible [EC2 module](https://docs.ansible.com/ansible/latest/modules/ec2_module.html#ec2-module) to create and delete an instance.


## Cloud Native Storage

Storage for cloud computing come in many shapes and forms. Compute instances boot off block devices provided by the IaaS through the hypervisor. More devices may be attached for application data to keep host OS and application separate. Most clouds allow these devices to be snapshotted, cloned and reattached to other instances. These block devices are normally offered with different backend media, such as flash or spinning disks. Depending on the use cases and budgets parameters may be tuned to be just right.

For unstructured workloads, API driven object storage is the dominant technology due to the dramatic difference in cost and simplicity vs cloud provided block storage. An object is uploaded through an API endpoint with HTTP and automatically distributed (highly configurable) to provide high durability. The URL of the object will remain static for the duration of its lifetime. The main prohibitor for object storage adoption is that existing applications relying on POSIX filesystems need to be rewritten. 

### Key Attributes

These are some of the key elements of Cloud Native Storage.

- Provisioned and attached via APIs through IaaS if using block storage.
- Data and metadata is managed with RESTful APIs if using object. No backend to manage. Consumers use standard URLs to retrieve data.
- Highly durable with object storage. Durability equal to a local RAID device for block storage.
- Some cloud providers offer Filesystem-as-a-Service, normally standard NFS or CIFS.
- Backup and recovery of application data still needs to managed like traditional storage for block. Multi-region multi-copy persistence for object storage.

### Learning Resources

Curated list of learning resources for Cloud Native Storage.

- **Wikipedia:** [Object storage](https://en.wikipedia.org/wiki/Object_storage)<br />
  Digestible overview of Object Storage.
- **Tutorial:** [Host images on Amazon S3](https://www.channelape.com/uncategorized/host-images-amazon-s3-cheap-5-minutes/)<br />
  A five minute step-by-step guide how to host images on Amazon S3.
- **Reference:** [Amazon EBS features](https://aws.amazon.com/ebs/features/)<br />
  An overview of typical attributes for cloud provided block storage.
- **Reference:** [HPE Cloud Storage Cost Calculator](https://hpe.valuestoryapp.com/bvc/cloudvolumes)<br />
  Calculate the real costs of cloud storage based on highly dynamic data management environments.

### Practical Exercises

How to get hands-on experience of Cloud Native Storage.

- Setup a S3 compatible object storage server or use a public cloud.
    - Scality has a open source [S3 server](https://github.com/scality/cloudserver) for non-production use.
    - Configure [s3cmd](https://s3tools.org/s3cmd) to upload and retrieve files from a bucket.
- Analyze costs of 100TB of data for one year on Amazon S3 vs Azure Manage Disks.


## Containers Intro

A container is operating system-level virtualization and has been around for quite some time. By definition, the container share the kernel of the host and relies on certain abstractions to be useful. Docker the company made the technology approachable and incredibly more convenient than any predecessor. In the simplest of forms, a container image contains a virtual filesystem that contains only the dependencies the application needs. An example would be to include the Python interpreter if you wrote a program in Python. 

Containerized applications are primarily designed to run headless. In most cases these applications need to communicate with the outside world or allow inbound traffic depending on the application. Docker containers should be treated as transient, each instance starts in a known state and any data stored inside the virtual filesystem should be treated as ephemeral. This makes it extremely easy and convenient to upgrade and rollback a container. 

If data is required to persist between upgrades and rollbacks of the container, it needs to be stored outside of the container mapped from the host operating system.

The wide adoption of containers are because they're lightweight, reproducible and run everywhere. Iterations of software delivery lifecycles may be cut down to seconds from weeks with the right processes and tools.

Container images are layered per change made when the container is built. Each layer has a cryptographic hash and the layer itself can be shared between multiple containers readonly. When a new container is started from an image, the container runtime creates a COW (copy-on-write) filesystem where the particular container data is stored. This is in turn very effective as you only need one copy of a layer on the host. For example, if a bunch of applications are based off a Ubuntu base image, the base image only needs to be stored once on the host.

### Key Attributes
These are some of the key elements of Containers.

- Runs on modern architectures and operating systems. Not necessarily as a single source image. 
- Headless services (webservers, databases etc) in microservice architectures.
- Often orchestrated on compute clusters like Kubernetes, Apache Mesos Marathon or Docker Swarm.
- Software vendors often provide official and well tested container images for their applications.

### Learning Resources
Curated list of learning resources for Containers.

- **Interactive:** [Play with Docker](https://training.play-with-docker.com/)<br />
  Great interactive tutorials where you learn how to build, ship and run containers. Also has a follow-on interactive training on Kubernetes.
- **Tutorial:** [Docker for beginners](https://docker-curriculum.com/)<br />
  Comprehensive introduction to get started with Docker all the way to running it on a PaaS.
- **Cartoon:** [The Illustrated Children's Guide to Kubernetes](https://www.youtube.com/watch?v=4ht22ReBjno)<br />
  Illustrative and easy to grasp story of what Kubernetes is.
- **Bonus cartoon:** [A Kubernetes story: Phippy goes to the zoo](https://www.youtube.com/watch?v=R9-SOzep73w)<br />
  A high production quality cartoon explaining Kubernetes API objects.
- **Blog:** [How to choose the right container orchestration and how to deploy it](https://www.freecodecamp.org/news/how-to-choose-the-right-container-orchestration-and-how-to-deploy-it-41844021c241/})<br />
  A brief overview of container orchestrators.
- **Standards:** [opencontainers.org](https://www.opencontainers.org/)<br />
  Components of a container system is standards based. The Open Container Initiative is the standards body.
- **Blog/reference:** [Demystifying container runtimes](https://lwn.net/Articles/741897/)<br />
  Discusses different container runtime engines. 

### Practical Exercises
How to get hands-on experience of Containers.

- Install [Docker Desktop](https://www.docker.com/products/docker-desktop) or just Docker if using Linux.
    - Click through the [Get Started](https://docs.docker.com/get-started/) tutorial.
    - Advanced: Run any of the images built in the tutorial on a public cloud service.


## Container Tooling

Most of the tooling around containers is centered around what particular container orchestrator or development environment is being utilized. Usage of the tools differ greatly depending on the role of the user. As an operator the toolkit includes both IaaS and managing the platform to perform upgrades, user management and peripheral services such as storage and ingress load balancers. 

While many popular platforms today are based on Kubernetes, the tooling has nuances. Upstream Kubernetes uses `kubectl`, Red Hat OpenShift uses the OpenShift CLI, `oc`. With other platforms such as Rancher, nearly all management can be done through a web UI.

### Key Attributes

These are some of the key elements of Container Tooling.

- Most tools are simple, yet powerful and follow UNIX principles of doing one thing and doing it well.
- The `docker` and `kubectl` CLIs are the two most dominant for low level management.
- Workload management usually relies on external tools for simplicity, such as `docker-compose`, `kompose` and `helm`.
- Some platforms have ancillary tools to marry the IaaS with the cluster orchestrator. Such an example is `rke` for Rancher and `gkectl` for GKE On-Prem.
- The public clouds have builtin container orchestrator and container management into their native CLIs, such as `aws` and `gcloud`.
- Client side tools normally rely on environment variables and user environment configuration files that store credentials, API endpoint locations and other security aspects.

### Learning Resources

Curated list of learning resources for Container Tooling.

- **Reference:** [Use the Docker command line](https://docs.docker.com/engine/reference/commandline/cli/)<br />
  Docker CLI reference.
- **Reference:** [The kubectl Cheat Sheet](https://kubernetes.io/docs/reference/kubectl/cheatsheet/)<br />
  The kubectl cheat sheet.
- **Utility:** [kustomize.io](https://kustomize.io/)<br />
  Kubernetes native configuration managment.
- **Tutorial:** [The Ultimate Guide to Podman, Skopeo and Buildah](https://www.opensource.sa/2019/06/20/the-ultimate-guide-to-podman-skopeo-and-buildah/)<br />
  An alternative container toolchain to Docker using Podman, Buildah and Skopeo.

### Practical Exercises
How to get hands-on experience of Container Tooling.

- Install [Docker Desktop](https://www.docker.com/products/docker-desktop) or just Docker if using Linux.
    - Build a container image of an application you understand (docker build).
    - Run the container image locally (docker run).
    - Ship it to Docker Hub (docker push).
- Create an Amazon EKS cluster or equivalent.
    - Retrieve the `kubeconfig` file.
    - Run `kubectl get nodes` on your local machine.
    - Start a `Pod` using the container image built in previous exercise.


## Container Storage

Due to the ephemeral nature of a container, storage is predominantly served from the host the container is running on and is dependent on which container runtime is being used where data is stored. In the case of Docker, the overlay filesystems are under `/var/lib/docker`. If a certain path inside the container need to persist between upgrades, restarts on a different host or any other operation that will lose the locality of the data, the mount point needs to be replaced with a "bind" mount from the host.

There are also container runtime technologies that are designed to persist the entire container, effectively treating the container more like a long-lived Virtual Machine. Examples are Canonical LXD, WeaveWorks Footloose and HPE BlueData. This is particularly important for applications that rely on its projected node info to remain static throughout its entire lifecycle.

We can then begin to categorize containers into three main categories based on their lifecycle vs persistence needs.

- **Stateless Containers**<br />
  No persistence needed across restarts/upgrades/rollbacks
- **Stateful Containers**<br />
  Require certain mountpoints to persist across restarts/upgrades/rollbacks
- **Persistent Containers**<br />
  Require static node identity information across restarts/upgrades/rollbacks

Some modern Software-defined Storage solutions are offered to run alongside applications in a distributed fashion. Effectively enforcing multi-way replicas for reliability and eat into CPU and memory resources of the IaaS bill. This also introduces the dilemma of effectively locking the data into the container orchestrator and its compute nodes. Although it's convenient for developers to become self-entitled storage administrators. 

To stay in control of the data and remain mobile, storing data outside of the container orchestrator is preferable. Many container orchestrators provide plugins for external storage, some are built-in and some are supplied and supported by the storage vendor. Public clouds provide storage drivers for their IaaS storage services directly to the container orchestrator. This is widely popular pattern we're also seeing in BYO IaaS solutions such as VMware vSphere.

### Key Attributes
These are some of the key elements of Container Storage.

- Ephemeral storage needs to be fast and expandable as environments scale with more diverse applications.
- Data for stateful containers  is ideally stored outside of the container orchestrator, either the IaaS or external highly-available storage.
- Persistent containers require niche storage solution tightly coupled with the container runtime and the container orchestrator or scheduler.
- Most storage solutions provide an "access mode" often referred to as ReadWriteOnce (RWO) which only allow one Pod (in the Kubernetes case or containers from the same host access the volume. To allow multiple Pods and containers from multiple hosts, a distributed filesystem or an NFS server (widely adopted) is required to provide ReadWriteMany (RWX) access.

### Learning Resources

Curated list of learning resources for Container Storage.

- **Talk:** [Kubernetes Storage Lingo 101](https://youtu.be/uSxlgK1bCuA)<br />
  A talk that lays out the nomenclature for storage in Kubernetes in an understandable way.
- **Reference:** [Docker: Volumes](https://docs.docker.com/storage/volumes/)<br />
  Fundamental reference on how to make mount points persist for containers.
- **Reference:** [Kubernetes: Volumes](https://kubernetes.io/docs/concepts/storage/volumes/)<br />
  Using volumes in Kubernetes Pods.
- **Podcast:** [Kubernetes Storage with Saad Ali](https://softwareengineeringdaily.com/2019/06/10/kubernetes-storage-with-saad-ali/)<br />
  Essential listen understand the difference between high-availability and automatic recovery.

## Practical Exercises
How to get hands-on experience of Container Storage.

- Use Docker Desktop.
    - Replace a mount point in an interactive container with a mount point from the host
- Deploy a Amazon EKS or equivalent cluster.
    - Create a Persistent Volume Claim.
  - Run `kubectl get pv -o yaml` and match the Persistent Volume against the IaaS block volumes.

## DevOps

There are many interpretations of what DevOps "is". A bird's eye view is that there are people, processes and tools that come together to drive business outcomes through value streams. There are many core principles that could ultimately drive the outcome and no cookie cutter solution for any given organization. Breaking down problems into small pieces and creating safe systems to work in and eliminate toil are some of those principles. 

Agile development and lean manufacturing are both predecessors and role models for driving DevOps principles.

### Key Attributes

These are some of the key elements of DevOps.

- Well-defined processes, safe and proportionally sized work units for each step in a value stream.
- Autonomy through tooling backed by well-defined processes.
- Operations, development and stakeholders unified behind common goals.
- Continuous improvement, robust feedback loops and problem "swarming" of value streams.
- All work in the value stream must be visible and measurable.
- From CEO and down buy-in to prevent failure of DevOps implementations. 
- DevOps is essential to be successful when investing in a "digital transformation".

### Learning Resources

Curated list of learning resources for DevOps.

- **Opinions:** [Define DevOps: What is DevOps](http://www.itskeptic.org/content/define-devops)<br />
  Industry voices defining what DevOps is and means.
- **Blog:** [Toil: Finally a Name For a Problem We've All Felt](https://www.rundeck.com/blog/toil-finally-a-name-for-a-problem)<br />
  Broad definition of toil.
- **Hardbacks:** [DevOps Books](https://itrevolution.com/devops-books/)<br />
  Author Gene Kim has written novels and "cookbooks" of DevOps.
- **Reference:** [The DevOps Institute](https://devopsinstitute.com/)<br />
  Focuses on the human side of successfully implementing DevOps.
- **Talk:** [Bank on Open Source for DevOps Success - Capital One](https://www.youtube.com/watch?v=CTDx627FRVg)<br />
  How a disruptive company differentiate with DevOps at its core.

### Practical Exercises

How to get hands-on experience of DevOps.

- Getting practical with DevOps requires an organization and a value stream. 
    - Listen to the [The Phonenix Project](https://soundcloud.com/itrevolution/sets/the-phoenix-project-part-2) for a glimpse into how to implement DevOps.


## DevOps Tooling

The tools in DevOps are centered around the processes and value streams that support the business. Said tools also promote visibility, openness and collaboration. Inherently following security patterns, audit trails and safety. No one person should be able to misuse one tool to cause major disturbance in a value stream without quick remediation plans.

Many times CI/CD (Continuous Integration, Continuous Delivery and/or Deployment) is considered synonymous with DevOps. That is both right and wrong. If the value stream inherently contains software, yes. 

### Key Attributes

These are some of the key elements of DevOps Tooling.

- Just the right amount of privileges for a particular task.
- Issue/project tracking, kanban, source code control, CI/CD, logging and reporting are essential.
- Visibility and traceability is a key element, no work should be hidden. By person or machine.

### Learning Resources

Curated list of learning resources for DevOps Tooling.

- **Reference:** [Periodic Table of DevOps Tools](https://xebialabs.com/periodic-table-of-devops-tools/)<br />
  The most comprehensive chart of current and popular DevOps tools.
- **Blog:** [Continuous integration vs. continuous delivery vs. continuous deployment](https://www.atlassian.com/continuous-delivery/principles/continuous-integration-vs-delivery-vs-deployment)<br />
  Distinguish the components of CI/CD and what each facet encompass.
- **Reference:** [Kanban](https://www.agilealliance.org/glossary/kanban/)<br />
  Understanding Kanban helps understanding flow of work to adjust tools to work for humans, not against them.

### Practical Exercises

How to get hands-on experience of DevOps Tooling.

- Study some of the tools available to perform automated tasks on complex systems.
    - [Jenkins](https://jenkins.io)
    - [Rundeck](https://rundeck.com)
    - [Ansible Tower](https://redhat.com/ansible)
    - [Morpheus Data](https://www.morpheusdata.com/)
    - [Delphix](https://delphix.com)

The common denominator across these platforms is the observability and the ability to limit scope of controls through Role-based Access Control (RBAC). Ensuring the tasks are well-defined, automated, scoped and safe to operate.

## DevOps Storage

There aren't any particular storage paradigms (file/block/object) that are associated with DevOps. It's the implementation of the application and how it consumes storage that we vaguely may associate with DevOps. It's more of the practice that the right security controls are in place and whomever needs storage resource are fully self serviced. Human or machine.

### Key Attributes

These are some of the key elements of DevOps Storage.

- API driven through RBAC. Ensuring automation may put in place for the endpoint or person that needs access to the resource.
- Rich data management. If a value stream only needs a low performing read-only view of a certain dataset, resources supporting the value stream should only have read-only access with performance constrains.
- Agile and mobile. At will, data should be made available for a certain application or resource for its purpose. Whether it's in the public cloud, on-prem or as-a-service through safe and secure automation.

### Learning Resources

Curated list of learning resources for DevOps Storage.

- **Blog:** [Is Your Storage Too Slow for DevOps?](https://devops.com/is-your-storage-too-slow-for-devops/)<br />
  Characterization of DevOps Storage attributes.

### Practical Exercises

How to get hands-on experience of DevOps Storage.

- Familiarize yourself with a storage system's RESTful API and automation capabilities.
    - Deploy an [Ansible Tower](https://www.ansible.com/products/tower/trial) trial.
    - Write an Ansible playbook that creates a storage resource on said system.
    - Create a job in Ansible Tower with the playbook and make it available to a restricted user.

## Summary

If you have any suggestions or comments, head over to [GitHub](https://github.com/hpe-storage/scod) and file a PR or leave an issue. 
