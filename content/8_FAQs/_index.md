---
title: "FAQs"
menuTitle: "FAQs"
weight: 80
---


  - **Can you terminate the GENEVE tunnels on a different ENI like ENI1\port2?**

Yes.  This can be done but would require you to use an IP based target group instead of an Instance based target group.  This would not work for the official FortiGate Auto Scale solution so this would be limited to a manual scale deployment.

  - **Can you send new flows to private resources via the GENEVE tunnels?**

No.  Traffic must come through a GWLBe endpoint so that GWLB knows where to send the reply traffic for each flow.  For example, this means that traffic generated by the FGT itself will need to go out port1 (to public resources) or port2 (to private resources).  To reach private resources, create static routes on the FGTs with the AWS VPC router for the connected private subnet to reach private resources (within the same VPC or via TGW).  The VPC router is the first host IP of each AWS subnet, (ie 10.1.0.1 host IP for 10.1.0.0/24 subnet).