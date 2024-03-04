## Overview

Security is a big and complex topic, especially in a distributed system like Kubernetes. Thus, we are just going to cover some of the concepts that deal with security in the context of Kubernetes. A developer would need to know about security to understand why their resources are not able to run in a production cluster, but may have run on their local workstation.

Then, we are going to focus on the authentication aspect of the API server and we will also cover authorization using RBAC, which is now the default configuration when you bootstrap a Kubernetes cluster with kubeadm.

We are going to look at the admission control part of the kube-apiserver, which handles and possibly modifies the API requests, asks an outside server, or deny or accept those requests.

Following that, we're going to look at a few other concepts, including how you can secure your Pods more tightly using security contexts and pod security policies, which are deprecated and will probably be replaced with Open Policy Agent. While these are often configured by the cloud admin, they limit what your deployments may be able to do, and should be understood such that you know what rights to request.

Finally, we will look at network policies. Historically, there were no limits put in inside the cluster traffic, which let any traffic flow through all of our pods, in all the different namespaces. Using network policies, we can actually define Ingress and Egress rules so that we can restrict the traffic between the different pods, namespaces, and networks. The network plugin in use, such as Flannel or Calico will determine if and how a network policy can be implemented. Multi-tenant clusters often use network policies in addition to operating system firewall configuration. Again, a developer may never need to configure the firewall, but should be able to determine if it is causing an issue with their resources, and perhaps suggest a change to the policy.

## Accessing the API

To perform any action in a Kubernetes cluster, you need to access the API and go through three main steps:

- Authentication (Certificate or Webhook)
- Authorization (RBAC or Webhook)
- Admission Controls.
  These steps are described in more detail in "Controlling Access to the Kubernetes API" (https://kubernetes.io/docs/concepts/security/controlling-access/) and illustrated by the picture below.

Retrieved from the Kubernetes website (https://kubernetes.io/images/docs/admin/access-control-overview.svg)

Once a request reaches the API server securely, it will first go through any authentication module that has been configured. The request can be rejected if authentication fails or it gets authenticated and passed to the authorization step.

At the authorization step, the request will be checked against existing policies. It will be authorized if the user has the permissions to perform the requested actions. Then, the requests will go through the last step of admission controllers. In general, admission controllers will check the actual content of the objects being created and validate them before admitting the request.

In addition to these steps, the requests reaching the API server over the network are encrypted using TLS. This needs to be properly configured using SSL certificates. If you use kubeadm, this configuration is done for you; otherwise, follow "Kubernetes the Hard Way" (https://github.com/kelseyhightower/kubernetes-the-hard-way) by Kelsey Hightower, or review the API server configuration options. (https://kubernetes.io/docs/reference/command-line-tools-reference/kube-apiserver/)

## Authentication

There are three main points to remember with authentication in Kubernetes:

- In its straightforward form, authentication is done with certificates, tokens or basic authentication (i.e. username and password).
- Users are not created by the API, but should be managed by the operating system or an external server.
- System accounts are used by processes to access the API (to learn more read "Configure Service Accounts for Pods") (https://kubernetes.io/docs/tasks/configure-pod-container/configure-service-account/).

There are two more advanced authentication mechanisms: Webhooks which can be used to verify bearer tokens, and connection with an external OpenID provider. These are becoming common in environments which already have a single sign on or other existing authentication process.

The type of authentication used is defined in the kube-apiserver startup options. Below are four examples of a subset of configuration options that would need to be set depending on what choice of authentication mechanism you choose:

`--basic-auth-file`

`--oidc-issuer-url`

`--token-auth-file`

`--authorization-webhook-config-file`

One or more Authenticator Modules are used: x509 Client Certs; static token, bearer or bootstrap token; static password file; service account and OpenID connect tokens. Each is tried until successful, and the order is not guaranteed. Anonymous access can also be enabled, otherwise you will get a 401 response. Users are not created by the API, and should be managed by an external system.

To learn more about authentication, see the official Kubernetes Documentation (https://kubernetes.io/docs/reference/access-authn-authz/authentication/).

## Authorization

Once a request is authenticated, it needs to be authorized to be able to proceed through the Kubernetes system and perform its intended action.

There are three main authorization modes and two global Deny/Allow settings. The three main modes are:

- Node
- RBAC
- Webhook.
  They can be configured as kube-apiserver startup options:

`--authorization-mode=Node,RBAC`

`--authorization-mode=Webhook`

The Node authorization is dedicated for kubelet to communicate to the kube-apiserver such that it does not need to be allowed via RBAC. All non-kubelet traffic would then be checked via RBAC.

The authorization modes implement policies to allow requests. Attributes of the requests are checked against the policies (e.g. user, group, namespace, verb).

## RBAC and the RBAC Process Overview

- RBAC
  RBAC stands for Role Based Access Control. We highly recommend that you read the documentation page linked here and take note of the following sections:

- Role example
- ClusterRole example
- RoleBinding and ClusterRoleBinding
- Referring to resources
- Aggregated ClusterRoles
- Referring to subjects
- Command line utilities.
  All resources are modeled API objects in Kubernetes, from Pods to Namespaces. They also belong to API Groups, such as core and apps. These resources allow HTTP verbs such as POST, GET, PUT, and Delete, which we have been working with so far. RBAC settings are additive, with no permission allowed unless defined

Rules are operations which can act upon an API group. Roles are one or more rules which affect, or have a scope of, a single namespace, whereas ClusterRoles have a scope of the entire cluster.

Each operation can act upon one of three subjects, which are User which relies on the operating system to manage, Group which also is managed by the operating system and ServiceAccounts which are intended to be assigned to pods and processes instead of people.

- The RBAC Process Overview
  While RBAC can be complex, the basic flow is to create a certificate for a subject, associate that to a role perhaps in a new namespace, and test. As users and groups are not API objects of Kubernetes, we are requiring outside authentication. After generating the certificate against the cluster certificate authority, we can set that credential for the subject using a context, which is a combination of user name, cluster name, authinfo, and possibly namespace. This information can be seen with the kubectl config get-contexts command.

Roles can then be used to configure an association of apiGroups, resources, and the verbs allowed to them. The user can then be bound to a role limiting what and where they can work in the cluster.

Here is a summary of the RBAC process, typically done by the cluster admin:

- Determine or create namespace for the subject
- Create certificate credentials for the subject
- Set the credentials for the user to the namespace using a context
- Create a role for the expected task set
- Bind the user to the role
- Verify the user has limited access.​

## Admission Controller

The last step in completing an API request is one or more admission controls.

Admission controllers are pieces of software that can access the content of the objects being created by the requests. They can modify the content or validate it, and potentially deny the request.

Admission controllers are needed for certain features to work properly. Controllers have been added as Kubernetes matured. Starting with the 1.13.1 release of the kube-apiserver, the admission controllers are now compiled into the binary, instead of a list passed during execution. To enable or disable, you can pass the following options, changing out the plugins you want to enable or disable:

`--enable-admission-plugins=NamespaceLifecycle,LimitRanger`
`--disable-admission-plugins=PodNodeSelector`

Controllers becoming more common are MutatingAdmissionWebhook and ValidatingAdmissionWebhook, which will allow the dynamic modification of the API request, providing greater flexibility. These calls reference an exterior service, such as OPA, and wait for a return API call. Each admission controller functionality is explained in the documentation. For example, the ResourceQuota controller will ensure that the object created does not violate any of the existing quotas. More information can be found on the Dynamic Admission Control (https://kubernetes.io/docs/reference/access-authn-authz/extensible-admission-controllers/) page in the Kubernetes documentation.

## Security Contexts

Pods and containers within pods can be given specific security constraints to limit what processes running in containers can do. For example, the UID of the process, the Linux capabilities, and the filesystem group can be limited.

Clusters installed using kubeadm allow pods any possible elevation in privilege by default. For example, a pod could control the nodes networking configuration, disable SELinux, override root, and more. These abilities are almost always limited by cluster administrators.

This security limitation is called a security context. It can be defined for the entire pod or per container, and is represented as additional sections in the resources manifests. A notable difference is that Linux capabilities are set at the container level.

For example, if you want to enforce a policy that containers cannot run their process as the root user, you can add a pod security context like the one below:

```
apiVersion: v1
kind: Pod
metadata:
  name: nginx
spec:
  securityContext:
    runAsNonRoot: true
  containers:
  - image: nginx
    name: nginx
```

Then, when you create this Pod, you will see a warning that the container is trying to run as root and that it is not allowed. Hence, the Pod will never run:

`$ kubectl get pods`

```
NAME   READY  STATUS                                                 RESTARTS  AGE
nginx  0/1    container has runAsNonRoot and image will run as root  0         10s
```

To learn more, read the "Configure a Security Context for a Pod or Container" (https://kubernetes.io/docs/tasks/configure-pod-container/security-context/) section in the Kubernetes documentation.

## Pod Security Policies (PSPs)

While deprecated, PSPs are still in common usage.

To automate the enforcement of security contexts, you can define PodSecurityPolicies (PSP) (https://kubernetes.io/docs/concepts/security/pod-security-policy/). A PSP is defined via a standard Kubernetes manifest following the PSP API schema. An example is presented below. Be aware that due to usability issues and confusion, PSPs have been deprecated. The replacement has not fully been decided. You can read more about it in the "PodSecurityPolicy Deprecation: Past, Present, and Future" (https://kubernetes.io/blog/2021/04/06/podsecuritypolicy-deprecation-past-present-and-future/) blog post.

These policies are cluster-level rules that govern what a pod can do, what they can access, what user they run as, etc.

For instance, if you do not want any of the containers in your cluster to run as the root user, you can define a PSP to that effect. You can also prevent containers from being privileged or use the host network namespace, or the host PID namespace.

While PSP has been helpful, there are other methods gaining popularity. The Open Policy Agent (OPA) (https://www.openpolicyagent.org/), often pronounced as "oh-pa", provides a unified set of tools and policy framework. This allows a single point of configuration for all of your cloud deployments.

OPA can be deployed as an admission controller inside of Kubernetes, which allows OPA to enforce or mutate requests as they are received. Using the OPA Gatekeeper it can be deployed using Custom Resource Definitions.

You can see an example of a PSP below:

```
apiVersion: policy/v1beta1
kind: PodSecurityPolicy
metadata:
  name: restricted
spec:
  seLinux:
    rule: RunAsAny
  supplementalGroups:
    rule: RunAsAny
  runAsUser:
    rule: MustRunAsNonRoot
  fsGroup:
    rule: RunAsAny
```

For Pod Security Policies to be enabled, you need to configure the admission controller of the controller-manager to contain PodSecurityPolicy. These policies make even more sense when coupled with the RBAC configuration in your cluster. This will allow you to finely tune what your users are allowed to run and what capabilities and low level privileges their containers will have.

See the PSP RBAC example (https://github.com/kubernetes/examples/blob/master/staging/podsecuritypolicy/rbac/README.md) on GitHub for more details.

## Pod Security Standards

There are three policies to limit what a pod is allowed to do. These policies are cumulative. The namespace is given the appropriate label for each policy. New pods will then be restricted. Existing pods would not be changed by an edit to the namespace. Expect the particular fields and allowed values to change as the feature matures.

Privileged - No restrictions from this policy. Details can be found in the Kubernetes documentation (https://kubernetes.io/docs/concepts/security/pod-security-standards/#privileged).

```
apiVersion: v1
kind: Namespace
metadata:
  name: no-restrictions-namespace
  labels:
    pod-security.kubernetes.io/enforce: privileged
    pod-security.kubernetes.io/enforce-version: latest
```

Baseline - Minimal restrictions. Does not allow known privilege escalations. Details can be found in the Kubernetes documentation (https://kubernetes.io/docs/concepts/security/pod-security-standards/#baseline).

```
apiVersion: v1
kind: Namespace
metadata:
  name: baseline-namespace
  labels:
    pod-security.kubernetes.io/enforce: baseline
    pod-security.kubernetes.io/enforce-version: latest
    pod-security.kubernetes.io/warn: baseline
    pod-security.kubernetes.io/warn-version: latest
```

Restricted - Most restricted policy. Follows current pod hardening best practices. Details can be found in the Kubernetes documentation (https://kubernetes.io/docs/concepts/security/pod-security-standards/#restricted).

```
apiVersion: v1
kind: Namespace
metadata:
  name: my-restricted-namespace
  labels:
    pod-security.kubernetes.io/enforce: restricted
    pod-security.kubernetes.io/enforce-version: latest
    pod-security.kubernetes.io/warn: restricted
    pod-security.kubernetes.io/warn-version: latest
```

## Network Policies

While setting up network policies is done by cluster administrators, it is important to understand how they work and could prevent your microservices from communicating with each other or outside the cluster.

By default, all pods can reach each other all ingress and egress traffic is allowed. This has been a high-level networking requirement in Kubernetes. However, network isolation can be configured and traffic to pods can be blocked. In newer versions of Kubernetes, egress traffic can also be blocked. This is done by configuring a NetworkPolicy. As all traffic is allowed, you may want to implement a policy that drops all traffic, then, other policies which allow desired ingress and egress traffic.

The spec of the policy can narrow down the effect to a particular namespace, which can be handy. Further settings include a podSelector, or label, to narrow down which Pods are affected. Further ingress and egress settings declare traffic to and from IP addresses and ports.

Not all network providers support the NetworkPolicies kind. A non-exhaustive list of providers with support includes Calico, Romana, Cilium, Kube-router, and WeaveNet.

In previous versions of Kubernetes, there was a requirement to annotate a namespace as part of network isolation, specifically the net.beta.kubernetes.io/network-policy= value. Some network plugins may still require this setting.

On the next page, you can find an example of a NetworkPolicy recipe. More network policy recipes (https://github.com/ahmetb/kubernetes-network-policy-recipes) can be found on GitHub.

## Network Policy Example

The use of policies has become stable, noted with the v1 apiVersion. The example below narrows down the policy to affect the default namespace.

Only Pods with the label of role: db will be affected by this policy, and the policy has both Ingress and Egress settings.

The ingress setting includes a 172.17 network, with a smaller range of 172.17.1.0 IPs being excluded from this traffic.

```
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: ingress-egress-policy
  namespace: default
spec:
  podSelector:
    matchLabels:
      role: db
  policyTypes:
    - Ingress
    - Egress
  ingress:
    - from:
        - ipBlock:
            cidr: 172.17.0.0/16
            except:
              - 172.17.1.0/24
        - namespaceSelector:
            matchLabels:
              project: myproject
        - podSelector:
            matchLabels:
              role: frontend
      ports:
        - protocol: TCP
          port: 6379
  egress:
    - to:
        - ipBlock:
            cidr: 10.0.0.0/24
      ports:
        - protocol: TCP
          port: 5978
```

These rules change the namespace for the following settings to be labeled project: myproject. The ingress traffic is allowed from the pods that match the label role: frontend. Finally, TCP traffic on port 6379 would be allowed from these Pods.

The egress rules have the to settings, in this case the 10.0.0.0/24 range TCP traffic to port 5978.

The use of empty ingress or egress rules denies all type of traffic for the included Pods, though this is not suggested. Use another dedicated NetworkPolicy instead.

```
podSelector:
  matchExpressions:
    - {key: inns, operator: In, values: ["yes"]}
```

## Default Policy Example

The empty braces will match all Pods not selected by other NetworkPolicy and will not allow ingress traffic. Egress traffic would be unaffected by this policy.

```
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: default-deny
spec:
  podSelector: {}
  policyTypes:
  - Ingress​
```

With the potential for complex ingress and egress rules, it may be helpful to create multiple objects which include simple isolation rules and use easy to understand names and labels.

Some network plugins, such as WeaveNet, may require annotation of the Namespace. The following shows the setting of a DefaultDeny for the myns namespace:

```
kind: Namespace
apiVersion: v1
metadata:
  name: myns
  annotations:
    net.beta.kubernetes.io/network-policy: |
     {
        "ingress": {
          "isolation": "DefaultDeny"
        }
     }
```
