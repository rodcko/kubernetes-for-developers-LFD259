## Traditional Applications - Considerations

One of the larger hurdles towards implementing Kubernetes in a production environment is the suitability of application design. Optimal Kubernetes deployment design changes are more than just the simple containerization of an application. Traditional applications were built and deployed with the expectation of long-term processes and strong interdependence.

For example, an Apache web server allows for incredible customization. Often, the server would be configured and tweaked without interruption. As demand grows, the application may be migrated to larger and larger servers. The build and maintenance of the application assumes the instance would run without reset and have persistent and tightly coupled connections to other resources, such as networks and storage.

In early usage of containers, applications were containerized without redevelopment. This lead to issues after resource failure, or upon upgrade, or configuration. The cost and hassle of redesign and re-implementation should be taken into account.

## Decoupled Resources

The use of decoupled resources is integral to Kubernetes. Instead of an application using a dedicated port and socket, for the life of the instance, the goal is for each component to be decoupled from other resources. The expectation and software development toward separation allows for each component to be removed, replaced, or rebuilt.

Instead of hard-coding a resource in an application, an intermediary, such as a Service, enables connection and reconnection to other resources, providing flexibility. A single machine is no longer required to meet the application and user needs any number of systems could be brought together to meet the needs when, and, as long as, necessary.

As Kubernetes grows, even more resources are being divided out, which allows for an easier deployment of resources. Also, Kubernetes developers can optimize a particular function with fewer considerations of others objects.

## Transience

Equally important is the expectation of transience. Each object should be developed with the expectation that other components will die and be rebuilt. With any and all resources planned for transient relationships to others, we can update versions or scale usage in an easy manner.

An upgrade is perhaps not quite the correct term, as the existing application does not survive. Instead, a controller terminates the container and deploys a new one to replace it, using a different version of the application or setting. Typically, traditional applications were not written this way, opting toward long-term relationships for efficiency and ease of use.

## Flexible Framework

Like a school of fish, or pod of whales, multiple independent resources working together, but decoupled from each other and without expectation of individual permanent relationship, gain flexibility, higher availability and easy scalability. Instead of a monolithic Apache server, we can deploy a flexible number of nginx servers, each handling a small part of the workload. The goal is the same, but the framework of the solution is distinct.

A decoupled, flexible and transient application and framework is not the most efficient. In order for the Kubernetes orchestration to work, we need a series of agents, otherwise known as controllers or watch-loops, to constantly monitor the current cluster state and make changes until that state matches the declared configuration.

The commoditization of hardware has enabled the use of many smaller servers to handle a larger workload, instead of single, huge systems.

## Using Label Selectors

Labels allow for objects to be selected, which may not share other characteristics. For example, if a developer were to label their pods using their name, they could affect all of their pods, regardless of the application or deployment the pods were using.

Labels are how operators, also known as watch-loops, track and manage objects. As a result, if you were to hard-code the same label for two objects, they may be managed by different operators and cause issues. For example, one deployment may call for ten pods, while another with the same label calls for five. All the pods would be in a constant state of restarting, as each operator tries to start or stop pods until the status matches the spec.

Consider the possible ways you may want to group your pods and other objects in production. Perhaps you may use development and production labels to differentiate the state of the application. Perhaps you may want to add labels for the department, team, and primary application developer.

Selectors are namespace-scoped. Use the --all-namespaces argument to select matching objects in all namespaces.

The labels, annotations, name, and metadata of an object can be found near the top of

`$ kubectl get object-name -o  yaml `

For example:

`ckad1$ kubectl get pod examplepod-1234-vtlzd -o yaml`

```
apiVersion: v1
kind: Pod
metadata:
  annotations:
    cni.projectcalico.org/podIP: 192.168.113.220/32
  creationTimestamp: "2023-12-11T15:13:08Z"
  generateName: examplepod-1234-
  labels:
    app: examplepod
    pod-template-hash: 1234
  name: examplepod-1234-vtlzd
....
```

Using this output, one way we could select the pod would be to use the app pod label. In the command line, the colon(:) would be replaced with an equals (=) sign and the space removed. The use of -l or --selector options can be used with kubectl.

`ckad1$ kubectl -n test2 get --selector app=examplepod pod`

```
NAME                   READY  STATUS   RESTARTS  AGE
examplepod-1234-vtlzd  1/1    Running  0         25m
```

There are several built-in object labels. For example nodes have labels such as the arch, hostname, and os, which could be used for assigning pods to a particular node, or type of node.

`ckad1$ kubectl get node worker`

```
....
   creationTimestamp: "2020-05-12T14:23:04Z"
   labels:
     beta.kubernetes.io/arch: amd64
     beta.kubernetes.io/os: linux
     kubernetes.io/arch: amd64
     kubernetes.io/hostname: worker
     kubernetes.io/os: linux
   managedFields:
....
```

The nodeSelector: entry in the podspec could use this label to cause a pod to be deployed on a particular node with an entry such as:

```
     spec:
       nodeSelector:
         kubernetes.io/hostname: worker
       containers:
```

## Multi-Container Pods

The idea of multiple containers in a Pod goes against the architectural idea of decoupling as much as possible. One could run an entire operating system inside a container, but would lose much of the granular scalability Kubernetes is capable of. But there are certain needs in which a second or third co-located container makes sense. By adding a second container, each container can still be optimized and developed independently, and both can scale and be repurposed to best meet the needs of the workload.

## Sidecar, Adapter, Ambassador and initContainers

- Sidecar
  The idea for a sidecar container is to add some functionality not present in the main container. Rather than bloating code, which may not be necessary in other deployments, adding a container to handle a function such as logging solves the issue, while remaining decoupled and scalable. Prometheus monitoring and Fluentd logging leverage sidecar containers to collect data.

- Adapter
  The basic purpose of an adapter container is to modify data, either on ingress or egress, to match some other need. Perhaps, an existing enterprise-wide monitoring tools has particular data format needs. An adapter would be an efficient way to standardize the output of the main container to be ingested by the monitoring tool, without having to modify the monitor or the containerized application. An adapter container transforms multiple applications to singular view.

- Ambassador
  Ambassador is,

"an open source, Kubernetes-native API gateway for microservices built on Envoy".

It allows for access to the outside world without having to implement a service or another entry in an ingress controller: proxy local connection, reverse proxy, limits HTTP requests, re-route from the main container to the outside world.​

- initContainer
  The use of an initContainer allows one or more containers to run only if one or more previous containers run and exit successfully. For example, you could have a checksum verification scan container and a security scan container check the intended containers. Only if both containers pass the checks would the following group of containers be attempted. You can see a simple example below:

```
spec:
  containers:
  - name: intended
    image: workload
  initContainers:
  - name: scanner
    image: scanapp
```

## Custom Resource Definitions

Custom Resource Definitions
We have been working with built-in resources, or API endpoints. The flexibility of Kubernetes allows for dynamic addition of new resources as well. Once these Custom Resources (CRD) have been added, the objects can be created and accessed using standard calls and commands like kubectl. The creation of a new object stores new structured data in the etcd database and allows access via the kube-apiserver.

To make a new custom resource part of a declarative API, there needs to be a controller to retrieve the structured data continually and act to meet and maintain the declared state. This controller, or operator, is an agent to create and manage one or more instances of a specific stateful application. We have worked with built-in controllers such for Deployments and other resources.

The functions encoded into a custom operator should be all the tasks a human would need to perform if deploying the application outside of Kubernetes. The details of building a custom controller are outside the scope of this course and not included.

There are two ways to add custom resources to your Kubernetes cluster. The easiest, but less flexible, way is by adding a Custom Resource Definition to the cluster. The second, which is more flexible, is the use of Aggregated APIs, which requires a new API server to be written and added to the cluster.

As we have learned, the decoupled nature of Kubernetes depends on a collection of watcher loops, or operators, interrogating the kube-apiserver to determine if a particular configuration is true. If the current state does not match the declared state, the controller makes API calls to modify the state until they do match. If you add a new API object and operator, you can use the existing kube-apiserver to monitor and control the object. The addition of a Custom Resource Definition will be added to the cluster API path, currently under apiextensions.k8s.io/v1.

If the cluster is using Calico, you will find there are several CRDs already in use. More information on the operator framework and SDK can be found on GitHub (https://github.com/operator-framework). You can also search through existing operators which may be useful on the OperatorHub website (https://operatorhub.io/).

## Points to Ponder

Is my application as decoupled as it could possibly be?

Is there anything that I could take out, or make its own container?

These questions, while essential, often require an examination of the application within the container. Optimal use of Kubernetes is not typically found by containerization of a legacy application and deployment. Rather, most applications are rebuilt entirely, with a focus on decoupled micro-services.

When every container can survive any other containers regular termination, without the end-user noticing, you have probably decoupled the application properly.

---

Is each container transient, does it expect and have code to properly react when other containers are transient?

Could I run Chaos Monkey to kill ANY and multiple Pods, and my user would not notice?

Most Kubernetes deployments are not as transient as they should be, especially among legacy applications. The developer holds on to a previous approach and does not create the containerized application such that every connection and communication is transient and will be terminated. With code waiting for the service to connect to a replacement Pod and the containers within.

Some will note that this is not as efficient as it could be. This is correct. We are not optimizing and working against the orchestration tool. Instead, we are taking advantage of the decoupled and transient environment to scale and use only the particular microservice the end user needs.

---

Can I scale any particular component to meet workload demand?

Have I used the most open standard stable enough to meet my needs?

The minimization of an application, breaking it down into the smallest parts possible, often goes against what developers have spent an entire career doing. Each division requires more code to handle errors and communication, and is less efficient than a tightly coupled application. Machine code is very efficient for example, but not portable. Instead, code is written in a higher level language, which may not be interpreted to run as efficiently, but will run in varied environments. Perhaps approach code meant for Kubernetes in the sense that you are making the highest, and most non-specific way possible. If you have broken down each component, then you can scale only the most necessary component. This way, the actual workload is more efficient, as the only software consuming CPU cycles is that which the end user requires. There would be minimal application overhead and waste.

## Jobs

Just as we may need to redesign our applications to be decoupled, we may also consider that microservices may not need to run all the time. The use of Jobs and CronJobs can further assist with implementing decoupled and transient microservices.

Jobs are part of the batch API group. They are used to run a set number of pods to completion. If a pod fails, it will be restarted until the number of completion is reached.

While they can be seen as a way to do batch processing in Kubernetes, they can also be used to run one-off pods. A Job specification will have a parallelism and a completion key. If omitted, they will be set to one. If they are present, the parallelism number will set the number of pods that can run concurrently, and the completion number will set how many pods need to run successfully for the Job itself to be considered done. Several Job patterns can be implemented, like a traditional work queue.

Cronjobs work in a similar manner to Linux jobs, with the same time syntax. The Cronjob operator creates a Job operator at some point during the configured minute. Depending on the duration of the pods a Job runs, there are cases where a job would not be run during a time period or could run twice; as a result, the requested Pod should be idempotent.

An option spec field is .spec.concurrencyPolicy which determines how to handle existing jobs, should the time segment expire. If set to Allow, the default, another concurrent job will be run, and twice as many pods would be running. If set to Forbid, the current job continues and the new job is skipped. A value of Replace cancels the current job and the controlled pods, and starts a new job in its place.
