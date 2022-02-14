Considerations for possible KupenStack integrations and innovations
*******************************************************************

This section highlights the possible future integrations of KupenStack with various technologies and paradigms.

Virtual Machine Images with tagging
===================================

Process container technology open-sourced by Docker brought immutable images with tagging. Tagged images are
handy in the GitOps world as this provides version control over image releases. In the process of making OpenStack
cloud-native, we encourage to have similar tagging for Virtual Machines artifacts in KupenStack. Possibly, the OCI
standard can be reused to store Virtual Machine artifacts. This gives benefits like completely reusing OCI image
registries and hubs. KupenStack can internally implement a component that automatically converts these OCI artifacts
to the respective image format for the hypervisor. OCI standard is being used to create non-running container artifacts
by open-source projects like Cloud-Native Application Bundles [22] and manifest-bundles ( Kubernetes SIG
Cluster-Lifecycle) .

Building Kubernetes concepts on top of KupenStack Resources
===========================================================

With the introduction of KupenStack, there is space to build more Kubernetes concepts( like Daemonset, Deployment,
RepicaSets, Jobs, etc.) on top of KupenStack custom resources for better management of OpenStack resources.

Crossplane
===========

**Crossplane** is an open-source project that aims to build a unified control plane for multi-vendor resources running
remotely. For example, a Crossplane user should be able to define a unified API through Crossplane Composite Resource
Definition (XRD) for SQL service offered from multiple vendors covering AWS, GCP, etc. Crossplane XRD
assumes underlying Managed Resources to be implemented by the respective provider. As of this writing, Crossplane
maintains the implementation of providers aws, gcp, alibaba, azure, and ibm in its open-source repos, but there is no
implementation on managing remote OpenStack resources in any open-source project. These provider implementations
in Crossplane only focus on provisioning and managing resources running remotely and keep no account of topology of
workload placement as there is no cluster-level management. Crossplane assumes that cloud infrastructure is running
remotely and doesn’t focus on scaling, LCM, configuration management, and upgradation of the cloud infrastructure
itself. It should be possible for Crossplane XRDs to integrate any implementation of KupenStack custom resources
along with other public cloud alternatives.

Cloud Provider OpenStack
========================

**Cloud-Provider-OpenStack** is another open-source CNCF project under SIG-Cloud-Provider. With the
Cloud-Provider-OpenStack project, we can use OpenStack resources like Octavia-LoadBalancer and Cinder-Volumes as
a provider to Kubernetes resources. For example, with the Cinder CSI Plugin in the Cloud-Provider-OpenStack project,
we can get Persistence-Volume(PV) backed by Cinder inside Kubernetes. As of this writing, Cloud-Provider-OpenStack
has some work on integration on Octavia, Cinder, Keystone, Manila, Barbican, Magnum components to Kubernetes.
These integrations can be reused in KupenStack; this gives use cases like using Cinder volumes of OpenStack with
Pods as well as VM.

Cluster-API (CAPI) & OpenStack-Magnum
======================================

**Cluster-API** or **CAPI** is another open-source CNCF project under SIG-Cluster-Lifecycle. CAPI gives a
declarative way of provisioning, upgrading, and operating multiple Kubernetes clusters. CAPI can provision Kubernetes
clusters on various targets like bare metal, AWS, GCP, Azure, and OpenStack. Integration of CAPI-OpenStack with
KupenStack should be possible, giving us deployment topologies like Kubernetes-on-OpenStack-on-Kubernetes(
KO3K). This can be used in a scenario where admins operating on KupenStack deployment want to provide a separate
isolated Kubernetes cluster to their tenants. In some ways, this is multitenant Kubernetes. It is to note that the
OpenStack-Magnum project is also a Kubernetes-as-a-Service project but for OpenStack. Ironically, for KupenStack
both can be used. Hence choosing out of CAPI and OpenStack-Magnum depends on the user requirements.

Airship
=======

**Airship** is an open-source project that declaratively automates cloud provisioning. A cloud software may be OpenStack,
Ceph, Kubernetes, etc. For OpenStack, Airship internally uses OpenStack-Helm project, and this is where it overlaps
with KupenStack. But Airship does not provide OpenStack resources as custom resources in Kubernetes, and to achieve
this, Airship can integrate, and provision KupenStack based OpenStack-Helm cloud. So, hierarchy changes slightly,
and KupenStack can become the middle layer between Airship and OpenStack-Helm project.

Anthos
======

**Anthos** is a hybrid and multi-cloud platform leveraging Kubernetes; it lets us run containerized Kubernetes applications
without any modification across multiple platforms, clusters, and locations being on-prem or public cloud. As
KupenStack builds OpenStack resources as first-class citizens into Kubernetes, this means in the future, we can see a
true hybrid cloud, multi-cloud deployments for OpenStack through Anthos with no changes.

KubeFed
=======

**KubeFed(Kubernetes Cluster Federation)** is another open-source CNCF project which is under SIG-multicluster.
KubeFed is a multi-cluster level project which aims to coordinate and manage resources over multiple clusters.

KubeFed is extremely useful in complex multi-cluster use cases such as deploying multi-geo applications and failure
domains. As an integration to KupenStack, similar Federation implementation can be extended to KupenStack resources.
Here we would like to point out our example reference implementation in this paper. We proposed ignoring concepts of
Regions and Domain in KupenStack, as ignoring those simplifies design and such higher-level integration on top like
KubeFed become simpler to achieve and use in practice.

Cloud-Native Network Function Virtualization infrastructure
============================================================

There has been a lot of innovation and development in cloud technologies since the introduction of NFV. Over the last
two years, as of writing this paper, more and more SDOs(Standards Defining Organizations) like ETSI, CNTT have
adopted cloud-native in their definitions. Which makes Kubernetes a more suitable choice as seen in standard references
like RI2 (Reference Implementation 2) in CNTT. But still, the unsolved problem happened to be an NFVi platform
of choice that is cloud-native and supports both Container as well as Virtual Machine Workloads. KupenStack can
greatly solve this problem as it is Kubernetes based and brings OpenStack to Kubernetes, a complete IaaS platform.
This gives cloud-native platform to network orchestrators like ONAP for running 5G implementations having
cloud-native CNFs and VNFs.

KubeVirt, Kata Containers, etc.
===============================

**KubeVirt** is an open-source project under CNCF that adds virtual machines support in Kubernetes. Also, there are
Kata containers, which are lightweight Virtual Machines or MicroVMs that support running as Pods in Kubernetes.
These are open-source projects where KupenStack overlaps, and if requirements are only running Virtual Machines,
then KupenStack is more than just Virtual Machines. KupenStack brings a complete OpenStack IaaS.

Terraform-Provider-OpenStack
============================

**Terraform** is an open-source infrastructure as code automation tool that uses a declarative configuration language.
Terraform-Provider-OpenStack has a terraform implementation for provisioning OpenStack resources like Virtual
Machines, Images, Keypair, etc. This is where KupenStack overlaps with Terraform-Provider-OpenStack with the
advantage that KupenStack is Kubernetes control-plane based and can closely work with core Kubernetes resources. As
far as automation is concerned, Helm charts can easily provide automation to KupenStack resources.



