# Leitstand in a Nutshell

Leitstand aims to create a disaggregated management system for disaggregated access networks by means of providing the foundation to model the resources forming the network and an environment to implement management applications supplying the actual network management functions.

![Leitstand Overview](./doc/assets/leitstand_overview.png "Leitstand Overview") 

Leitstand relies on _service-oriented architecture_ , 
loose coupling through [REST APIs](./doc/REST.md) and 
service encapsulation in docker containers in order to support a distributed deployment.
The Leitstand architecture is also influenced by Eric Evans domain-driven design, in the sense of putting the focus on the network management domain and leveraging open-source software for general management aspects like log management, telemetry streaming, or time series visualization. 
Connectors are specialized services to connect Leitstand with third-party products.

The [Leitstand Web User Interface](../leitstand-ui/README.md) relies on the REST APIs and allows to manage the resource inventory and to use the Leitstand applications.

## Repositories

### Leitstand Platform

- The [leitstand-inventory](../leitstand-inventory/README.md) repository contains the Leitstand Resource Inventory to store all elements of a disaggregated access network.
- The [leitstand-job](../leitstand-job/README.md) repository contains the Leitstand Job Scheduler to execute management jobs. 
- The [leitstand-events](../leitstand-events/README.md) repository contains transactual domain event support to subscribe for resource inventory or job state changes.
- The [leitstand-security](../leitstand-security/README.md) repository contains a built-in user repository, access key support for inter-system authentication, encryption utilities, and OAuth 2.0 support.
- The [leitstand-ui](../leitstand-ui/README.md) repository contains the Leitstand UI framework and cross-cutting UI functions (e.g. login/logut).

### Leitstand Connectors

- The [leitstand-powerdns](../leitstand-powerdns/README.md) connector forwards DNS record changes to PowerDNS. PowerDNS can either be used as DNS server or as gateway to manage DNS records in the actual DNS server.

### Leitstand Applications

The following Leitstand Applications are in the pipeline for being open-sourced and will be available early in Q1/20.

- __Log Application__, the log application connects Leitstand with elasticsearch and supplies conext-sensitive log searches by means of generating queries from inventory records. This simplifies the search for log messages for a certain element or group of elements.
- __Telemetry Application__, the telementry application allows to configure metrics that can be sampled from the elements including their visualization and monitoring. It also includes Leitstand Connectors for Prometheus and Grafana.
- __Topology Application__, the topology application visualizes the link-state graph of a disaggregated  network.

