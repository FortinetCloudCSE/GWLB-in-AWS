---
title: "Overview"
menuTitle: "Overview"
weight: 20
---

FortiOS supports integrating with AWS Gateway Load Balancer (GWLB) starting in version 6.4.4 GA and newer versions.  GWLB makes it easier to inspect traffic with a fleet of active-active FortiGates.  GWLB track flows and sends traffic for a single flow to the same FortiGate so there is no need to apply source NAT to have symmetrical routing.  Also with the use of VPC Ingress Routing and GWLB endpoints (GWLBe), you can easily use VPC routes to granularly direct traffic to your GWLB and FortiGates for transparent traffic inspection.

This solution works with a fleet of FortiGates deployed in multiple availability zones.  The FortiGates can be a set of independent FortiGates or even a clustered auto scale group.

The main benefits of this solution are:
  - Active-Active scale out design for traffic inspection
  - Symmetrical routing of traffic without the need for source NAT
  - VPC routing is easily used to direct traffic for inspection
  - Support for cross zone load balancing and failover

**Note:**  Other Fortinet solutions for AWS such as FGCP HA (Dual or Single AZ) and Auto Scaling are available.  Please visit [**www.fortinet.com/aws**](https://www.fortinet.com/aws) for further information.