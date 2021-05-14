---
title: "Extending Kubernetes with Metacontroller pt. 1"
url: extending-kubernetes-with-metacontroller-part1
date: 2021-05-09T00:00:00+02:00 # TODO - change
draft: true
---

Kubernetes is great for many reasons. It is very good for orchestrating workload over multiple nodes in a cluster. But I think that the extensibility of Kubernetes is what makes it so powerful. This gives us endless possibilities.

# Kubernetes - briefly summarized

As said in the intro, Kubernetes is a platform for orchestrating containerized workloads across a cluster. One of the key components of the cluster is the control plane. The control plane contains [components](https://kubernetes.io/docs/concepts/overview/components/#control-plane-components) which ensures Kubernetes realizes the "desired state". The desired state is that thing that the user defines in YAML or JSON format. One example is that the user can define a Deployment of two replicas. The control plane ensures that two Pods are spun up on cluster nodes.

Kubernetes does this by employing the controller pattern. A controller is responsible for making the the current state of the cluster, the desired state. It does so by tracking at least one Kubernetes resource type. This can be a Deployment, Job or a custom type (defined by a Custom Resource Definition - CRD).

This article is about how to create custom controllers and resources. Several frameworks are built for this purpose, for example [kubebuilder](https://book.kubebuilder.io/), [KUDO](https://kudo.dev/) and [Metacontroller](https://metacontroller.github.io/metacontroller/). For this article we will use Metacontroller.

# Metacontroller

Creating a controller using other tools, such as kubebuilder, can be really difficult. It requires deep knowledges of both the Kubernetes inner workings and the Golang language.
A tool such as Metacontroller takes away that pain, allowing people to create controllers in a relative simple manner. 

Concepts

Composite controller vs Decorate controller

# Put into practice - PythonLambda Operator

crd
controller
webhook implementation

# Conclusion

# What's next?