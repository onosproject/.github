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
solution. All communicate via gRPC APIs using TLS; option to bypass TLS will also be available.

### Persistent NIB (onos-topo)
Functionality of the NIB is expected to be provided by the existing `onos-topo` subsystem. It offers extensible and elastic 
(Entity/Relation/Kind) model via its NB gRPC API to other components and applications. As such, it is suitable for storing and retrieving 
information about major network assets, e.g. devices, links, controllers, hosts, virtual network entities.

The information, organized as a dynamic graph, can be searched, browsed, and monitored easily by any platform or app components.

### Device Provisioner Component (device-provisioner)
This component is primarily responsible for forwarding pipeline configuration/reconciliation using P4Runtime and device chassis configuration via gNMI.

It will expose NB gRPC API that allows management of different pipeline configuration artifact sets (p4info, architecture-specific binaries, etc.) 
and chassis configuration artifacts. Support for additional types of configuration blobs and upload protocols can be provided in the future, e.g. gNOI.

This API is split into two parts:
*	Service for managing the inventory of pipeline configurations (add/delete/get/etc.)
*	Service for monitoring the status of pipeline reconciliation on devices

Mapping of pipeline configurations to devices is to be done separately, via an aspect attached to the device entity in `onos-topo`.

### Topology Discovery Component (topo-discovery)
The discovery component is responsible for discovering devices (switches, IPUs) their connectivity via infrastructure links,
and connected end-stations (hosts). Discovery of links and hosts will occur indirectly through local agents to distribute the I/O load and to
help maintain performance at scale.

For the most part, the component will operate autonomously. However, it will also expose an auxiliary gRPC API to allow coarse-grained control
of the discovery mechanism, e.g., trigger all or trigger device port/link discovery.

### Path Service Component (path-service)
The path service component provides path and spanning-tree computation services via a gRPC API. To accomplish this, the component interfaces
with onos-topo (NIB) to maintain an up-to-date in-memory weighted directed multigraph from the NIB link entities, on which it can perform 
efficient path computation. The API should support multiple path computation path algorithms, i.e. Dijkstra, Yen K-shortest paths, Bellman-Ford,
Kruskal, Johnson (but not necessarily all of these).   _This component has not yet been designed or developed._

### Switch Local Discovery Agent (link-agent - to be renamed) 
The local agent (LA) components are expected to be deployed on each data-plane device (switch or IPU). At the minimum, they are responsible for
local emissions and handling of ARP and LLDP packets for the purpose of host and infrastructure link discovery.

The LA will provide access to the discovered information/state via gNMI and a custom YANG model. Such mechanism should also be provided to allow
configuration of the agent host and link discovery behavior.

### Data Plane Reconciliation Library (onos-control)
This piece of the µONOS architecture is provided as a library, rather than a separate component. The principal aim of the library is to provide
µONOS components and applications with common and uniform means to program the behavior of the data plane using P4Runtime constructs,
while remaining reasonably insulated from the details of a particular P4 program on the networking device, i.e. physical pipeline.


_More documentation will be provided over time..._
