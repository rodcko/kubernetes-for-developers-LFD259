## Overview

Kubernetes relies on API calls and is sensitive to network issues. Standard Linux tools and processes are the best method for troubleshooting your cluster. If a shell, such as bash, is not available in an affected Pod, consider deploying another similar pod with a shell, like busybox. DNS configuration files and tools like dig are a good place to start. For more difficult challenges, you may need to install other tools, like tcpdump.

Large and diverse workloads can be difficult to track, so monitoring of usage is essential. Monitoring is about collecting key metrics, such as CPU, memory, and disk usage, and network bandwidth on your nodes, as well as monitoring key metrics in your applications. These features have not been ingested into Kubernetes, so exterior tools will be necessary, such as Prometheus (https://www.prometheus.io/). Prometheus provides a time-series database, as well as integration with Grafana for visualization and dashboards.

Logging activity across all the nodes is another feature not part of Kubernetes. Using Fluentd can be a useful data collector for a unified logging layer. Having aggregated logs can help visualize the issues, and provides the ability to search all logs. It is a good place to start when local network troubleshooting does not expose the root cause. It can be downloaded from the Fluentd website (https://www.fluentd.org/).

We are going to review some of the basic kubectl commands that you can use to debug what is happening, and we will walk you through the basic steps to be able to debug your containers, your pending containers, and also the systems in Kubernetes.

## Basic Troubleshooting Steps

The troubleshooting flow should start with the obvious. If there are errors from the command line, investigate them first. The symptoms of the issue will probably determine the next step to check. Working from the application running inside a container to the cluster as a whole may be a good idea. The application may have a shell you can use, for example:

`$ kubectl create deployment troubleshoot --image=nginx`
`$ kubectl exec -ti troubleshoot- -- /bin/sh`

If the Pod is running, use kubectl logs pod-name to view the standard out of the container. Without logs, you may consider deploying a sidecar container in the Pod to generate and handle logging. The next place to check is networking, including DNS, firewalls and general connectivity, using standard Linux commands and tools.

Security settings can also be a challenge. RBAC, covered in the security chapter, provides mandatory or discretionary access control in a granular manner. SELinux and AppArmor are also common issues, especially with network-centric applications.

The issues found with a decoupled system like Kubernetes are similar to those of a traditional datacenter, plus the added layers of Kubernetes controllers:

- Errors from the command line
- Pod logs and state of Pods
- Use shell to troubleshoot Pod DNS and network
- Check node logs for errors, make sure there are enough resources allocated
- RBAC, SELinux or AppArmor for security settings​
- API calls to and from controllers to kube-apiserver
- Inter-node network issues, DNS and firewall
- Control plane controllers (control Pods in pending or error state, errors in log files, sufficient resources, etc).

## Ongoing (Constant) Change

Unlike typical single vendor programs, there is no one organization responsible for updates to Kubernetes. As a high-velocity open source software (OSS) project, there are thousands of people working all the time. As each one of them works to solve problems or add features, it becomes possible for new problems to be caused, or features to be duplicated. Project teams and regular community meetings try to minimize issues, but they do happen.

With single vendor products, slow to change by OSS standards, one can get a list of known issues and troubleshoot to discover the particular problem they encounter. With open access, issues are typically fixed quickly, meaning that a traditional approach may not be optimal. Instead, troubleshooting is an ongoing process, where even code or a feature which worked yesterday may have an issue today. Regular revisiting of assumptions and applying an iterative flow to discover and fix issues becomes essential.

Be aware that the growth of agents follows project-based needs. When people working on cloud interaction needed to wait and integrate each change with overall agent management, a new agent was created, as well as a new project to develop it, called cloud controller manager (CCM). As a result, finding known bug information may require regular review of various websites as the project changes.

## Basic Troubleshooting Flow: Pods

Should the error not be seen while starting the Pod, investigate from within the Pod. A flow working from the application running inside a container to the larger cluster as a whole may be a good idea. Some applications will have a shell available to run commands, for example:

`$ kubectl create deployment tester --image=nginx`
`$ kubectl exec -ti tester- -- /bin/sh`

The kubectl debug feature is designed to simplify the troubleshooting and debugging process in Kubernetes clusters. It allows you to deploy a sidecar container with debugging tools into an existing pod, giving you an interactive shell for debugging purposes.

`$ kubectl debug -h`
`<output_omitted>`

`$ kubectl debug registry-6b5bb79c4-jd8fj --image=busybox`

```
error: ephemeral containers are disabled for this cluster
(error from server: "the server could not find the requested resource").
```

Now test a different feature. Create a container to understand the view a container has of the node it is running on. Change out the node name, master, for your master node. Explore the commands you have available.

`$ kubectl debug node master -it --image=busybox`

On the container:

```
/ # ps
....
31585 root   2:00 /coredns -conf /etc/coredns/Corefile
31591 root   0:21 containerd-shim -namespace moby -workdir /var/lib/containe
31633 root   0:12 /usr/bin/kube-controllers
31676 root   0:01 containerd-shim -namespace moby -workdir /var/lib/containe
31704 root   0:07 /usr/local/bin/kube-proxy --config=/var/lib/kube-proxy/con

/ # df
<output_omitted>
```

If the Pod is running, use kubectl logs pod-name to view the standard out of the container. Without logs, you may consider deploying a sidecar container in the Pod to generate and handle logging.

The next place to check is networking, including DNS, firewalls, and general connectivity, using standard Linux commands and tools.

For pods without the ability to log on their own, you may have to attach a sidecar container. These can be configured to either stream application logs to their own standard out, or run a logging agent.

Troubleshooting an application begins with the application itself. Is the application behaving as expected? Transient issues are difficult to troubleshoot; difficulties in troubleshooting are also encountered if the issue is intermittent, or if it concerns occasional slow performance.

Assuming the application is not the issue, begin by viewing the pods with the kubectl get command. Ensure the pods report a status of Running. A status of Pending typically means a resource is not available from the cluster, such as a properly tainted node, expected storage, or enough resources. Other error codes typically require looking at the logs and events of the containers for further troubleshooting. Also, look for an unusual number of restarts. A container is expected to restart due to several reasons, such as a command argument finishing. If the restarts are not due to that, it may indicate that the deployed application is having an issue and failing due to panic or probe failure.

View the details of the pod and the status of the containers using the kubectl describe pod command. This will report overall pod status, container configurations and container events. Work through each section looking for a reported error.

Should the reported information not indicate the issue, the next step would be to view any logs of the container, in case there is a misconfiguration or missing resource unknown to the cluster, but required by the application. These logs can be seen with the kubectl logs <pod-name> <container-name> command.

## Basic Troubleshooting Flow: Node and Security

Security settings can also be a challenge. RBAC, covered in the Security chapter, provides mandatory or discretionary access control in a granular manner. SELinux and AppArmor are also common issues, especially with network-centric applications.

Disabling security for testing would be a common response to an issue. Each tool has its own logs and indications of rule violation. There could be multiple issues to troubleshoot, so be sure to re-enable security and test the workload again.

Internode networking can also be an issue. Changes to switches, routes, or other network settings can adversely affect Kubernetes. Historically, the primary causes were DNS and firewalls. As Kubernetes integrations have become more common and DNS integration is maturing, this has become less of an issue. Still, check connectivity for recent infrastructure changes as part of your troubleshooting process. Every so often, an update which was said shouldn’t cause an issue may, in fact, be causing an issue.

## Basic Troubleshooting Flow: Agents

The issues found with a decoupled system like Kubernetes are similar to those of a traditional datacenter, plus the added layers of Kubernetes controllers:

- Control pods in pending or error state
- Look for errors in log files
- Are there enough resources?
- etc.

## Monitoring

Monitoring is about collecting metrics from the infrastructure, as well as applications.

Prometheus is part of the Cloud Native Computing Foundation (CNCF). As a Kubernetes plugin, it allows one to scrape resource usage metrics from Kubernetes objects across the entire cluster. It also has several client libraries which allow you to instrument your application code in order to collect application level metrics.

Collecting metrics is the first step, making use of the data collected is the next. We have the ability to expose lots of data points. Graphing the data with a tool like Grafana can allow for visual understanding of the cluster and application status. Some facilities fill walls with large TVs offering the entire company a real-time view of demand and utilization.

## Logging Tools

Logging, like monitoring, is a vast subject in IT. It has many tools that you can use as part of your arsenal.

Typically, logs are collected locally and aggregated before being ingested by a search engine and displayed via a dashboard which can use the search syntax. While there are many software stacks that you can use for logging, the Elasticsearch, Logstash, and Kibana Stack (ELK) has become quite common.

In Kubernetes, the kubelet writes container logs to local files (via the Docker logging driver). The kubectl logs command allows you to retrieve these logs.

Cluster-wide, you can use Fluentd to aggregate logs. Check the cluster administration logging concepts (https://kubernetes.io/docs/concepts/cluster-administration/logging/) for a detailed description.

Fluentd is part of the Cloud Native Computing Foundation and, together with Prometheus, they make a nice combination for monitoring and logging. You can find a detailed walk-through of running Fluentd on Kubernetes in the Kubernetes Documentation.

Setting up Fluentd for Kubernetes logging is a good exercise in understanding DaemonSets. Fluentd agents run on each node via a DaemonSet, they aggregate the logs, and feed them to an Elasticsearch instance prior to visualization in a Kibana dashboard.

## Monitoring Applications

As a distributed system, Kubernetes lacks monitoring and tracing tools which are cluster-aware. Other CNCF projects have started to handle various areas. As they are independent projects, you may find they have some overlap in capability. On the next page, we will briefly discuss Prometheus, Fluentd, OpenTracing and Jaeger.

## CNCF Projects

- Prometheus
  Prometheus (https://prometheus.io/) focuses on metrics and alerting. Provides a time-series database, queries and alerts to gather information and alerts for the entire cluster. With integration with Grafana, as well as simple graphs and console templates, you can easily visualize the cluster condition graphically.

- Fluentd
  Fluentd (https://www.fluentd.org/) is a unified logging layer which can collect logs from over 500 plugins and deliver to a multitude of outputs. A single, enterprise-wide tool to filter, buffer, and route messages just about anywhere you may want.

- OpenTracing
  While we just learned about logging and metrics, neither of the previously mentioned projects allow for a granular understanding of a transaction across a distributed architecture. OpenTracing (https://opentracing.io/) project seems to provide worthwhile instrumentation to propagate tracing among all services, code and packages, so the trace can be complete. Its goal is a “single, standard mechanism” for everything.

- Jaeger
  Jaeger (https://www.jaegertracing.io/) tracing system developed by Uber, focused on distributed context propagation, transaction monitoring, and root cause analysis, among other features. This has become a common tracing implementation of OpenTracing. You saw a Jaeger tracing example on the previous page.

## System and Agent Logs

Where system and agent files are found depends on the existence of systemd. Those with systemd will log to journalctl, which can be viewed with journalctl -a. Unless the /var/log/journal directory exists, the journal is volatile. As Kubernetes can generate a lot of logs, it would be advisable to configure log rotation, should this directory be created.

Without systemd, the logs will be created under /var/log/<agent>.log, in which case it would also be advisable to configure log rotation.

Container components:

- kube-scheduler
- kube-proxy​

Non-container components:​

- kubelet
- Docker
- etc.

## Conformance Testing

The flexibility of Kubernetes can lead to the development of a non-conforming cluster:

- Meet the demands of your environment
- Several vendor-provided tools for conformance testing
- For ease of use, Sonobuoy by Heptio can be used to understand the state of the cluster.​

## Certified Kubernetes Conformance Program

With more than 60 known distributions, there is a challenge to consistency and portability. In late 2017, a new program was started by CNCF to certify distributions that meet essential requirements and adhere to complete API functionality:

- Confidence that workloads from one distribution work on others
- Guarantees complete API functions
- Testing and Architecture Special Interest Groups.

You can read more about submitting conformance results (https://github.com/cncf/k8s-conformance/blob/master/instructions.md) on GitHub.​

## More Resources

There are several things that you can do to quickly diagnose potential issues with your application and/or cluster.

The official Documentation offers additional materials to help you get familiar with troubleshooting:

- "Troubleshooting" (https://kubernetes.io/docs/tasks/debug/)
- "Troubleshooting Applications" (https://kubernetes.io/docs/tasks/debug/debug-application/debug-pods/)
- "Troubleshoot Cluster" (https://kubernetes.io/docs/tasks/debug/debug-cluster/)
- "Debug Pods and ReplicationControllers" (https://kubernetes.io/docs/tasks/debug/debug-application/debug-pods/)
- "Debug Services" (https://kubernetes.io/docs/tasks/debug/debug-application/debug-service/)

  You can also follow:

- Kubernetes GitHub resources for issues and bug tracking (https://github.com/kubernetes/kubernetes/issues)
- Kubernetes Slack channel (https://kubernetes.slack.com/)
