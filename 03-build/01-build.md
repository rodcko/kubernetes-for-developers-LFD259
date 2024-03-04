## Container Options

There are many competing organizations working with containers. As an orchestration tool, Kubernetes is being developed to work with many of them, with the overall community moving toward open standards and easy interoperability. The early and strong presence of Docker meant that historically, this was not the focus. As Docker evolved, spreading their vendor-lock characteristics through the container creation and deployment life cycle, new projects and features have become popular. As other container engines become mature, Kubernetes continues to become more open and independent.

A container runtime is the component which runs the containerized application upon request. Docker Engine was common, though containerd and CRI-O and others are gaining community support. We will be using containerd in the labs.

The Open Container Initiative (OCI) was formed to help with flexibility and the use of any desired engine. Docker donated their libcontainer project to form a new codebase called runC to support these goals. More information about runC (https://github.com/opencontainers/runc) can be found on GitHub.

Where Docker was once the only real choice for developers, the trend toward open specifications and flexibility indicates that building with vendor-neutral features is a wise choice. Now that Docker is owned by Mirantis, many vendors have gone to other tools, such as open source options found on GitHub (https://github.com/containers).

## Container Runtime Interface (CRI)

The goal of the Container Runtime Interface (CRI) is to allow easy integration of container runtimes with kubelet. By providing a protobuf method for API, specifications and libraries, new runtimes can easily be integrated without needing deep understanding of kubelet internals.

The project is in early stage, with lots of development in action. Now that Docker-CRI integration is done, new runtimes should be easily added and swapped out. At the moment, CRI-O, rktlet and frakti are listed as work-in-progress.

## containerd

The intent of the containerd project is not to build a user-facing tool; instead, it is focused on exposing highly-decoupled low-level primitives. Because of it modularity and low overhead, large cloud providers use this engine. User facing tools such as crictl, ctr, and nerdctl are being further developed.

- ​Defaults to runC to run containers according to the OCI Specifications
- Intended to be embedded into larger systems
- Minimal CLI, focused on debugging and development.

With a focus on supporting the low-level, or backend, plumbing of containers, this project is better suited to integration and operation teams building specialized products, instead of typical build, ship, and run application.

## CRI-O

This project is currently in incubation as part of Kubernetes. It uses the Kubernetes Container Runtime Interface with OCI-compatible runtimes, thus the name CRI-O (https://github.com/cri-o/cri-o). Currently, there is support for runC (default) and Clear Containers, but a stated goal of the project is to work with any OCI-compliant runtime.

While newer than Docker or rkt, this project has gained major vendor support due to its flexibility and compatibility.

## Docker

Launched in 2013, Docker became synonymous with running containerized applications. Docker made containerizing, deploying, and consuming applications easy. As a result, it became the default option in production. With an open registry of images, Docker Hub, you can download and deploy vendor or individual-created images on multiple architectures with a single and easy to use toolset. This ease meant it was the sensible default choice for any developer as well. Issues with rapid updates and interaction with stakeholders lead to major vendors moving away from Docker soon after it became part of Mirantis.

Over the past few years, Docker has continued to grow and add features, including orchestration, which they call Swarm. These added features addressed some of the pressing needs in production, but also increased the vendor-lock and size of the product. This has lead to an increase in open tools and specifications such as CRI-O. A developer looking toward the future would be wise to work with mostly open tools for containers and Kubernetes, but he or she should understand that Docker is still the production tool of choice outside of a Red Hat environment at the moment.

## rkt

The rkt runtime, pronounced rocket, provides a CLI for running containers. Announced by CoreOS in 2014, it is now part of the Cloud Native Computing Foundation family of projects. Learning from early Docker issues, it is focused on being more secure, open and interoperable. Many of its features have been met by Docker improvements. It is not quite an easy drop-in replacement for Docker, but progress has been made. rkt uses the appc specification, and can run Docker, appc and OCI images. It deploys immutable pods.

There has been a lot of attention to the project and it was expected to be the leading replacement for Docker until CRI-O became part of the official Kubernetes Incubator.

## Containerizing an Application

To containerize an application, you begin by creating your application. Not all applications do well with containerization. The more stateless and transient the application, the better. Also, remove any environmental configuration, as those can be provided using other tools, like ConfigMaps and secrets. Develop the application until you have a single build artifact which can be deployed to multiple environments without needing to be changed, using decoupled configuration instead. Many legacy applications become a series of objects and artifacts, residing among multiple containers.

The use of Docker had been the industry standard. Large companies like Red Hat are moving to other open source tools. While currently new, one may expect them to become the new standard.

- buildah
  buildah is a tool that focuses on creating Open Container Initiative (OCI) images. It allows for creating images with and without a Dockerfile. It also does not require superuser privilege. A growing golang-based API allows for easy integration with other tools.

- podman
  podman, a "pod manager" tool, allows for the life cycle management of a container, including creating, starting, stopping and updating. You could consider this a replacement for docker. Red Hat's goal was to replace all Docker functionality with podman, using the same syntax to minimize transition issues. Some places alias docker to actually run podman.

## Rewrite Legacy Applications

Moving legacy applications to Kubernetes often brings up the question if the application should

be containerized as is, or rewritten as a transient, decoupled microservice. The cost and time of rewriting legacy applications can be high, but there is also value to leveraging the flexibility of Kubernetes.

## Creating the Dockerfile

Create a directory to hold application files. The docker build process will pull everything in the directory when it creates the image. Move the scripts and files for the containerized application into the directory.

Also, in the directory, create a Dockerfile. The name is important, as it must be those ten characters, beginning with a capital “D” newer versions allow a different filename to be used after -f <filename>. This is the expected file for the docker build to know how to create the image.

Each instruction is iterated by the Docker daemon in sequence. The instructions are not case-sensitive, but are often written in uppercase to easily distinguish from arguments. This file must have the FROM instruction first. This declares the base image for the container. Following can be a collection of ADD, RUN and a CMD to add resources and run commands to populate the image. More details about the Dockerfile reference can be found in the Docker Documentation.

Test the image by verifying that you can see it listed among other images, and use docker run <app-name> to execute it. Once you find the application performs as expected, you can push the image to a local repository in Docker Hub, after creating an account. Here is process summary:

- Create Dockerfile
- Build the container
  `sudo docker build -t simpleapp`
- Verify the image
  `sudo docker images`
  `sudo docker run simpleapp`
- Push to the repository
  `sudo docker push`

## Hosting a Local Repository

While Docker has made it easy to upload images to their Hub, these images are then public and accessible to the world. A common alternative is to create a local repository and push images there. While it can add administrative overhead, it can save downloading bandwidth, while maintaining privacy and security.

Once the local repository has been configured, you can use docker tag, then push to populate the repository with local images. Consider setting up an insecure repository until you know it works, then configuring TLS access for greater security.

## Creating a Deployment

Once you can push and pull images using the podman, crictl or docker commands, try to run a new deployment inside Kubernetes using the image. The string passed to the --image argument includes the repository to use, the name of the application, then the version.

- Use kubectl create to test the image:
  `kubectl create deployment <Deploy-Name> --image=<repo>/<app-name>:<version>`
  `kubectl create deployment time-date --image=10.110.186.162:5000/simpleapp:v2.2`

- Verify the Pod shows a running status and that all included containers are running as well:
  `​kubectl get pods`

Test that the application is performing as expected, running whichever tempest and QA testing the application has. Terminate a Pod and test that the application is as transient as designed.

## Running Commands in a Container

Part of the testing may be to execute commands within the Pod. What commands are available depend on what was included in the base environment when the image was created. In keeping to a decoupled and lean design, it's possible that there is no shell, or that the Bourne shell is available instead of bash. After testing, you may want to revisit the build and add resources necessary for testing and production.

Use the -it options for an interactive shell instead of the command running without interaction or access.

If you have more than one container, declare which container:

`kubectl exec -i​t <Pod-Name> -- /bin/bash`

## Multi-Container Pod

It may not make sense to recreate an entire image to add functionality like a shell or logging agent. Instead, you could add another container to the Pod, which would provide the necessary tools.

Each container in the Pod should be transient and decoupled. If adding another container limits the scalability or transient nature of the original application, then a new build may be warranted.

Every container in a Pod shares a single IP address and namespace. Each container has equal potential access to storage given to the Pod. Kubernetes does not provide any locking, so your configuration or application should be such that containers do not have conflicting writes.

One container could be read only while the other writes. You could configure the containers to write to different directories in the volume, or the application could have built in locking. Without these protections, there would be no way to order containers writing to the storage.

There are three terms often used for multi-container pods: ambassador, adapter, and sidecar. Based off of the documentation, you may think there is some setting for each. There is not. Each term is an expression of what a secondary pod is intended to do. All are just multi-container pods.

- Ambassador
  This type of secondary container would be used to communicate with outside resources, often outside the cluster. Using a proxy, like Envoy or other, you can embed a proxy instead of using one provided by the cluster. It is helpful if you are unsure of the cluster configuration.

- Adapter
  This type of secondary container is useful to modify the data generated by the primary container. For example, the Microsoft version of ASCII is distinct from everyone else. You may need to modify a datastream for proper use.

- Sidecar
  Similar to a sidecar on a motorcycle, it does not provide the main power, but it does help carry stuff. A sidecar is a secondary container which helps or provides a service not found in the primary application. Logging containers are a common sidecar.

## readinessProbe, livenessProbe, and startupProbe

- readinessProbe
  Oftentimes, our application may have to initialize or be configured prior to being ready to accept traffic. As we scale up our application, we may have containers in various states of creation. Rather than communicate with a client prior to being fully ready, we can use a readinessProbe. The container will not accept traffic until the probe returns a healthy state.

With the exec statement, the container is not considered ready until a command returns a zero exit code. As long as the return is non-zero, the container is considered not ready and the probe will keep trying.

Another type of probe uses an HTTP GET request (httpGet). Using a defined header to a particular port and path, the container is not considered healthy until the web server returns a code 200-399. Any other code indicates failure, and the probe will try again.

The TCP Socket probe (tcpSocket) will attempt to open a socket on a predetermined port, and keep trying based on periodSeconds. Once the port can be opened, the container is considered healthy.

- livenessProbe
  Just as we want to wait for a container to be ready for traffic, we also want to make sure it stays in a healthy state. Some applications may not have built-in checking, so we can use livenessProbes to continually check the health of a container. If the container is found to fail a probe, it is terminated. If under a controller, a replacement would be spawned.

- startupProbe
  startupProbe is useful for testing application which takes a long time to start. Some applications, especially legacy applications, might take longer to start. In such cases, regular liveness probes might start checking the health of the application before it’s fully up, leading to the killing and restarting of these containers unnecessarily. Startup probes give these applications enough time to start before regular health checks begin.

If kubelet uses a startupProbe, it will disable liveness and readiness checks until the application passes the test. The duration until the container is considered failed is failureThreshold times periodSeconds. For example, if your periodSeconds was set to five seconds, and your failureThreshold was set to ten, kubelet would check the application every five seconds until it succeeds, or is considered failed after a total of 50 seconds. If you set the periodSeconds to 60 seconds, kubelet would keep testing for 300 seconds, or five minutes, before considering the container failed.

## Testing

With the decoupled and transient nature and great flexibility, there are many possible combinations of deployments. Each deployment would have its own method for testing. No matter which technology is implemented, the goal is the end user getting what is expected. Building a test suite for your newly deployed application will help speed up the development process and limit issues with the Kubernetes integration.

In addition to overall access, building tools which ensure the distributed application functions properly, especially in a transient environment, is a good idea.

While custom-built tools may be best at testing a deployment, there are some built-in kubectl arguments to begin the process. The first one is describe and the next would be logs.

You can see the details, conditions, volumes and events for an object with describe. The Events at the end of the output can be helpful during testing, as it presents a chronological and node view of cluster actions and associated messages. A simple nginx pod may show the following output:

`$ kubectl describe pod test1`

....
Events:  
 Type Reason Age From Message

---

Normal Scheduled 18s default-scheduler Successfully assigned default/test1 to master  
 Normal Pulling 17s kubelet, master Pulling image "nginx"  
 Normal Pulled 16s kubelet, master Successfully pulled image "nginx"  
 Normal Created 16s kubelet, master Created container nginx  
 Normal Started 16s kubelet, master Started container nginx

A next step in testing may be to look at the output of containers within a pod. Not all applications will generate logs, so it could be difficult to know if the lack of output is due to error or configuration.

`$ kubectl logs test1`

A different pod may be configured to show lots of log output, such as the etcd pod:

`$ kubectl -n kube-system logs etcd-master` ##<-- Use Tab to complete for your etcd pod

....
2020-01-15 22:08:31.613174 I | mvcc: store.index: compact 446329
2020-01-15 22:08:31.627117 I | mvcc: finished scheduled compaction at 446329 (took 13.540866ms)
2020-01-15 22:13:31.622615 I | mvcc: store.index: compact 447019
2020-01-15 22:13:31.638184 I | mvcc: finished scheduled compaction at 4477019 (took 15.122934ms)
2020-01-15 22:18:31.626845 I | mvcc: store.index: compact 447709
2020-01-15 22:18:31.640592 I | mvcc: finished scheduled compaction at 447709 (took 13.309975ms)
2020-01-15 22:23:31.637814 I | mvcc: store.index: compact 448399
2020-01-15 22:23:31.651318 I | mvcc: finished scheduled compaction at 448399 (took 13.170176ms)

## Helm

Helm
In addition to pushing the image to a repository, you may want provide the image and other objects, such that they can be easily deployed. The Helm package manager is the package manager for Kubernetes.

Helm uses a chart, or collection of YAML files to deploy one or more objects. For flexibility and dynamic installation, a values.yaml file is part of a chart and is often edited before deployment.

A chart can come from many locations, with ArtifactHub becoming a centralized site to publish and find charts.
