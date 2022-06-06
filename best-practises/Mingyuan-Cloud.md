# Mingyuan Cloud Skyline Platform's sealer-based On-premise Delivery Practice

author: Mingyuan Cloud Skyline Container Platform BU

<p align="center">
  <img src="https://user-images.githubusercontent.com/9465626/172105044-5c67237d-ff53-42d0-ba10-a0709cbcf375.png">
</p>

## Introduction of Mingyuan Cloud Skyline Platform

Mingyuan Cloud is a leading digital solution service provider for real estate ecological chain in China. It has provided digital solutions and products for more than 7,000 real estate development and operation enterprises. The Mingyuan Cloud Skyline platform contains many core competencies and supports the construction of stable and sensitive applications through low-code methods. Through the flexible combination of hybrid cloud, integration and being integrated, integrating data analysis and AIoT scenario-based application capabilities, Mingyuan Cloud keeps building a stable, efficient and open enterprise-level digital symbiosis ecological platform. Mingyuan Cloud Skyline Platform creates a full range of open capabilities for the platform layer, business middle platform layer and application layer for enterprises.

## Delivery Challenges of Skyline PaaS Platform 

<p align="center">
  <img src="https://user-images.githubusercontent.com/9465626/172083843-e72bb2e6-295f-42c2-aecf-32be0639d6c3.png">
</p>

Skyline PaaS platform is a benchmark scenario of Mingyuan Cloud based on nearly 2,000 enterprises. It integrates the best practices and forward-looking research of digitization in the real estate industry, and is built through agile development, global integration, process-driven, data intelligence, technological innovation, ecological integration and other capabilities. The integrated digital platform of Mingyuan Cloud is the technical base for Mingyuan Cloud to build the digital transformation of the real estate industry. With the growth of business, customers' demand for on-premise delivery based on private data centers has gradually increased, and product delivery has become a major pain point. Mingyuan Cloud Skyline PaaS platform follows the standards of cloud native Kubernetes platform from the beginning of design, and the application is fully based on the self-developed container release management platform. Although the overall capability of the powerful Kubernetes platform is very strong, the overall delivery cycle after the integration of many business applications, middleware systems and Kubernetes on the Skyline platform is extremely long and long, and the delivery quality is difficult to control. Using Kubernetes as the core "infrastructure" to smooth out the cross-cloud environment is a huge challenge for traditional operation and maintenance deployment and delivery.

Fast, stable and automated, has become the primary goal of Mingyuan Cloud Skyline PaaS platform delivery field.

## How to tackle delivery optimization

In the traditional delivery mode, in order to achieve the goal of "deployment within one hour", we use one-click scripts, Ansible and other methods to improve the efficiency of the deployment of the basic environment. However, in the process of automation, it is more difficult, especially in the process of product operation and maintenance, it is still necessary to flexibly add or delete nodes for the cluster to ensure the high availability of the Kubernetes cluster, the master node, and the maintenance of the cluster certificate.

At that time, we were very excited to see Alibaba's open source sealer solution in the community, now a project under CNCF. we think this is a powerful tool for delivery. It can package Kubernetes and related applications in a form similar to Docker packaging container images. After learning about sealer, we quickly customized our own cluster image and completed the test, which met our expectations.

<p align="center">
  <img src="https://user-images.githubusercontent.com/9465626/169629191-365974a5-16a4-4c7b-9f20-57d8807590d8.png">
</p>

The concept of ealer is easy to understand. If the user is familiar with Docker, the understanding of sealer is not a problem.

<p align="center">
  <img src="https://user-images.githubusercontent.com/9465626/172084034-e5a27821-7f26-457b-8929-f00cd1ee752c.png">
</p>

The comparison between sealer's cluster image and Docker image:

* A Docker image is the definition of a single service that determines how it runs in a container
* Cluster image is the definition of a type of service and determines how this type of service runs on Kubernetes

<p align="center">
  <img src="https://user-images.githubusercontent.com/9465626/172084079-2d378db4-4913-463b-b2b1-0067dbefb0b7.png">
</p>

## sealer-based Practice from Mingyuan Cloud

### Construct a Kubefile for Mingyuan Cloud Skyline Platform

The use of sealer is very simple. After understanding the concept of the sealer Kubefile, the engineers of Mingyuan Cloud Skyline PaaS platform team quickly started to prepare the Kubefile of the skyline platform. After a series of business sorting and splitting, the following Kubefile was finally written.

Step1: Using the sealer image construction capability, we customize an image that meets our own business needs and package the overall image.

<p align="center">
  <img src="https://user-images.githubusercontent.com/9465626/172084121-e66fabbd-ff1f-4a5b-9523-eeb4ccf0ee03.png">
</p>

In the above Kubefile, we can clearly see that in sealer's cluster image:
* Added the corresponding helm chart package.
* Basic configuration of the installation environment.
* Upgrade a number of middleware to the specified version, such as flink, redis, ceph, postgresql, loki, etc.
* Finally, the execution command is set for the cluster mirroring.

### Define a Clusterfile to represent cluster

<p align="center">
  <img src="https://user-images.githubusercontent.com/9465626/172084177-65a134c0-77f8-49c9-bd96-7b7f50e9d048.png">
</p>

Through the above Kubefile, execute the sealer build command to build a complete cluster image. After having the cluster image, sealer completes the definition of the running cluster by specifying the Clusterfile, and finally runs the cluster image.

Step2: Automatic deployment and completion of metallb configuration through Clusterfile.

Step3: Add the agent configuration of the internal publishing platform to realize the communication and management with the cluster.

<p align="center">
  <img src="https://user-images.githubusercontent.com/9465626/172084244-a50a8739-1c54-4161-9a8a-deeb7f260d52.png">
</p>

The arrival of clusterfile v2 supports more advanced configurations. Since the default number of 110 pods on a single node cannot meet business requirements, in advanced configuration, we can increase the number of pods and optimize more cluster parameters.

### Image Cache Optimization

The problem of pulling application images is a major pain point in on-premise environment. Due to the disconnection of the network in on-premise environment, a way to directly package all images in a unified manner becomes particularly important. When sealer uses docker proxy and docker registry to build the cluster image of the application, it directly pulls all the images used by the application to the local warehouse. When the application image exists in the local repository and the cluster image starts and runs, the docker engine has the ability to pull from the local repository.

<p align="center">
  <img src="https://user-images.githubusercontent.com/9465626/172084280-379b0870-8950-4d13-b10e-0364e979813b.png">
</p>

### Practice to build a HA cluster

How to achieve load balancing in a high-availability architecture of a production cluster? sealer supports automatic configuration of multi-master nodes and loads, and uses lvscare to maintain ipvs rules. lvscare itself only monitors apiserver, and directly monitors apiserver on node. If it kneels down, remove the corresponding rules, and it will be added back automatically after restarting. It is equivalent to a dedicated load balancer.

<p align="center">
  <img src="https://user-images.githubusercontent.com/9465626/172084310-abe09677-34a8-45ed-a6dd-38489d94f81f.png">
</p>

Finally run our cluster in minutes with `sealer apply -f Clusterfile`.

## Efficiency comparison between sealer and traditional delivery modes

Full offline environment comparison sealer v0.3.3 version (landing version)

|Comparison|traditional mode|sealer mode|
|:-:|:-:|:-:|
|degree of automation|30%|90%|
|HA| manual HA| support HA natively|
|cert|one year| 100 years|
|degree of apps deployment automation|not at all|support|
|image pulling|time costing, need full access to internet| image cache localling, support offline|
|delivery time| two days| one hour|

## Summary and Future Plans

sealer solves the first step of delivery and makes cloud-native application delivery easy. The innovative and open-source technology concept made it a CNCF Sandbox project in less than a year. Driven by active community forces, sealer will bring more capabilities to resolve application packaging and smooth infrastructure differences, helping cloud native development.