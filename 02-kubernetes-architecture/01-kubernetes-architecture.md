## What Is Kubernetes?

What Is Kubernetes?
Running a container on a laptop is relatively simple. Deploying and connecting containers across multiple hosts, scaling them, deploying applications without downtime, and service discovery among several aspects can be complex.

Kubernetes addresses those challenges from the start with a set of primitives and a powerful open and extensible API. The ability to add new objects and operators allows easy customization for various production needs. The ability to add multiple schedulers and multiple API servers adds to this customization.

According to the kubernetes.io website, Kubernetes is:

"an open-source system for automating deployment, scaling, and management of containerized applications".

A key aspect of Kubernetes is that it builds on 15 years of experience at Google in a project called Borg.

Google's infrastructure started reaching high scale before virtual machines became pervasive in the datacenter, and containers provided a fine-grained solution for packing clusters efficiently. Efficiency in using clusters and managing distributed applications has been at the core of Google challenges.

Kubernetes can be an integral part of Continuous Integration/Continuous Delivery (CI/CD), as it offers many of the necessary components.

Continuous Integration
A consistent way to build and test software. Deploying new packages with code written each day, or every hour, instead of quarterly. Tools like Helm and Jenkins are often part of this with Kubernetes.
Continuous Delivery
An automated way to test and deploy software into various environments. Kubernetes handles the lifecycle of containers and connection of infrastructure resources to make rolling updates and rollbacks easy, among other deployment schemes.
There are several options and possible configurations when building a CI/CD pipeline. Tools such as Jenkins, Spinnaker, GitHub, GitLab and Helm, among others, may be part of your particular pipeline. To learn more about CI/CD take a look at the "DevOps and SRE Fundamentals: Implementing Continuous Delivery" (LFS261) course.

## Components of Kubernetes

Components of Kubernetes
Deploying containers and using Kubernetes may require a change in the development and the system administration approach to deploying applications. In a traditional environment, an application (such as a web server) would be a monolithic application placed on a dedicated server. As the web traffic increases, the application would be tuned, and perhaps moved to bigger and bigger hardware. After a couple of years, a lot of customization may have been done in order to meet the current web traffic needs.

Instead of using a large server, Kubernetes approaches the same issue by deploying a large number of small web servers, or microservices. The server and client sides of the application expect that there are one or more microservices, called replicas, available to respond to a request. It is also important that clients expect the server processes to be terminated and eventually to be replaced, leading to a transient server deployment. Instead of a large tightly-coupled Apache web server with several httpd daemons responding to page requests, there would be many nginx servers, each responding without knowledge of each other.

The transient nature of smaller services also allows for decoupling. Each aspect of the traditional application is replaced with a dedicated, but transient, microservice or agent. To join these agents, or their replacements together, we use services and API calls. A service ties traffic from one agent to another (for example, a frontend web server to a backend database) and handles new IP or other information, should either one die and be replaced.

Developers new to Kubernetes sometimes assume it is another virtual-machine manager, similar to what they have been using for decades, and continue to develop applications in the same way as prior to using Kubernetes. This is a mistake. The decoupled, transient, microservice architecture is not the same. Most legacy applications will need to be rewritten to optimally run in a cloud. In the diagram below we see the legacy deployment strategy on the left with a monolithic applications deployed to nodes. On the right, we see an example of the same functionality, on the same hardware, using multiple microservices.

Communication to, as well as internally, between components, is API call-driven, which allows for flexibility. Configuration information is stored in a JSON format, but is most often written in YAML. Kubernetes agents convert the YAML to JSON prior to persistence to the database.

Kubernetes is written in Go Language, a portable language which is like a hybridization between C++, Python, and Java. Some claim it incorporates the best (while some claim the worst) parts of each.

## Challenges

Containers have seen a huge rejuvenation in the past three years. They provide a great way to package, ship, and run applications - that is the Docker motto, now provided by many tools.

The developer experience has been boosted tremendously thanks to containers. Containers, and Docker specifically, have empowered developers with ease of building container images, simplicity of sharing images via registries, and providing a powerful user experience to manage containers. Now several tools, such as Buildah, Podman, containerd, and others allow for easy container creation and management.

However, managing containers at scale and architecting a distributed application based on microservices' principles is still challenging.

You first need a continuous integration pipeline to build your container images, test them, and verify them. Then, you need a cluster of machines acting as your base infrastructure on which to run your containers. You also need a system to launch your containers, and watch over them when things fail and self-heal. You must be able to perform rolling updates and rollbacks, and eventually tear down the resource when no longer needed.

All of these actions require flexible, scalable, and easy-to-manage network and storage.​ As containers are launched on any worker node, the network must join the resource to other containers, while still keeping the traffic secure from others. We also need a storage structure which provides and keeps or recycles storage in a seamless manner.

Would users notice if you ran Chaos Monkey, which terminates any container randomly? If so, you may have more work making containers and applications more decoupled and transient.

## The Borg Heritage

What primarily distinguishes Kubernetes from other systems is its heritage. Kubernetes is inspired by Borg - the internal system used by Google to manage its applications (e.g. Gmail, Apps, GCE).

With Google pouring the valuable lessons they learned from writing and operating Borg for over 15 years into Kubernetes, this makes Kubernetes a safe choice when having to decide on what system to use to manage containers. While a powerful tool, part of the current growth in Kubernetes is making it easier to work with and handle workloads not found in a Google data center.

Kubernetes Lineage
Chip Childers, Cloud Foundry Foundation
Retrieved from "The Platform for Forking Cloud Native Applications" (https://es.slideshare.net/chipchilders/cloud-foundry-the-platform-for-forging-cloud-native-applications) slide presentation

To learn more about the ideas behind Kubernetes, you can read the "Large-Scale Cluster Management at Google with Borg" (https://research.google/pubs/large-scale-cluster-management-at-google-with-borg/) paper.

Borg has inspired current data center systems, as well as the underlying technologies used in container runtime today. Google contributed cgroups to the Linux kernel in 2007; it limits the resources used by collection of processes. Both cgroups and Linux namespaces are at the heart of containers today, including Docker.

Mesos was inspired by discussions with Google when Borg was still a secret. Indeed, Mesos builds a multi-level scheduler, which aims to better use a data center cluster.

The Cloud Foundry Foundation embraces the twelve-factor application principles. These principles provide great guidance to build web applications that can scale easily, can be deployed in the cloud, and whose build is automated. Borg and Kubernetes address these principles as well.

## Kubernetes Architecture

To quickly demystify Kubernetes, let's have a look at the Kubernetes Architecture graphic, which shows a high-level architecture diagram of the system components.

In its simplest form, Kubernetes is made of one or more central managers (aka masters) and worker nodes (we will see in a follow-on chapter how you can actually run everything on a single node for testing purposes). The manager runs an API server, a scheduler, various operators and a datastore to keep the state of the cluster, container settings, and the networking configuration.

Kubernetes exposes an API via the API server: you can communicate with the API using a local client called kubectl or you can write your own client. The kube-scheduler sees the API requests for running a new container and finds a suitable node to run that container. Each node in the cluster runs two components: kubelet and kube-proxy. The kubelet systemd service receives spec information for container configuration, downloads and manages any necessary resources and works with the container engine on the local node to ensure the container runs or is restarted upon failure. The kube-proxy pod creates and manages local firewall rules and networking configuration to expose containers on the network.

## Terminology

We have learned that Kubernetes is an orchestration system to deploy and manage containers. Containers are not managed individually; instead, they are part of a larger object called a Pod. A Pod consists of one or more containers which share an IP address, access to storage and namespace. Typically, one container in a Pod runs an application, while other containers support the primary application.

Orchestration is managed through a series of watch-loops, also known as operators or controllers. Each operator interrogates the kube-apiserver for a particular object state, modifying the object until the declared state matches the current state. The default, newest, and feature-filled operator for containers is a Deployment. A Deployment deploys and manages a different operator called a ReplicaSet. A replicaSet is an operator which deploys multiple pods, each with the same spec information. These are called replicas. The Kubernetes architecture is made up of many operators such as Jobs and CronJobs to handle single or recurring tasks, or custom resource definitions and purpose-built operators.

There are several other API objects which can be used to deploy pods, other than ensuring a certain number of replicas is running somewhere. A DaemonSet will ensure that a single pod is deployed on every node. These are often used for logging, metrics, and security pods. A StatefulSet can be used to deploy pods in a particular order, such that following pods are only deployed if previous pods report a ready status. This is useful for legacy applications which are not cloud-friendly.

To easily manage thousands of Pods across hundreds of nodes can be difficult. To make management easier, we can use labels, arbitrary strings which become part of the object metadata. These are selectors which can then be used when checking or changing the state of objects without having to know individual names or UIDs. Nodes can have taints, an arbitrary string in the node metadata, to inform the scheduler on Pod assignments used along with toleration in Pod metadata, indicating it should be scheduled on a node with the particular taint.

There is also space in metadata for annotations, which remain with the object, but cannot be used as a selector; however, they could be leveraged by other objects or Pods.

While using lots of smaller, commodity hardware could allow every user to have their very own cluster, often multiple users and teams share access to one or more clusters. This is referred to as multi-tenancy. Some form of isolation is necessary in a multi-tenant cluster, using a combination of the following, which we introduce here but will learn more about in the future:

- namespace
  A segregation of resources, upon which resource quotas and permissions can be applied. Kubernetes objects may be created in a namespace or cluster-scoped. Users can be limited by the object verbs allowed per namespace. Also the LimitRange admission controller constrains resource usage in that namespace. Two objects cannot have the same Name: value in the same namespace.

- context
  A combination of user, cluster name and namespace. A convenient way to switch between combinations of permissions and restrictions. For example you may have a development cluster and a production cluster, or may be part of both the operations and architecture namespaces. This information is referenced from ~/.kube/config.

- Resource Limits
  A way to limit the amount of resources consumed by a pod, or to request a minimum amount of resources reserved, but not necessarily consumed, by a pod. Limits can also be set per-namespaces, which have priority over those in the PodSpec.

- Pod Security Policies
  Deprecated. Was a policy to limit the ability of pods to elevate permissions or modify the node upon which they are scheduled. This wide-ranging limitation may prevent a pod from operating properly. This was replaced with Pod Security Admission. Some have gone towards Open Policy Agent, or other tools instead.

- Pod Security Admission
  A beta feature to restrict pod behavior in an easy-to-implement and easy-to-understand manner, applied at the namespace level when a pod is created. These will leverage three profiles: Privileged, Baseline, and Restricted policies.

- Network Policies
  The ability to have an inside-the-cluster firewall. Ingress and Egress traffic can be limited according to namespaces and labels as well as typical network traffic characteristics.

## Control Plane Node

The Kubernetes master runs various server and manager processes for the cluster. Among the components of the master node are the kube-apiserver, the kube-scheduler, and the etcd database. As the software has matured, new components have been created to handle dedicated needs, such as the cloud-controller-manager; it handles tasks once handled by the kube-controller-manager to interact with other tools, such as Rancher or DigitalOcean for third-party cluster management and reporting.

There are several add-ons which have become essential to a typical production cluster, such as DNS services. Others are third-party solutions where Kubernetes has not yet developed a local component, such as cluster-level logging and resource monitoring.

## Control Plane Node Components

- kube-apiserver
  The kube-apiserver is central to the operation of the Kubernetes cluster.

All calls, both internal and external traffic, are handled via this agent. All actions are accepted and validated by this agent, and it is the only agent which connects to the etcd database. As a result, it acts as a master process for the entire cluster, and acts as a frontend of the cluster's shared state. Each API call goes through three steps: authentication, authorization, and several admission controllers.

- kube-scheduler
  The kube-scheduler uses an algorithm to determine which node will host a Pod of containers. The scheduler will try to view available resources (such as available CPU) to bind, and then assign the Pod based on availability and success. The scheduler uses pod-count by default, but complex configuration is often done if cluster-wide metrics are collected.

There are several ways you can affect the algorithm, or a custom scheduler could be used simultaneously instead. A Pod can also be assigned bind to a particular node in the pod spec, though the Pod may remain in a pending state if the node or other declared resource is unavailable.

One of the first configurations referenced during creation is if the Pod can be deployed within the current quota restrictions. If so, then the taints and tolerations, and labels of the Pods are used along with those of the nodes to determine the proper placement. Some is done as an admission controller in the kube-apiserver, the rest is done by the chosen scheduler.

You can find more details about the scheduler on GitHub.

- etcd Database
  The state of the cluster, networking, and other persistent information is kept in an etcd database, or, more accurately, a b+tree key-value store. Rather than finding and changing an entry, values are always appended to the end. Previous copies of the data are then marked for future removal by a compaction process. It works with curl and other HTTP libraries, and provides reliable watch queries.

Simultaneous requests to update a particular value all travel via the kube-apiserver, which then passes along the request to etcd in a series. The first request would update the database. The second request would no longer have the same version number as found in the object, in which case the kube-apiserver would reply with an error 409 to the requester. There is no logic past that response on the server side, meaning the client needs to expect this and act upon the denial to update.

There is a cp database along with possible followers. They communicate with each other on an ongoing basis to determine which will be master, and determine another in the event of failure. While very fast and potentially durable, there have been some hiccups with some features like whole cluster upgrades. The kubeadm cluster creation tool allows easy deployment of a multi-master cluster with stacked etcd or an external database cluster.

- Other Agents
  The kube-controller-manager is a core control loop daemon which interacts with the kube-apiserver to determine the state of the cluster. If the state does not match, the manager will contact the necessary controller to match the desired state. There are several controllers in use, such as endpoints, namespace, and replication. The full list has expanded as Kubernetes has matured.

Remaining in beta as of v1.16, the cloud-controller-manager interacts with agents outside of the cloud. It handles tasks once handled by kube-controller-manager. This allows faster changes without altering the core Kubernetes control process. Each kubelet must use the --cloud-provider-external settings passed to the binary.

## Worker Nodes

All worker nodes run the kubelet and kube-proxy, as well as the container engine, such as containerd or cri-o. Other management daemons are deployed to watch these agents or provide services not yet included with Kubernetes.

The kubelet interacts with the underlying Docker Engine also installed on all the nodes, and makes sure that the containers that need to run are actually running. The kubelet agent is the heavy lifter for changes and configuration on worker nodes. It accepts the API calls for Pod specifications (a PodSpec is a JSON or YAML file that describes a Pod). It will work to configure the local node until the specification has been met.

Should a Pod require access to storage, Secrets or ConfigMaps, the kubelet will ensure access or creation. It also sends back status to the kube-apiserver for eventual persistence.

The kube-proxy is in charge of managing the network connectivity to the containers. It does so through the use of iptables entries. It also has the userspace mode, in which it monitors Services and Endpoints using a random high-number port to proxy traffic. Use of ipvs can be enabled, with the expectation it will become the default, replacing iptables.

Kubernetes does not have cluster-wide logging yet. Instead, another CNCF project is used, called Fluentd. When implemented, it provides a unified logging layer for the cluster, which filters, buffers, and routes messages.

Cluster-wide metrics is not quite fully mature, so Prometheus is also often deployed to gather metrics from nodes and perhaps some applications.

## Pods

The whole point of Kubernetes is to orchestrate the life cycle of a container. We do not interact with particular containers. Instead, the smallest unit we can work with is a Pod. Some would say a pod of whales or peas-in-a-pod. Due to shared resources, the design of a Pod typically follows a one-process-per-container architecture.

Containers in a Pod are started in parallel by default. As a result, there is no way to determine which container becomes available first inside a Pod. initContainers can be used to ensure some containers are ready before others in a pod. To support a single process running in a container, you may need logging, a proxy, or special adapter. These tasks are often handled by other containers in the same Pod.

There is only one IP address per Pod with most network plugins. HPE Labs created a plugin which allows more than one IP per pod. As a result, if there is more than one container, they must share the IP. To communicate with each other, they can use IPC, the loopback interface, or a shared filesystem.

While Pods are often deployed with one application container in each, a common reason to have multiple containers in a Pod is for logging. You may find the term sidecar for a container dedicated to performing a helper task, like handling logs and responding to requests, as the primary application container may not have this ability.

Pods and other objects can be created in several ways. They can be created by using a generator, which. historically, has changed with each release:

`$ kubectl run newpod --image=nginx --generator=run-pod/v1`

Or, they can be created and deleted using properly formatted JSON or YAML files:

`$ kubectl create -f newpod.yaml`

`$ kubectl delete -f newpod.yaml`

Other objects will be created by operators/watch-loops to ensure the specifications and current status are the same.

## Services

With every object and agent decoupled we need a flexible and scalable operator which connects resources together and will reconnect, should something die and a replacement is spawned. Each Service is a microservice handling a particular bit of traffic, such as a single NodePort or a LoadBalancer to distribute inbound requests among many Pods.

A Service also handles access policies for inbound requests, useful for resource control, as well as for security.

A service, as well as kubectl, uses a selector in order to know which objects to connect. There are two selectors currently supported:

- equality-based
  Filters by label keys and their values. Three operators can be used, such as =, ==, and !=. If multiple values or keys are used, all must be included for a match.

- set-based
  Filters according to a set of values. The operators are in, notin, and exists. For example, the use of status notin (dev, test, maint) would select resources with the key of status which did not have a value of dev, test, nor maint.

## Operators

An important concept for orchestration is the use of operators. These are also known as watch-loops and controllers. They query the current state, compare it against the spec, and execute code based on how they differ. Various operators ship with Kubernetes, and you can create your own, as well. A simplified view of an operator is an agent, or Informer, and a downstream store. Using a DeltaFIFO queue, the source and downstream are compared. A loop process receives an obj or object, which is an array of deltas from the FIFO queue. As long as the delta is not of the type Deleted, the logic of the operator is used to create or modify some object until it matches the specification.

The Informer which uses the API server as a source requests the state of an object via an API call. The data is cached to minimize API server transactions. A similar agent is the SharedInformer; objects are often used by multiple other objects. It creates a shared cache of the state for multiple requests.

A Workqueue uses a key to hand out tasks to various workers. The standard Go workqueues of rate limiting, delayed, and time queue are typically used.

The endpoints, namespace, and serviceaccounts operators each manage the eponymous resources for Pods.

## Single IP per Pod

A Pod represents a group of co-located containers with some associated data volumes. All containers in a Pod share the same network namespace.

The diagram below shows a pod with two containers, MainApp and Logger, and two data volumes, made available under two mount points. Containers MainApp and Logger share the network namespace of a third container, known as the pause container. The pause container is used to get an IP address, then all the containers in the pod will use its network namespace. You won’t see this container from the Kubernetes perspective, but you would by running sudo docker ps. The volumes are shown for completeness and will be discussed later.

To communicate with each other, containers can use the loopback interface, write to files on a common filesystem, or via inter-process communication (IPC). As a result, co-locating applications in the same pod may have issues. There is a network plugin which will allow more than one IP address, but so far, it has only been used within HPE labs.

Support for dual-stack, IPv4 and IPv6 continues to increase with each release. For example, in a recent release kube-proxy iptables supports both stacks simultaneously.

## CNI Network Configuration File

To provide container networking, Kubernetes is standardizing on the Container Network Interface (CNI) (https://github.com/containernetworking/cni) specification. As of v1.6.0, kubeadm (the Kubernetes cluster bootstrapping tool) uses CNI as the default network interface mechanism.

CNI is an emerging specification with associated libraries to write plugins that configure container networking and remove allocated resources when the container is deleted. Its aim is to provide a common interface between the various networking solutions and container runtimes. As the CNI specification is language-agnostic, there are many plugins from Amazon ECS, to SR-IOV, to Cloud Foundry, and more.

With CNI, you can write a network configuration file:

```
{
   "cniVersion": "0.2.0",
   "name": "mynet",
   "type": "bridge",
   "bridge": "cni0",
   "isGateway": true,
   "ipMasq": true,
   "ipam": {
       "type": "host-local",
       "subnet": "10.22.0.0/16",
       "routes": [
           { "dst": "0.0.0.0/0" }
            ]
   }
}
```

This configuration defines a standard Linux bridge named cni0, which will give out IP addresses in the subnet 10.22.0.0/16. The bridge plugin will configure the network interfaces in the correct namespaces to define the container network properly. The main README of the CNI GitHub repository (https://github.com/containernetworking/cni) has more information.

## Pod-to-Pod Communication

While a CNI plugin can be used to configure the network of a pod and provide a single IP per pod, CNI does not help you with pod-to-pod communication across nodes.

The early requirement from Kubernetes was the following:

All pods can communicate with each other across nodes.
All nodes can communicate with all pods.
No Network Address Translation (NAT).
Basically, all IPs involved (nodes and pods) are routable without NAT. This can be achieved at the physical network infrastructure if you have access to it (e.g. GKE). Or, this can be achieved with a software defined overlay with solutions like:

Weave
Flannel
Calico
Cilium.
Most network plugins now support the use of Network Policies, which act as an internal firewall, limiting ingress and egress traffic.

For more information see the "Cluster Networking" Documentation page or the list of networking add-ons (https://kubernetes.io/docs/concepts/cluster-administration/addons/).

## Cloud Native Computing Foundation (CNCF)

Kubernetes is an open source software with an Apache license. Google donated Kubernetes to a newly formed collaborative project within the Linux Foundation in July 2015, when Kubernetes reached the v1.0 release. This project is known as the Cloud Native Computing Foundation (CNCF).

CNCF is not just about Kubernetes, it serves as the governing body for open source software that solves specific issues faced by cloud native applications (i.e. applications that are written specifically for a cloud environment).

CNCF has many corporate members that collaborate, such as Cisco, the Cloud Foundry Foundation, AT&T, Box, Goldman Sachs, and many others.

## Resource Recommendations

If you want to go beyond this general introduction to Kubernetes, here are a few things we recommend:

- Read the "Large-Scale Cluster Management at Google with Borg" paper. https://research.google/pubs/large-scale-cluster-management-at-google-with-borg/
- Listen to John Wilkes talking about Borg and Kubernetes. https://www.gcppodcast.com/post/episode-46-borg-and-k8s-with-john-wilkes/
- Add the Kubernetes community hangout to your calendar, and attend at least once. https://www.gcppodcast.com/post/episode-46-borg-and-k8s-with-john-wilkes/
- Join the community on Slack and go in the #kubernetes-users channel. https://communityinviter.com/apps/kubernetes/community
- Check out the Stack Overflow community https://stackoverflow.com/nocaptcha?s=80a4ba4b-765d-4d07-84f9-73ab71344544
