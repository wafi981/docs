Introduction To Project
***********************

OpenStack is a private cloud and manages end-to-end resources needed in a private cloud from Bare metal to data plane
to control plane. OpenStack was mainly used to manage virtual machines, but it is also used to manage Containers over
time. There are various implementations available for deploying OpenStack i.e., OpenStack On Kubernetes(OOK),
OpenStack on OpenStack(OOO); among various deployments, each deployment has its own pros and cons. Among all,
OOK has the least overhead and better management. Airship is one of the projects which uses OOK. Airship automates
cloud provisioning and management by establishing Under Cloud Platform(UCP) leveraging Kubernetes. Airship also
uses OpenStack-Helm for deployment activities. For interoperability and better adoption, a new project was launched in
OpenStack named as OpenStack-Helm.

OpenStack-Helm provides a collection of Helm charts that simply, resiliently, and flexibly deploy OpenStack 
and related services on Kubernetes. OpenStack Helm helps in making OpenStack containerized means each of its 
services like Nova, Keystone; Neutron is running from a container. Currently, many Container Orchestration 
Engines are available like Docker Swarm, Kubernetes, Apache Mesos, etc. For managing
containers, CNCF launched Kubernetes. It is an orchestrator that manages Life Cycle Management of Containers.
Kubernetes is now a de facto for Multi-cloud. Multi-cloud means workload can be switched between different public
clouds, e.g., AWS, Azure, GCP. Kubernetes also plays a key role in Hybrid Cloud. In a hybrid cloud, workloads can
be switched between any private cloud and a different public cloud. Kubernetes installs containerized OpenStack
components with the help of OpenStack-Helm in a declarative way

OpenStack Deployment, Management and Upgradation (DMU) is a tedious task and requires many skills. To overcome
this and provide abstraction, we propose **KupenStack**, which automates the DMU and provides a seamless experience
for developers. **KupenStack** can also be used in both hybrid or multi-cloud like Kubernetes. **KupenStack** can provide
consistent, repeatable, easy, scalable, customizable OpenStack deployments.

