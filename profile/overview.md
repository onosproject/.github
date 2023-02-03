# µONOS SDN Platform

The intent of this project is to bootstrap a design and development effort to establish a new generation platform to replace ONOS-classic 
for SD-Fabric and similar SDN projects.

While ONOS-classic has served well as a platform, it is aging and should be replaced with a lighter-weight platform to a) enable better 
scaling and b) ensure long-term viability of the SD-Fabric solution.

This project will consist of a suite of components and libraries - following the established µONOS patterns – aimed to replace ONOS-classic 
for programming the data plane forwarding behaviors. The SD-Fabric use-case will be the proximate driving force guiding the decisions and 
priorities for this project and any associated design decisions. While some of the components will ultimately become usable in a more general 
context, this should not be the first-order priority.

## Scaling, Performance & Resilience
The platform architecture should scale to 1.1K devices (100 switches, 1K IPUs) and 20K hosts. The system should allow for some form of 
horizontal scaling to distribute the control I/O load and to reduce the “blast radius” of a component failure.

## Key Platform Components 
The following set of components can be classified as being part of a more general platform as they are not inherently specific to the SD-Fabric 
solution. All communicate via [gRPC APIs][onos-api] using TLS; option to bypass TLS will also be available. The components are available as Docker images and have corresponding Helm charts available under the [onos-helm-charts][onos-helm-charts] repository.

![Key µONOS Platform Components](https://raw.githubusercontent.com/onosproject/.github/master/profile/micro-onos-components.png)

### Persistent NIB ([onos-topo][onos-topo])
This subsystem offers persistence of network entities using an extensible and elastic (Entity/Relation/Kind) model 
via its NB gRPC API to other components and applications. As such, it is suitable for storing and retrieving 
information about major network assets, e.g. devices, links, controllers, hosts, virtual network entities.

The information, organized as a dynamic graph, can be searched, browsed, and monitored easily by any platform or app components.

### Device Provisioner Component ([device-provisioner][device-provisioner])
This component is primarily responsible for forwarding pipeline configuration/reconciliation using P4Runtime and device chassis configuration via gNMI.

It will expose NB gRPC API that allows management of different pipeline configuration artifact sets (p4info, architecture-specific binaries, etc.) 
and chassis configuration artifacts. Support for additional types of configuration blobs and upload protocols can be provided in the future, e.g. gNOI.

This API is split into two parts:
*	Service for managing the inventory of pipeline configurations (add/delete/get/etc.)
*	Service for monitoring the status of pipeline reconciliation on devices

Mapping of pipeline configurations to devices is to be done separately, via an aspect attached to the device entity in `onos-topo`.

### Topology Discovery Component ([topo-discovery][topo-discovery])
The discovery component is responsible for discovering devices (switches, IPUs) their connectivity via infrastructure links,
and connected end-stations (hosts). Discovery of links and hosts will occur indirectly through local agents to distribute the I/O load and to
help maintain performance at scale.

For the most part, the component will operate autonomously. However, it will also expose an auxiliary gRPC API to allow coarse-grained control
of the discovery mechanism, e.g., trigger all or trigger device port/link discovery.

### Data Plane Reconciliation Library ([onos-control][onos-control])
This piece of the µONOS architecture is provided as a library, rather than a separate component. The principal aim of the library is to provide
µONOS components and applications with common and uniform means to program the behavior of the data plane using P4Runtime constructs,
while remaining reasonably insulated from the details of a particular P4 program on the networking device, i.e. physical pipeline.

This component is under active development.

### Switch Local Discovery Agent ([link-agent][link-agent] - to be renamed) 
The local agent (LA) components are expected to be deployed on each data-plane device (switch or IPU). At the minimum, they are responsible for
local emissions and handling of ARP and LLDP packets for the purpose of host and infrastructure link discovery.

The LA will provide access to the discovered information/state via gNMI and a custom YANG model. Such mechanism should also be provided to allow
configuration of the agent host and link discovery behavior.

### Path Service Component (path-service)
The path service component provides path and spanning-tree computation services via a gRPC API. To accomplish this, the component interfaces
with onos-topo (NIB) to maintain an up-to-date in-memory weighted directed multigraph from the NIB link entities, on which it can perform 
efficient path computation. The API should support multiple path computation path algorithms, i.e. Dijkstra, Yen K-shortest paths, Bellman-Ford,
Kruskal, Johnson (but not necessarily all of these).   _This component has not yet been designed or developed._

### Consolidated CLI ([onos-cli][onos-cli])
All major components have command-line packages under the `onos-cli` repository, creating a consolidate CLI client - similar to what `kubectl` provides for Kubernetes.

### Fabric Simulator ([fabric-sim][fabric-sim])
Fabric Simulator provides simulation of a network of switches, IPUs and hosts via P4Runtime, gNMI and gNOI control interfaces. Controllers and applications can interact with the network devices using the above interfaces to learn about the network devices and the topology of the network. Note that, unlike `mininet`, the `fabric-sim` does not actually emulate data-plane traffic. It merely emulates control interactions and mimics certain behaviours of the data-plane. For example, if an LLDP packet-out is emitted by an application via P4Runtime, it will result in an LLDP packet-in received by the application from the neighboring device.

The simulator can be run as a single application or a docker container. This serves as a cost-effective means to stress-test controllers and applications at scale and a light-weight fixture for software engineers during development and testing of their applications.

### Miscellaneus
There are also various library modules such as `onos-lib-go` and `onos-net-lib`. These provide facilities used by various platform components and also applicable to custom SDN application components. 

_More documentation will be provided over time..._



[onos-api]: https://github.com/onosproject/onos-api
[onos-cli]: https://github.com/onosproject/onos-cli
[onos-topo]: https://github.com/onosproject/onos-topo
[onos-control]: https://github.com/onosproject/onos-control
[device-provisioner]: https://github.com/onosproject/device-provisioner
[topo-discovery]: https://github.com/onosproject/topo-discovery
[link-agent]: https://github.com/onosproject/link-agent
[fabric-sim]: https://github.com/onosproject/fabric-sim
[onos-helm-charts]: https://github.com/onosproject/onos-helm-charts

