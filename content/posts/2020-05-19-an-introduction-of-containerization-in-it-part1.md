---
title: "An introduction of containerization in IT pt. 1"
url: an-introduction-of-containerization-in-it-part1
date: 2020-05-19T21:56:00+02:00
draft: false
---

Containers are relatively new in IT and are here to stay. In this article I would like to give an introduction of containerization. I hope to explain some use cases and workings by explaining the current most popular platforms in high-level.

# Why containerization?

It is not uncommon for companies to have big servers running many different applications. Typically they run many business critical services next to their databases. Every application requires certain frameworks and tooling to be installed. This makes it very hard to maintain. Servers like these require lots of resources and pose risks to the company. Some examples are:

 - **Installation**: applications might require certain or maybe multiple installations of a runtime engine (such as the Java Runtime Engine for Java applications).
 - **Runtime**: the applications are not isolated from each other, which could result in the failure of all applications when one application manages to mess something up.
 - **Scalability**: vertical scaling is not a problem, but has its limits. Horizontal scaling on the contrary is very hard, because multiple of those big servers have to be maintained, and kept in sync with each other.
 - **Uptime**: whenever the server has to undergo maintenance (e.g. for OS upgrades) all applications will be down at the same time.
Most of this can be solved by applying containerization properly. Containers are isolated units which packages the software and all its dependencies together. This ensures a quick startup and a reliable runtime on different environments.

# Docker

[Docker](https://www.docker.com/) is the default containerization tool out there. Docker images are the blueprints of a Docker container. Images are described in textfiles, called Dockerfiles. The image contains the necessary OS, applications, libraries, and commands to do a certain job. This enables an endless amount of possibilities. Some examples include running a Python web application, running a database, building Java applications, scanning source-code for security vulnerabilities, and so on.

Docker images can be pushed to Docker registries. Others can download the image, removing the need to build it again. This ensures the same runtime on all stages of a DTAP street. This means that exactly the same application and environment, which has been developed, tested and accepted, will end up on production. Even if the underlying operating systems where Docker runs on are different. "Works on my machine" no more!

![Simplified lifecycle of a Docker workflow](/images/posts/intro-containerization-docker-lifecycle.png "Simplified lifecycle of a Docker workflow")

Running one Docker container is easy, but this is often not enough for a company – especially with today’s microservices hype. Managing a cluster of containers is getting more complicated. Reliably solving complex problems by hand – like networking, scalability, security, and deployments – is very hard to do. There are several tools out there, which can be of great help in these matters. [Docker Compose](https://docs.docker.com/compose/) and [Docker Swarm](https://docs.docker.com/engine/swarm/) are bundled with Docker, but Google’s [Kubernetes](https://kubernetes.io/) takes the crown regarding popularity and feature richness.

# Conclusion

In this part we learned why containerization is exercised in IT. We then followed up with Docker, the current most popular tool for creating and managing images and containers.  
In the next part we will dive deep into Kubernetes, a tool for orchestrating multiple containers. 
[Read all about it here](/an-introduction-of-containerization-in-it-part2).