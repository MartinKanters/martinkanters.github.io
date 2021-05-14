---
title: "Extending Kubernetes with Metacontroller"
url: extending-kubernetes-with-metacontroller
date: 2021-05-09T00:00:00+02:00 # TODO - change
draft: true
---

Kubernetes is great for many reasons. It is very good for orchestrating workload over multiple nodes in a cluster. But I think that the extensibility of Kubernetes is what makes it so powerful. This gives us endless possibilities.

# Kubernetes controllers

As said in the intro, Kubernetes is a platform for orchestrating containerized workloads across a cluster. One of the key components of the cluster is the control plane. The control plane contains [components](https://kubernetes.io/docs/concepts/overview/components/#control-plane-components) which ensures Kubernetes realizes the "desired state". The desired state is that thing that the user defines in YAML or JSON format. One example is that the user can define a Deployment of two replicas. The control plane ensures that two Pods are spun up on cluster nodes.

Kubernetes does this by employing the controller pattern. A controller is responsible for making the the current state of the cluster, the desired state. It does so by tracking at least one Kubernetes resource type. This can be a Deployment, Job or a custom type (defined by a Custom Resource Definition - CRD).

This article is about how to create custom controllers and resources. Several frameworks are built for this purpose, for example [kubebuilder](https://book.kubebuilder.io/), [KUDO](https://kudo.dev/) and [Metacontroller](https://metacontroller.github.io/metacontroller/). For this article we will use Metacontroller.

# Metacontroller

Creating a controller using tools, such as kubebuilder, can be really difficult. It requires deep knowledges of both the Kubernetes inner workings and the Golang language.
Metacontroller takes away that pain, allowing people to create controllers in a relatively simple manner. 
By implementing Metacontroller CRDs, users define which resource type should be watched, also called the parent resource. 

So, how does it work?
Once such a resource changes, an HTTP webhook is executed against a configurable URL.
The URL will typically be pointed to an HTTP endpoint created and managed by the user.
The endpoint's implementation calculates the desired state based on the data in the request and returns that in the response. 
The request contains, among other data, the (watched) parent resource.
Lastly, Metacontroller ensures that the desired state is applied to the cluster.

That is right, Metacontroller just calls an HTTP endpoint once a resource changes. 
There are no limitations as to the technology used for the webservice which contains the endpoint.
Also, there are no difficult low-level Kubernetes concepts to deal with. 

Metacontroller offers two different controller types:

- The composite controller allows developers to create and manage new children resources. Think of a Deployment managing ReplicaSets, which itself manages Pods.
- The decorator controller allows for changing the (watched) parent resource itself. Think of a controller automatically adding sidecars to Pods.

This article will show an example of the composite controller.

# Put into practice - PythonLambda Operator

explanation

// todo: image

crd
controller
webhook implementation

# Conclusion
