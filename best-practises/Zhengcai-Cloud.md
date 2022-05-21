# Zhengcai Cloud's On-premise Business Delivery Practise based on sealer

author: [@lllwan](https://github.com/lllwan)

The Internet has developed rapidly in recent years. There are a large number of promising technologies which spring up like mushrooms to follow the rapid growth of business. Cloud-native container-centric technologies are growing so quickly, and Kubernetes, the de facto standard for container orchestration, is undoubtedly the most notable one.

Kubernetes solves the problems of large-scale application deployment, resource management and scheduling. However, to be honest, it is still not so convenient for business delivery, especially in on-premise scenarios. Moreover, deployment of Kubernetes is so complex and costing. Among the emerging applications oriented with the Kubernetes ecosystem, there is always a lack of business-centric applications that can integrate business, middleware, and Kubernetes cluster for integrated delivery. 

[sealer](https://github.com/sealerio/sealer), an open-source project initiated by the Alibaba Cloud, remedies the Kubernetes deficiency in integrated delivery. sealer considers the overall delivery of cluster and distributed applications with a very elegant design scheme. As a representative of the government procurement sector, [Zhengcai Cloud](https://www.zhengcaiyun.cn/) has used sealer to complete the overall on-premise delivery of large-scale distributed applications. The delivery practice proves that sealer has the flexible and powerful capability for integrated delivery.

<p align="center">
  <img src="https://user-images.githubusercontent.com/9465626/169629972-d2d72c82-cad7-46f3-b437-e65421856be9.png">
</p>


## Background

Customers of Zhengcai Cloud's on-premise delivery are government enterprises. They usually need to deliver large-scale business with 300-plus business components and 20-plus middleware. The infrastructure of delivery targets is quite different and uncontrollable, and the network limits are very strict. In some sensitive scenarios, the biggest pain point of business delivery is handling of **deployment dependency** and **delivery consistency**, which are completely isolated from the network. The unified delivery of business based on Kubernetes achieves the consistency of the running environment. However, it is still urgent to solve problems, such as：

* unified processing of all images
* various packages depended during the deployment
* the consistency of delivery systems.

<p align="center">
  <img src="https://user-images.githubusercontent.com/9465626/169628526-22c24680-bf36-4dc6-9d92-55c8937ad77f.png">
</p>

As shown in the figure above, the process of localization delivery of Zhengcai Cloud is divided into six steps: 

* Confirm user requirements
* give resource demand to users
* obtain a resource list provided by users
* generate preparation configurations based on the resource list
* prepare deployment scripts and dependency
* complete delivery 

Pre-preparation and delivery require a lot of worker force and time to be prepared and deployed. 

## Problems of On-premise Delivery

In cloud native era, the emergence of Docker deals with the environment consistency and packaging problems of a single-host application. The docker-based delivery of business no longer spends a lot of time deploying environment dependencies like traditional delivery. Then, the advent of container orchestration systems (such as Kubernetes) deals with the unified scheduling of underlying resources and unified orchestration of applications runtime. However, the delivery of a complex business itself is a huge issue. For example, Zhengcai Cloud needs to deploy and configure various resource objects, such as helm chart, RBAC, Istio gateway, CNI, and middleware, and deliver more than 300 business components. 

Zhengcai Cloud is in a period of rapid business development. The demand of on-premise deployment projects is constantly increasing. However, it is more difficult to support actual needs with high-cost delivery methods. Learning how to reduce delivery costs and ensure delivery consistency is the most urgent problem for O&M teams to solve. 

## The Discovery of sealer

In the early days, Zhengcai Cloud used Ansible to deliver business. The Ansible solution achieves automation and reduces delivery costs to some degree. However, it causes the following problems: 

1. Ansible only solves problems during deployment and needs to prepare the dependency required for deployment separately. The preparation of dependency and the verification of availability incur extra costs. In addition, the localization scenarios of Zhengcai Cloud strictly limit the external network. Therefore, it is not feasible to obtain dependency directly from the external network. 

2. Using Ansible to cope with differentiated requirements will be exhausting. Users have different requirements and business dependency in the on-premise delivery scenarios of Zhengcai Cloud. It takes a lot of time to debug the Ansible playbook during each delivery and re-editing.

3. When complex control logic is involved, the declaration language of Ansible is weak. 

4. It is required to prepare the running environment of Ansible before you deploy and deliver it. Thus, zero dependency cannot be realized. 

Ansible does more bonding and O&M work with simple logic. With increasing of on-preimise projects, the disadvantages of Ansible are beginning to appear. Each on-premise project requires a lot of time investment. The Zhengcai Cloud O&M Team has begun to explore the optimized direction of the delivery system. We have analyzed many technical solutions. The current Kubernetes delivery tools focus on the uppler layer application delivery instead of the delivery of the overall business layer. It can be encapsulated based on cluster deployment tools, but this solution is not fundamentally different from using Ansible to deploy upper-layer services after the deployment of clusters. 

Fortunately, we discovered the sealer project. sealer is a packaging and delivery solution for distributed applications. It solves the delivery problem of complex applications by packaging distributed applications and their dependency together. Its design concept is very elegant, and it can manage the packaging and delivery of the entire cluster in the ecosystem of container images. 

<p align="center">
  <img src="https://user-images.githubusercontent.com/8912557/117400612-97cf3a00-af35-11eb-90b9-f5dc8e8117b5.png">
</p>

When using Docker, we use Dockerfile to define the running environment and packaging of a single application. Correspondingly, the technical principle of sealer can be explained by the analogy of Docker. The entire cluster can be regarded as a machine, and Kubernetes can be defined as the operating system. The applications in this *operating system* are defined by Kubefile and packaged into images. Then, sealer run can deliver the entire cluster and applications (like Docker run) to deliver a single application.

<p align="center">
  <img src="https://user-images.githubusercontent.com/9465626/169629191-365974a5-16a4-4c7b-9f20-57d8807590d8.png">
</p>

We invited sealer's maintainers to Zhengcai Cloud to communicate with each other. Acatually We encounterd some problems and pitfalls, since sealer was born for short time as a new project. And we also found many issues that did not meet the demand. However, we didn’t give up because we had great expectations and confidence in the design model of sealer. We chose to work together with the community to grow together. The final successful implementation practices also proved that our choice was correct. 

## Community Collaboration

When deciding to cooperate with the community, we conducted a comprehensive evaluation of sealer. Considering our requirements, here are the main problems: 

1. The cost of image cache is too high. At first, sealer only provides cloud build, so the premise of packaging sealer cluster images aims to pull up a cluster based on cloud resources. This method costs too much. Therefore, we propose the build method of lite build, which supports image analysis and direct caching by parsing helm, resource definition YAML files, and image list. The lite build is the lowest-cost build method. Instead of pulling up a cluster, you only need a host that can run sealer to complete the build. 

2. After the business is delivered, there is no check mechanism. You need to manually check the status of each component in a Kubernetes cluster. Therefore, we provide the feature of checking the status of clusters and components. 

3. Some configurations of early sealer are fixed in rootfs. For example, the deployment host of the registry is fixed on the first master node. We need to customize the configuration of the registry in actual scenarios. Therefore, we offer the feature of customizing the configuration of the registry. 

4. After deploying a cluster based on sealer, you still need to add nodes to the cluster. Therefore, we provide the feature of sealer join. 

In addition, it is necessary to talk about several practical and powerful sealer features when we implement the sealer solution: 

1. It must be mentioned that the cluster images generated by sealer can be directly pushed to a private Docker image repository such as Harbor. Then, the cluster images generated by sealer can expand features and rebuild based on the existing images like the Docker image. 

2. The sealer community has optimized registry and Docker to support multi-source and multi-domain proxy caching. This is a useful feature. When dealing with image dependency, we need to change the address of an image to cache the image. For example, we need to cache a public image to a private image repository. The address of the image referenced by the corresponding resource object also needs to be changed to the address of the private image repository. However, the built-in registry of sealer is optimized to match the cache without modifying the image address. In addition, when the built-in registry in sealer is used as a proxy, it can be the proxy for multiple private image repositories, which is very practical in scenarios with multiple private repositories. 

## Implementation Practice

We redefined the delivery process to use sealer. The delivery of business components, containerized middleware, image cache, and other components is completed using sealer through Kubefile. The sealer lite build mode is used to automate the parsing and built-in caching of dependency images. 

sealer is used to save the complex process logic and dependency processing logic of a great deal of application delivery, which simplifies the implementation. The continuous simplification of the implementation logic makes delivery possible on a large scale. We use the new delivery system in practice scenarios. The delivery period is shortened from a 15 person-day to a 2 person-day. In addition, the delivery of a cluster containing 20 GB business image cache, more than 2,000 GB memory, and 800-plus core CPU is realized. In the next step, we plan to continuously simplify the delivery process so a freshman can complete the delivery of an entire project with simple training. 

<p align="center">
  <img src="https://user-images.githubusercontent.com/9465626/169629014-4edc77b0-3338-49be-ab65-37612e825dc7.png">
</p>

## Future Outlook

The success of the implementation of sealer is the result of the delivery system and the power of open-source. Moreover, we explore a new model of cooperation with the community. In the future, Zhengcai Cloud will continue to support and participate in the construction of the sealer community and contribute more to the community according to actual business scenarios. 

As a new open-source project, sealer is imperfect. It has problems that need solving and features that need optimizing. Also, there are still more requirements and business scenarios to be realized. We hope sealer can serve more user scenarios through our continuous contribution. At the same time, we also hope more partners can participate in the community construction to make sealer more promising.

## Appendix

[Zhengcai Cloud Co., Ltd.](https://www.zhengcaiyun.cn/) was founded on August 8, 2016. Based on the world's leading cloud computing, big data, artificial intelligence and other digital technologies, the company has built the China's first government procurement cloud service platform "Zhengcai Cloud Platform". On the basis of promoting government procurement and government digitization, the company continuously extends its service capacity, independently develops and operates the open platform for government and enterprise procurement -- Lecai Cloud platform, providing integrated digital procurement solutions for many enterprise users, industry users and government units.