---
title: "An introduction of containerization in IT pt. 2"
url: an-introduction-of-containerization-in-it-part2
date: 2020-05-19T21:57:00+02:00
draft: false
---

In the [last part](/posts/an-introduction-of-containerization-in-it-part1) we learned about many typical use cases of containers in IT.
We will continue with how to manage many of these containers together using the popular orchestrating tool, [Kubernetes](https://kubernetes.io/).

# Kubernetes

Kubernetes can be installed on one server or multiple servers in a cluster. As a client you tell the server what you want the cluster to look like, also known as the desired state. Kubernetes will try its best to make it that way, and keep it that way. An example in the context of a web shop could be the following:

 - two containers of the “users” service,
 - one container of the “inventory” service,
 - a connection between the two containers,
 - and an external-facing “users” endpoint.

 The above example can be expressed and uploaded to the server using YAML files. In these files everything can be expressed to handle the problems mentioned above. Kubernetes creates abstract layers on top of Docker containers to ensure all of these problems can be solved. These abstract layers are known as resource types. The example above is solved by requesting Kubernetes the following desired state:

 - A “Deployment” describing the “users” Docker image, including two replicas.
 - A “Deployment” describing the “inventory” Docker image.
 - A “Service” which routes an external port to the internal port of the “users” endpoint.

 Next to these resources, Kubernetes will create additional resources to make all of this happen. In case of “users” the “Deployment” creates a “ReplicaSet”, which creates two “Pods” containing the Docker containers. The complex problems mentioned above are solved out of the box by using Kubernetes.

 ![Kubernetes orchestrating multiple Docker containers](/images/posts/intro-containerization-kubernetes.png "Kubernetes orchestrating multiple Docker containers")

## Networking and service discovery
By default pods are deployed to Kubernetes’ internal network. Pods can reach each other by utilizing Kubernetes’ DNS service. A DNS entry is automatically added and assigned to the name of the Pod when it is deployed. This means that the “users” Pod can reach “inventory” just by calling “https://inventory/”, given the HTTPS port is open. There are multiple strategies for load balancing the traffic when multiple replicas of the target pod is deployed. It depends on the setup of the cluster what might be the ideal way to balance the traffic. A round-robin strategy is a straightforward strategy, but might result in a lot of latency when the cluster is distributed among several data centers worldwide.

## Scalability
A Pod in a Kubernetes cluster can be deployed with multiple instances at the same time. This can be done by setting the replicas setting in a Deployment to a value of more than one. By using Horizontal Pod Autoscalers Kubernetes can automatically scale the amount of replicas based on certain metrics (CPU by default).

## Deployments
Kubernetes offers different strategies of deploying new Pods to the cluster, for example rolling updates. To ensure that a service does not go down during a deployment, this strategy will first deploy the new version of a Pod next to the current version. Once and only when it is fully up, the old Pod will be brought down. This will be done replica by replica. Other deployment strategies allow for example A/B testing, canary testing and many more.

## Custom Resource Definitions
Kubernetes offers many different Resource types by its own which gives plenty of possibilities. Kubernetes can be extended with custom resource types, called Custom Resource Definitions. CRDs enable other types of use cases and tooling to profit from Kubernetes’ framework. Some interesting use cases are listed below:

 - Not only runtime applications can be dockerized, but also CI/CD pipelines. Tekton extends Kubernetes enabling you to write scalable and “native” build pipelines.
 - The Operator Framework allows you to deploy and create applications to your Kubernetes cluster. Typical use cases for this are databases. Databases contain state and have very specific instructions for aspects like upgrading (or downgrading) to other versions. By utilizing the Operator Framework correctly, all of these things can be automated.

## Silver bullet?
Kubernetes is the most popular distributed container platform, backed by a very active community. From a user point of view it is great. It offers everything you need when building scalable, highly available and resilient software. There are some downsides that should be pointed out:

 - The richness of features is a double-edged sword. A lot of abstraction is added before reaching the Docker container. For newcomers the learning curve is steep, even if they already have Docker knowledge. Definitely when comparing it to lesser extensive alternatives, like Docker Compose or Docker Swarm. To be fair, those are not full alternatives, as they only offer a part of what Kubernetes offers (but it might just be enough for your use case).
 - As a software engineer I do not speak from my own experience when talking about the infra side. One downside I often hear is that it is not easy to fully maintain it by yourself. This should definitely be taken into consideration before moving towards Kubernetes. It is not something for an engineer to pick up next to his main task. Luckily some big companies offer managed Kubernetes solutions, like Amazon’s EKS, Azure’s AKS, Google’s GKE or Red Hat’s (cloud independent) OpenShift. Red Hat is not into three letter acronyms, apparently.
 - Most engineers love CLI tooling, some love YAML. Be prepared to use them both. Kubernetes’ CLI tool is called kubectl. It is very powerful, but does not always give the most intuitive output. The desired state of the cluster is by default expressed in YAML. While it is designed to be easy to write and read, it can become hard when dealing with big files. Mistakes with indenting are also easily made. Then again, this varies per user and use case.

# Conclusion
Containers ensure isolated and consistent runtimes regardless of the environment. They help in making stateless applications scalable and provide easy maintenance. Docker is the most popular tool for creating images and running containers. Kubernetes extends Docker and takes care of orchestrating containers in complex compositions over multiple datacenters. This makes Kubernetes excellent in running scalable, highly available and resilient software. Its vast amount of features makes Kubernetes powerful, but comes with a price of complexity.

---

This blog first appeared [here](https://blogs.infosupport.com/an-introduction-of-containerization-in-it/).