---
title: "GitOps: Storing secrets safely in Git"
date: 2021-02-06T16:05:00+02:00
draft: false
---

Secret management in IT is a complex problem. Secrets should be stored safely, versioned and easy to use. The least amount of people should be able to manage or even see them. This article explains how to solve this problem using encryption in a GitOps fashion.

### External secrets vault
Let’s look at two techniques often used for secret management. Most of the time they are stored in external services, such as [Hashicorp Vault](https://www.vaultproject.io/docs/platform/k8s/injector). This is a product that is installed and running somewhere your application can access. It usually provides very refined access control mechanisms, like [RBAC](https://en.wikipedia.org/wiki/Role-based_access_control). Applications can read these secrets on runtime and use them any way they like. Some of these services even offer versioning capabilities of these secrets.

### Kubernetes Secrets
In case your application is running on Kubernetes, there is another way to externalize secrets. Kubernetes offers a specific resource type called a [Secret](https://kubernetes.io/docs/concepts/configuration/secret/). This is a key-value map, where the value is base64 encoded (read: not encrypted). A Kubernetes pod can then read the value from this secret into an environment variable or mount it onto the filesystem. RBAC rules can protect the Secrets from being read by unwanted eyes.

## Problems
There are some issues with both type of solutions:

 - **Versioning**: Both Kubernetes Secrets and external vaults have some support of versioning, but often work very different from version control systems that are already used in the company. Developers need to understand both ways of versioning and make it work together somehow.
 - **Independence**: Your application is not one contained package of deployment once external secrets are referenced from within. If the secret your application uses has been deleted in your Vault, the package that worked yesterday, doesn’t work anymore today.
 - **Process**: Application development and secret management are two separate processes often done by separate parties. A developer is not always able to manage secrets himself, so it’s possible that he deploys the application to production, while the required secret is not available on production yet.

# Introducing GitOps
Because of the reasons listed above and other things, it is becoming more and more common to store everything in Git. First only application code was stored in Git, but it was later followed by database evolution scripts and even CI pipelines. [GitOps](https://www.weave.works/technologies/gitops/) is an upcoming way of storing even the desired state of your application architecture and infrastructure in Git. Imagine a Git repository including all Kubernetes (and for example Terraform) definitions. You get all the benefits of a great source control system and you can easily attach a pipeline to it which automatically deploys your desired state to Kubernetes. Or even better, you can use a tool, like [Argo CD](https://blogs.infosupport.com/argo-cd/), for that!

Sounds nice, but as most people know, secrets are one of the things that should never be stored in Git. Once stored, everyone who can access the repository can see them, so they are compromised and will forever stay in the history (unless you dare to rewrite the history – nineteen eighty-four anyone?).

### Encryption
One way to overcome this problem is to encrypt the secrets before pushing them to your Git repository. By using asymmetric encryption every developer can encrypt secrets using a (public) certificate and upload them safely to Git. The private key is the only means of decrypting the secret, so this should obviously be kept hidden. It should be installed closest to the place where the secrets will be used. So how do we put this to practice?

# Bitnami SealedSecrets
[SealedSecrets](https://engineering.bitnami.com/articles/sealed-secrets.html) by Bitnami is an open-source tool which can be used in Kubernetes environments which works exactly like explained above. It extends Kubernetes with a custom resource definition (CRD). Once installed, a Kubernetes controller starts running. It will generate a private key and a certificate. The certificate should be made public to the developers of the company. The process then works as follows:

1. The developer installs the CLI tool that comes with SealedSecrets, called “kubeseal”.
1. The developer creates a regular Kubernetes Secret, exactly like how it should end up in the Kubernetes cluster.
1. The developer runs kubeseal against the Secret to encrypt the contents and to generate a SealedSecret.
1. The developer pushes the SealedSecret to their GitOps repository.
1. The GitOps repository pipeline deploys the SealedSecret to Kubernetes.
1. The SealedSecret controller notices the SealedSecret and decrypts it into a regular Kubernetes secret.
1. Now it can be used like any other regular Kubernetes secret.

![Showing how kubeseal CLI can be used to encrypt a regular Kubernetes Secret.](/images/posts/sealedsecret-usage.png "Showing how kubeseal CLI can be used to encrypt a regular Kubernetes Secret.")

_The developer creates a SealedSecret from a regular Kubernetes Secret using kubeseal._

This sounds simple, and it really is! Both the Kubernetes and CLI installation are done in a matter of minutes and the usage is simple as well.

### Keep this in mind, though…
 - Only the private key can decrypt these SealedSecrets. Once the private key is lost, the secret is lost as well. That’s why it’s smart to back up the private key. Next to that, writing down how to regenerate secrets and backing up non-regeneratable secrets will help you to recover in emergency situations.
 - Even though the exposure of the secret is now limited to one developer, you can think of ways to let the secret have no exposure at all. For example, a shared secret between two services can easily be generated, encrypted and committed to the GitOps repository by a script.
 - Make sure to limit the number of people who are able to see the unencrypted Secrets or the private key on Kubernetes. Additionally, things like auditing and 4-eyes processes can be put in place to ensure even those people cannot access them without permission.
 - Specific to SealedSecrets: Only one private key is created over a whole cluster, but the name of the secret and the namespace are used when encrypting it. So, when you are changing either the namespace or the name, make sure to encrypt it again, otherwise it cannot be decrypted.

# Alternatives
If you are running on Kubernetes and making use of Hashicorp Vault already, you can also make use of [Vault sidecar injectors](https://www.vaultproject.io/docs/platform/k8s/injector). Then you can inject secrets into your container using annotations on your Pod templates. This is definitely interesting, but the annotation template syntax can be quite cumbersome and error prone sometimes. You also need a sidecar next to every Pod. While this does not have to be a problem, it certainly increases the complexity of your application architecture.

I don’t know of any similar tools on non-Kubernetes environments, but I can imagine they exist, because the essence of such a tool is not something new. Writing a tool like this yourself is something I would never recommend – security is always hard to do right – but if there is no alternative it is always a possibility.

# Conclusion
Secrets are historically a hassle to do right. Products like Hashicorp Vault or Kubernetes Secrets can be used to store secrets, but then secret management is not integrated in application development, which results in an error-prone system. GitOps aims to store desired state in Git. Plaintext secrets cannot be stored in a repository, but they can when they are encrypted asymmetrically. A tool to use for this is Bitnami SealedSecrets. It is easy to install and use, but some processes should be put in place to use this safely.

---

This blog first appeared [here](https://blogs.infosupport.com/gitops-storing-secrets-safely-in-git/).