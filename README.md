# SIG-Windows

## Introduction

Sig-windows is a community effort to bring windows server container workloads to Kubernetes.  The current goal is not to have a windows node be the master, but to have it run the Kubelet and be able to run pods with windows server containers as part of a larger cluster with a Linux master node.  There are a couple of streams of work  with varying levels of activity currently in progress to achieve this.

Some of the remaining work is:

* Networking: Multi-container pod networking
* Metrics
* Secrets
* Config Maps
* Volumes
* Storage

**Note: This document is a work in progress and is intended to be updated with community input regularly as progress is made.  If you have any additions or changes you would like to add to this document, please submit a PR.**

## Networking

Networking was one of the first major hurdles.  Many of the current technologies used for cluster networking are not cross platform, and do not run on Windows.  One solution that does fulfill this requirement is [Open vSwitch](http://openvswitch.org/).

### Open vSwitch

https://github.com/openvswitch/ovs

Open vSwitch (OVS/OVN) is an [open-source](https://en.wikipedia.org/wiki/Open-source) implementation of a distributed virtual [multilayer switch](https://en.wikipedia.org/wiki/Multilayer_switch). 
The main purpose of Open vSwitch is to provide a [switching](https://en.wikipedia.org/wiki/Network_switch) stack for [hardware virtualization](https://en.wikipedia.org/wiki/Hardware_virtualization) environments, 
while supporting multiple protocols and standards used in [computer networks](https://en.wikipedia.org/wiki/Computer_network).  Open Virtual Network (OVN) is a part of OVS that was added to provide higher level abstractions over OVS, building logical networks with logical switches, logical routers, logical datapaths, and logical ports.  Documentation can be found at [ovn-architecture](http://openvswitch.org/support/dist-docs/ovn-architecture.7.html)

The basic architecture is that there is a master node, which contains the ovn northbound db & ovn southbound db, which contain the desired logical network, and it's corresponding datapath flows respectively.  Each of the other nodes will run an ovn-controller which will will receive configuration changes in the southbound db and be able to update the physical flows accordingly.

### ovn-kubernetes

https://github.com/openvswitch/ovn-kubernetes

The ovn-kubernetes project is a wrapper around Open vSwitch which creates the necessary logical network for a heterogeneous Kubernetes cluster.  It is currently written in Python, though there is a port to Go which has been started.  Each node that is setup has an ovn-k8s-overlay init command run (master-init|minion-init|gateway-init) with some of the networking configuration passed in which sets up the networking for that node by making calls to through to ovn and ovs.

#### ovn-kubernetes watchers/services

##### Master node - ovn-k8s-watcher
This watcher runs on the master node, and watches kubernetes events.  When a traditional linux based pod is deployed, it is responsible for creating logical ports required for pod networking.  Each pod will get a logical port added to it's worker node logical switch.  The watcher is a service that runs at startup via systemd, and logs to /var/log/openvswitch/ovn-k8s-watcher.log .

##### Gateway node - ovn-k8s-gateway-helper
Not explicitly a watcher, but another service that runs at startup to de-multiplex traffic on the gateway when a single interface is used for both management traffic (ssh) and the cluster's North-South traffic.

##### Windows Worker node - ovn-k8s service
This service runs in the background on windows, and intercepts docker calls for container creation and creates the logical networking necessary for the pod.


## Metrics

Stats and metrics are another area where Windows and Linux diverge.  Usually, stats are populated from CAdvisor, which reads them directly from cgroups on Linux. For Windows, CAdvisor is not supported and there are no plans to support it on Windows in the future https://github.com/google/cadvisor/issues/1394.

There is some work to provide status via the cadvisor shim on windows using a combination of native windows perf monitoring subsystem (PerfCounters to get node stats) and calling out to the docker engine REST API to get container stats.  The proposal can be found at https://github.com/kubernetes/kubernetes/issues/49398 .

There is also ongoing work to allow kubelet to consume stats via the CRI.  Once this is finalized, the work in the cadvisor shim is intended to be ported to the CRI implementation.


## Secrets

Investigation into this feature is ongoing.
## Config Maps

Investigation into this feature is ongoing.
## Volumes

Investigation into this feature is ongoing.
## Persistent Storage

Investigation into this feature is ongoing.


# Demo

The demo at [kubernetes-ovn-heterogeneous-cluster](https://github.com/apprenda/kubernetes-ovn-heterogeneous-cluster) walks through the manual provisioning and setup of an cluster on Google Cloud Platform (though the steps to setup on different cloud provider would be very similar). 

The cluster consists of:

* 1 Linux Kubernetes master node, which also doubles as the ovn/ovs master
* 1 Linux Kubernetes worker node, which will run Linux based pods
* 1 Windows Kubernetes worker node, which will run Windows Server Container pods
* 1 Linux Gateway node which acts as the gateway between the logical network provided by OVS/OVN, and the internet


# Get Involved

For general developer questions: Use the #sig-windows slack channel

Sign up for a free account at: http://slack.kubernetes.io 

Go to the #sig-windows channel and ask for help. e.g. I'm having problems deploying pods to my Windows node  

Bi-Weekly Standup (Every other Tuesday, 16:30 UTC) - http://www.zoom.us/my/sigwindows

# Links

Sig-Windows community page - https://github.com/kubernetes/community/tree/master/sig-windows 

ovn-kubernetes project page - https://github.com/openvswitch/ovn-kubernetes

Open vSwitch project page - https://github.com/openvswitch/ovs

Heterogeneous Cluster Demo - https://github.com/apprenda/kubernetes-ovn-heterogeneous-cluster