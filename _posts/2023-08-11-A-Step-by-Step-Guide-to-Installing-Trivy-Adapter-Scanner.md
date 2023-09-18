---
layout: post
title: A Step-by-Step Guide to Installing Trivy Adapter Scanner on a Kubernetes Cluster
date: '2023-08-11T10:47:00.000+05:30'
author: Middlewarebytes
sitemap:
  lastmod: 2023-09-01
  priority: 0.7
  changefreq: 'monthly'
  exclude: 'no'
tags:
- Kubernetes
- Docker
- DevSecOps
- Harbor
- Trivy
- cloud computing
readtime: true
---


 ![Photo by <a href="https://unsplash.com/@scottwebb?utm_source=unsplash&utm_medium=referral&utm_content=creditCopyText">Scott Webb</a> on <a href="https://unsplash.com/photos/yekGLpc3vro?utm_source=unsplash&utm_medium=referral&utm_content=creditCopyText">Unsplash</a>](/img/postimages/security.jpg)
Photo by <a href="https://unsplash.com/@scottwebb?utm_source=unsplash&utm_medium=referral&utm_content=creditCopyText">Scott Webb</a> on <a href="https://unsplash.com/photos/yekGLpc3vro?utm_source=unsplash&utm_medium=referral&utm_content=creditCopyText">Unsplash</a>
  

### Introduction:

Kubernetes has revolutionized container orchestration, enabling developers to efficiently manage and deploy containerized applications at scale. However, ensuring the security of these containers is equally important. One powerful tool for container security scanning is Trivy, and in this blog post, we will guide you through the process of installing Trivy Adapter Scanner on your Kubernetes cluster.

### Prerequisites

Before we begin, make sure you have the following prerequisites in place:

- A working Kubernetes cluster.
- kubectl command-line tool installed and configured to access your cluster.
- Helm, the Kubernetes package manager, installed on your local machine.

### Installation



#### Step 1: Install Helm


If you don't have Helm installed, you can follow the official Helm installation guide for your specific platform: [Helm Installation Guide](https://helm.sh/docs/intro/install/){:target="_blank"}.

#### Step 2: Add Trivy Helm Repository


To install Trivy Adapter Scanner using Helm, you need to add the Trivy Helm repository to your Helm client. Open your terminal and run the following command:

```bash
helm repo add aquasecurity https://aquasecurity.github.io/helm-charts
```
#### Step 3: Update Helm Repositories


To ensure you have the latest charts from the Trivy repository, update your Helm repositories:

```bash
helm repo update
```
#### Step 4: Install Trivy Adapter Scanner


Now that you've added the Trivy Helm repository and updated your Helm repositories, you can install Trivy Adapter Scanner on your Kubernetes cluster. Run the following command:

```bash
helm install trivy-aquasec aquasecurity/trivy-adapter
```
This command installs Trivy Adapter Scanner with the default configuration. You can customize the installation by providing your own values via a ``values.yaml`` file or by specifying them directly in the Helm command.

#### Step 5: Verify the Installation


Once the installation is complete, you can verify that Trivy Adapter Scanner is running in your cluster by checking the deployed pods:

```bash
kubectl get pods
```
You should see pods related to Trivy Adapter Scanner running.


 ### Installing Trivy as an Interrogation Service for Harbor


[Harbor](https://goharbor.io/docs/2.0.0/) provides static analysis of vulnerabilities in images through the open source projects Trivy and Clair.

Sometimes it is necessary to connect Harbor to other scanners for corporate compliance reasons, such as an organization might require you to connect to multiple sets of scanners which helps to capture different CVE sets from multiple vulnerabilty databases, for such a requirement multiple scanners can be connected with a registry like Harbor through its embedded interrogation service. To see the list of compatible of scanners with harbor, see the list [here](https://goharbor.io/docs/2.0.0/install-config/harbor-compatibility-list/#scanner-adapters){:target="_blank"}. 

 - Complete all the steps from **Step 1 to Step 3** as mentioned above.

 > The harbor scanner adapter is installed using the helm chart ```harbor-scanner-trivy```

 - To install the chart with the release name my-release:

 ```bash
 helm install my-release aqua/harbor-scanner-trivy
 ```
 > The command deploys scanner adapter on the Kubernetes cluster in the default configuration. Refer the The [Parameters](https://github.com/aquasecurity/harbor-scanner-trivy/tree/main/helm/harbor-scanner-trivy#parameters){:target="_blank"} section  which lists all the the parameters that can be configured during installation.

 - Configure Trivy scanner from Harbor Interrogation Menu

 -  Log in to the Harbor Console with an account that has Harbor **system administrator** privileges.

 -  Expand Administration, and select Interrogation Services.

 ![Harbor Interrogation Service](/img/postimages/harbor/interrogationsvc.png)

 -  Click the New Scanner button.

 ![Add scanner](/img/postimages/harbor/addexternalscanner.png)

 -  Enter the information to identify the scanner.

    -  A unique name for this scanner instance, to display in the Harbor interface.
    -  An optional description of this scanner instance.
    -  The address of the API endpoint that the scanner exposes to Harbor.

 -  Select how to connect to the scanner from the **Authorization** drop-down menu.

 ![Select Authorization Method](/img/postimages/harbor/interrogationsvc.png)

  - **None:** The scanner allows all connections without any security.
  - **Basic:** Enter a username and password for an account that can connect to the scanner.
  - **Bearer:** Paste the contents of a bearer token in the Token text box.
  - **APIKey:** Paste the contents of an API key for the scanner in the APIKey text box.

 - Select Skip certificate verification if the scanner uses a self-signed or untrusted certificate. (**OPTIONAL**)

 - Select Use internal registry address if the scanner should connect to Harbor using an internal network address rather than its external URL. (**OPTIONAL**)

 > NOTE: To use this option, the scanner must be deployed in a network that allows the scanner to reach Harbor via Harbor’s internal network.

 - Click Test Connection to make sure that Harbor can connect successfully to the scanner.

 - Click Add to connect Harbor to the scanner.



### Conclusion

In this blog post, we walked you through the process of installing Trivy Adapter Scanner on your Kubernetes cluster. Container security is crucial in today's containerized world, and tools like Trivy help you identify and remediate vulnerabilities in your container images. By following these steps, you can enhance the security of your Kubernetes applications and ensure a safer container environment.

**Happy scanning!**