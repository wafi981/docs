Cloud-Native OpenStack as-code on Kubernetes
********************************************

**KupenStack** is an OOK controller. KupenStack architecture contains controller patterns with the automated implementation 
of complex operations. KupenStack provides OpenStack services(like Nova, Neutron) and resources(like
VM, Subnet ) as custom resources. Instead of building Cloud-Native OpenStack from scratch, KupenStack proposes to
reuse Kubernetes control plane to make OpenStack as Cloud Native. This section starts with an introduction to the
KupenStack architecture and design principles. Then covers some observations on mappings between OpenStack and
Kubernetes concepts. Finally, giving a reference implementation example of the KupenStack model.

Architecture
============

OpenStack internally has a distributed service architecture, i.e., loosely coupled services communicate with each other
using REST APIs, RPC and messaging queues. This decoupled service architecture of OpenStack gives benefits which
include 
#. The ability to customize and cherry-pick components to install 
#. Freedom to scale individual components
#. Interoperability to integrate components from multiple vendors. 

The KupenStack architecture promises to keep these
benefits safe by not changing OpenStack implementation in anyways. KupenStack architecture proposes to deploy
OpenStack components as containers on top of Kubernetes. Fundamental reasons for KupenStack to choose containers
are:

* **KupenStack can operate on multi-vendor OpenStack deployments**
  KupenStack deploys the OpenStack services as containers and communicates to them with standard OpenStack APIs. This makes individual
  service images interchangeable with multiple vendor implementations

* **KupenStack can ensure OpenStack deployments that are highly portable**
  Container images can run on any Linux kernel-based operating system without any dependency failures. KupenStack components
  themselves will run as a containerized workload on Kubernetes; this builds trust for high portability

* **KupenStack can achieve OpenStack deployments that are robust and fault-tolerant**
  Container images follow the Image Immutability Principle(IIP). KupenStack promises to follow the immutable infrastructure
  paradigms [1] for configuration and version changes of OpenStack.

* **KupenStack can achieve Cloud Native Infrastructure operations**
  OOK deployment is possible by using containers. An OOK deployment provides scalability, resiliency, 
  and automation. With OOK deployments, KupenStack can easily build operations for OpenStack like zero-touch 
  provisioning, scaling, lcm, zerodowntime, self-healing, version upgrades, configuration management, and 
  many more. These operations themselves can be cloud-native by an implementation.

KupenStack extends Kubernetes using controllers and custom resources to build a cloud-native layer interacting between
OpenStack and Kubernetes. KupenStack follows two principles:

#. Should not change anything in OpenStack(i.e., Compatibility with any certified OpenStack)

#. Should not change anything in Kubernetes(i.e., Compatibility with any certified Kubernetes)

The degree of cloud-native behavior of custom resources depends on KupenStack implementation.


.. image:: Images/img1.png



This figure shows working of KupenStack architecture. The purple box represents KupenStack while blue and pink indicate 
Kubernetes and OpenStack respectively. Two flows are described by Dark arrow and Dotted arrow.

• **Dark Arrow**: The user tells what OpenStack components need to be installed and some configurations changes
  and version changes as declarative intent using custom resources. KupenStack-Controller-Manager notices the
  required changes and it queues corresponding controllers to take actions. The controller intelligently decides
  what action is required and calls the OpenStack-Operator to take those actions. The OpenStack-Operator
  builds Kubernetes manifest files for OpenStack container deployment using some existing templates. Applies
  those manifest to Kubernetes and then required OpenStack deployment gets spun up on top of Kubernetes.
  Now, suppose due to some reason some OpenStack service starts failing, then the OpenStack validation agent
  who is consistently monitoring these services, collecting logs, metrics notifies the required controller that what
  has failed and other details, logs. Controllers then decide what needs to be done, and if action is required then
  make a call to the OpenStack operator to make changes.

• **Dotted arrow**: The user tells what OpenStack resources are needed to be created, let’s say a Virtual Machine as
  a declarative intent using custom resources. KupenStack-Controller-Manager notices the required changes and
  it queues corresponding controllers to create a Virtual Machine. The controller makes a REST request to Nova
  service to create a Virtual Machine with the required spec. Nova creates Virtual Machine and returns an ID.


.. image:: Images/Img2.png


Now, controllers keep watching the status of Virtual Machine for failures and keep updating the status on the
custom resource. If for some reason Virtual Machines fails after some time, KupenStack self-heals it or else
notifies the user of the cause of the failure

The above-described architecture gives us the ”Cloud-Native OpenStack as-Code” pattern. Here ”as-Code” implies
that all APIs are machine-readable definition files, i.e. for using and operating this implementation of OpenStack we
need minimum manual operations. We only declare the required state and KupenStack does the heavy-lifting work for
managing OpenStack. In practice this would look like following YAML for provisioning virtual machine image in a
glance as shown in Fig. 3


.. image:: Images/img3.png


Mapping between OpenStack and Kubernetes
=========================================

OpenStack and Kubernetes are similar in some ways, primarily both orchestrate workloads. Discussing all similarities
is out of the scope of this paper. However, this section discusses some observations on the mapping between OpenStack
and Kubernetes concepts to build some high-level understandings. A specific implementation of KupenStack can follow
any approach while mapping OpenStack to Kubernetes through custom resources. KupenStack is a cloud-native layer
of interaction between OpenStack and Kubernetes, so it can map similar concepts and provide abstraction. This paper
doesn’t mandate that KupenStack should have a particular implementation for mapping resources. Evaluating the best
possible mapping of OpenStack resources to Kubernetes custom resources in KupenStack can be further explored.
Kubernetes cluster contains multiple nodes (bare metal or VM); a node can be either control-plane (Master
node) or compute nodes (Worker node) or both. OpenStack also follows the same kind of architecture as shown in Fig.5
Pods are the primary workload in Kubernetes and were initially designed keeping Virtual Machines in mind.
Hence, Pod is a group of containers sharing network namespace, have initContainers, etc. When pods are scheduled to
run on a node, Kubelet agent (on that node) talks to the container engine to create containers. In OpenStack when a
Virtual machine is scheduled to run on a node, libvirt agent (on that node) talks to the hypervisor to create a Virtual
Machine as shown in Fig. 4

.. image:: Images/img4.png

Namespace logically separates resources in a Kubernetes cluster. Similarly, OpenStack has Projects/Tenants to isolate
resources. Some resources in Kubernetes are cluster scoped (like Persistent Volume). In OpenStack, resources can be
used in two ways, i.e., Project-scoped or shared( Cross-project visibility) for example, Network resource in OpenStack
can be configured to be shared across Projects or be specific to Project. Hence Namespace in Kubernetes can be mapped
to Projects in OpenStack.
Domains are the highest level of logical isolation in OpenStack. Domains are organization-level isolation owning a
collection of users, groups, and projects. All resources created in a Domain are owned by that Domain and have no
visibility in other Domains. Kubernetes does not have any such implementation which isolates the control plane itself.
Both Kubernetes and OpenStack have a **Role-Based Access Control (RBAC)** to define authorization and access on
Namespaces and Projects, respectively, but the implementation of RBAC in both differ slightly. Every OpenStack
Project decides which User or User Group should have access to its resources using Roles. Roles have an RBAC policy
that defines rules to enforce on each OpenStack API endpoint. A rule defines whether an API endpoint can be accessed
by an admin, user, or both. For RBAC in Kubernetes, Roles and ClusterRoles are applied on Users or Groups using
RoleBindings and ClusterRoleBindings. Each Kubernetes Role and ClusterRole definition can define multiple rules,
where each rule defines access for resource types and verbs(actions) allowed on them. Kubernetes and OpenStack both
have Roles, but there cannot be any clear mapping between them in terms of usage.

For authentication, both Kubernetes and OpenStack have a notion of Users and User Groups, but both have entirely
different authentication systems. However, some work is available on using Keystone users to authenticate Kubernetes 
as this is a desirable feature for **Kubernetes on OpenStack(KOO)** for open-source projects like OpenStack
Magnum. As of this writing, we have no reference to using Kubernetes users(kubeconfig certificates) to authenticate to
OpenStack. While both Kubernetes and OpenStack allow authentication from external identity providers like LDAP.
OpenStack has hierarchical concepts like Regions, Host Aggregates and Availability Zones to provide logical partitioning within the cluster nodes for failure domains or to meet special hardware needs like sriov, GPU, etc. A
Region means complete OpenStack deployment level isolation. It provides separate API endpoints for all services
and resource(like Subnet, VM) pools. However, Regions share Keystone and Horizon services. Looking inside a
Region, we can have multiple Availability Zones, which in theory are for logical partitioning. In practice, the exact
implementation of Availability Zone depends on respective OpenStack services like Nova, Neutron, etc. Availability
Zones is one of the most ambiguous topics for beginners. For this section, let us look at Availability Zones for two
services, namely Nova and Cinder, to understand this problem.

In **Nova**, every Availability Zones is Host Aggregates but not vice versa. Availability Zones can contain one
or more Host Aggregates and vice versa. When a VM is created, it is scheduled to one Availability Zone, and two
Availability Zones cannot overlap with each other on nodes. However, Host Aggregates within an Availability Zone can
overlap.

In **Cinder**, we do not have Host Aggregates. When a volume is created, it gets scheduled in one of the
Availability Zones. Cinder Availability Zones also do not overlap with each other on nodes.
Kubernetes does not have these concepts like Regions, Availability Zones, Host Aggregates to keep things simpler.
Kubernetes provides the same values through concepts like node labels, selectors, and resource-specific features like
node affinity and anti-affinity in pods.

From the above observations, we see many concepts can be mapped while many cannot. It is not practical to discuss
all such mappings here, and we leave this section with the above observations as a reference to build these mappings
further as needed.


KupenStack Reference Implementation
===================================

Any **KupenStack** implementation should adhere to design and architectural principles specified in section 2.1. This
section will cover suggestions and proposals for a reference implementation. The following proposed KupenStack
implementation abstracts many OpenStack concepts with the intent of “OpenStack done the Kubernetes ways” and
simplified OpenStack usage. This also depicts the wide range of possibilities that KupenStack implementations can have.
We are proposing KupenStack for the first time, and we do not know what could be the best possible implementation for
OpenStack as-Code custom resource definitions. Also, the proposed implementation will be available as open-source 
for future research on KupenStack.
KupenStack deploys OpenStack services as containers and requires a high-level understanding of configuring and
integrating these containerized services. KupenStack gives freedom to choose images for these containers from any
source. For example, various open-source image projects like OpenStack-Ansible-LXC, kolla, LOCI can be

a source. Kubernetes has support for LXC containers and OCI containers both. The OpenStack community is also
leveraging containers with OpenStack in many ways.

KupenStack-controller can be implemented to deploy containers using custom manifest templates or using some existing
open-source ones. In this reference implementation, we propose to use some existing open-source OpenStack manifest
templates from OpenStack-Helm. Helm is a package manager for Kubernetes, and it neatly automates the
deployment of Kubernetes manifest files. Helm gives lots of flexibility in customizing these manifests. This is why we
propose OpenStack-Helm to be used along with our KupenStack reference implementation. Another benefit of adopting
OpenStack-Helm is interoperability between multiple vendors. The OpenStack-Helm project can work as a standard
reference for building any KupenStack reference implementation compatible images. There are other automation
projects which deploy containerized OpenStack, namely OpenStack-Ansible, Kolla-Ansible, Triple-O,
Kayobe (it is Kolla-Ansible with bare-metal provisioning support using Bifrost). Compared to the projects
mentioned above, we consider OpenStack-helm as the best option for KupenStack. With OpenStack-Helm, we benefit
from the automatic container lifecycle management feature of Kubernetes.

This Reference Implementation of KupenStack-controller implements operations like provisioning, maintaining, scaling,
self-healing, upgrading, etc., over OpenStack-Helm.

In this RI, we assume a one-to-one mapping between OpenStack and Kubernetes nodes for all resources and their
placements. We keep OpenStack controller nodes to be physically the same as Kubernetes control-plane nodes, and
OpenStack compute nodes to be the same as Kubernetes compute nodes as show in in Fig.5 . All custom resources (like
Instances, Image, KeyPair, Router, Subnet) created by KupenStack-Controller are managed accordingly. KupenStack
controller also performs intelligent cloud-native operations on OpenStack resources like self-healing of failed Instances
similar to failed pods in Kubernetes.


.. image:: Images/img5.png

For this implementation, we made Projects in OpenStack as abstract resources. Hence, rather than implementing
separate custom resources for managing OpenStack Project, KupenStack maps them to Kubernetes Namespaces (using
annotations in Kubernetes). As new Namespaces are created in Kubernetes, KupenStack creates corresponding Projects
in OpenStack. For every KupenStack custom resource created in the Kubernetes namespace, it’s actual resource is
created in the corresponding Project in OpenStack as shown in Fig.6

Domains had nothing to map in Kubernetes. So, for our reference implementation, we will have only one Domain(i.e.,
default) in KupenStack because logically, Domain == Kubernetes cluster.

For authentication, there are three ways to integrate Kubernetes and OpenStack. However, in this reference implementation, we propose a different approach by ignoring OpenStack RBAC and authentication completely. We propose
to use Kubernetes authentication only and depend on Kubernetes RBAC for all access management in KupenStack.
Using Kubernetes RBAC and authentication, OpenStack custom resources give unified and straightforward access
management in KupenStack. Kubernetes has a flexible and widely adopted RBAC. This approach also allows us to
change the default behaviour of Projects and resource visibility in OpenStack and builts OpenStack resources in the
Kubernetes native way of logical separation through name+namespaces.

We are mapping OpenStack Region with the Kubernetes cluster (Region == Kubernetes Cluster) and have only one
Region(named default). Availability Zones implementation depends on individual OpenStack resources and services.
For example, in Nova, we made a single Availability Zone(named default) for the Kubernetes cluster, i.e., Availability
Zone == Kubernetes cluster. Virtual Machines created through custom resources can benefit from OpenStack Host

.. image:: Images/img6.png

Aggregates through labels and selectors design in Kubernetes. Hence, Host Aggregates are implemented as an abstract
resource. This approach limits the use of OpenStack Regions, Availability Zone but gives perfect and smooth mapping
to Kubernetes concepts. In this implementation, only Host Aggregates are used to build use cases like a failure domain
as this is very similar to Kubernetes recommendations.
Similarly, more design choices can be made for OpenStack custom resources, and we will release more designs for other
resources in the future. This section described the flexibility of making such choices for Reference Implementation.

Networking between Kubernetes Pods and OpenStack Virtual Machines
=================================================================

It is desirable in a KupenStack environment that a Kubernetes Pod and OpenStack Virtual Machine are IP reachable to
each other, and there can be other Kubernetes networking concepts (like service discovery, network policy) on them.
There are open-source networking projects to achieve this with different approaches.
Kuryr-Kubernetes project tries to bring Neutron networking to Kubernetes pod in a Neutron-plugin agnostic way.
Kuryr gives a cni plugin that attaches port to Kubernetes pod using Neutron plugin. When we look at the mapping
between OpenStack and Kubernetes networking, we find the difference that Kubernetes leaves networking and follows a
simple principle that all pods should be IP reachable to each other without NATing. However, OpenStack builds network
infrastructure concepts like Network, Subnet, Router, etc., for its VM. One approach for providing connectivity between
pods and virtual machines can be to add extra interfaces to provide connectivity(while not disturbing the networking
concepts of the two platforms). Multus-CNI project is used to add an extra interface to the pod. Automation can
be applied on top of Kuryr and Multus CNI through the controller, making all pods IP reachable to VM while not
disturbing OpenStack networking. DANM CNI is an alternative to Multus. The choice of meta-cni-plugin should
not affect network performance as the principles on which these CNIs add multiple interfaces to pods is by delegating
requests to other CNIs that add actual interfaces. The second approach could be to build network plugins for CNI and
Neutron from scratch that understands both OpenStack and Kubernetes and the Calico project is one such popular
open-source project. OpenStack-Helm projects have charts for configuring calico policies.


