---
title: "Build It Yourself"
weight: 4
---

**Expand each section to see the details.**

{{% expand title="**Full Configuration for One-Arm (Distributed)**" %}}

```
### allows dataplane and management enis to be in same subnets
config system settings
set allow-subnet-overlap enable
end

### used as health check for gwlb target group
config system probe-response
set mode http-probe
set port 8008
set http-probe-value OK
end

### port1 is dataplane and terminates geneve tunnels, port2 is dedicated to management
config system interface
edit port1
set vdom root
set alias 1-arm
set mode dhcp
set allowaccess ping probe-response
set type physical
set mtu-override enable
set mtu 9001
next
edit port2
set vdom root
set alias dedicated-mgmt
set mode dhcp
set defaultgw enable
set allowaccess ping https fgfm
set vrf 1
set dedicated-to management
set type physical
set mtu-override enable
set mtu 9001
next
end

### geneve tunnels to each gwlb eni from port1
config system geneve
edit gwlb1-az1
set interface port1
set type ppp
set remote-ip ${gwlb_ip1}
next
edit gwlb1-az2
set interface port1
set type ppp
set remote-ip ${gwlb_ip2}
next
end

### zone used to simplify fw policy
config system zone
edit gwlb1-tunnels
set interface "gwlb1-az1" "gwlb1-az2"
next
end

### static routes to get to gwlb eni for geneve and RPF check route to allow dataplane traffic
config router static
edit 1
set dst ${gwlb_ip1}/32
set device port1
set dynamic-gateway enable
set comment 'host route to reach GWLB eni for geneve tunnel in AZ1'
next
edit 2
set dst ${gwlb_ip2}/32
set device port1
set dynamic-gateway enable
set comment 'host route to reach GWLB eni for geneve tunnel in AZ2'
next
edit 3
set distance 5
set priority 100
set device gwlb1-az1
set comment 'only used for RPF check and not for routing'
next
edit 4
set distance 5
set priority 100
set device gwlb1-az2
set comment 'only used for RPF check and not for routing'
next
end

### policy routes to hairpin traffic back out the same geneve tunnel
config router policy
edit 1
set input-device gwlb1-az1
set output-device gwlb1-az1
set comment '1-arm mode hairpins all traffic'
next
edit 2
set input-device gwlb1-az2
set output-device gwlb1-az2
set comment '1-arm mode hairpins all traffic'
next
end

### simple fw policy to hairpin traffic
config firewall policy
edit 1
set name "1-arm-hairpin"
set srcintf "gwlb1-tunnels"
set dstintf "gwlb1-tunnels"
set srcaddr "all"
set dstaddr "all"
set action accept
set schedule "always"
set service "ALL"
set logtraffic all
next
end

### sdn connector with alt-resource-ip enabled to allow advanced filtering
config system sdn-connector
edit aws-instance-role
set status enable
set type aws
set use-metadata-iam enable
set alt-resource-ip enable
next
end
```

{{% /expand %}}

{{% expand title="**Full Configuration for Two-Arm Model (Centralized)**" %}}


```
### allows dataplane and management enis to be in same subnets
config system settings
set allow-subnet-overlap enable
end

### used as health check for gwlb target group
config system probe-response
set mode http-probe
set port 8008
set http-probe-value OK
end

### port1 and port2 are dataplane and terminates geneve tunnels on port2, port3 is dedicated to management
config system interface
edit port1
set vdom root
set alias 2-arm-public
set mode dhcp
set allowaccess ping
set type physical
set mtu-override enable
set mtu 9001
next
edit port2
set vdom root
set alias 2-arm-private
set mode dhcp
set defaultgw disable
set allowaccess ping probe-response
set type physical
set mtu-override enable
set mtu 9001
next
edit port3
set vdom root
set alias dedicated-mgmt
set mode dhcp
set defaultgw enable
set allowaccess ping https fgfm
set vrf 1
set dedicated-to management
set type physical
set mtu-override enable
set mtu 9001
next
end

### geneve tunnels to each gwlb eni from port2
config system geneve
edit gwlb1-az1
set interface port2
set type ppp
set remote-ip ${gwlb_ip1}
next
edit gwlb1-az2
set interface port2
set type ppp
set remote-ip ${gwlb_ip2}
next
end

### zone used to simplify fw policy
config system zone
edit gwlb1-tunnels
set interface "gwlb1-az1" "gwlb1-az2"
next
end

### static routes to get to gwlb eni via port2 for geneve and RPF check route to allow dataplane traffic
config router static
edit 1
set dst ${gwlb_ip1}/32
set device port2
set dynamic-gateway enable
set comment 'host route to reach GWLB eni for geneve tunnel in AZ1'
next
edit 2
set dst ${gwlb_ip2}/32
set device port2
set dynamic-gateway enable
set comment 'host route to reach GWLB eni for geneve tunnel in AZ2'
next
edit 3
set distance 5
set priority 100
set device gwlb1-az1
set comment 'only used for RPF check and not for routing'
next
edit 4
set distance 5
set priority 100
set device gwlb1-az2
set comment 'only used for RPF check and not for routing'
next
end

### policy routes to hairpin traffic back out the same geneve tunnel but bypassed when routing to public IPs
config router policy
edit 1
set input-device gwlb1-az1
set dst "10.0.0.0/255.0.0.0" "172.16.0.0/255.240.0.0" "192.168.0.0/255.255.0.0"
set output-device gwlb1-az1
set comment '2-arm mode hairpins traffic to RFC1918 CIDR, otherwise skips policy route for public CIDRs'
next
edit 2
set input-device gwlb1-az2
set dst "10.0.0.0/255.0.0.0" "172.16.0.0/255.240.0.0" "192.168.0.0/255.255.0.0"
set output-device gwlb1-az2
set comment '2-arm mode hairpins traffic to RFC1918 CIDR, otherwise skips policy route for public CIDRs'
next
end

### simple fw policy to snat internet bound traffic out port1 and hairpin rfc1918 traffic
config firewall policy
edit 1
set name "2-arm-egress"
set srcintf "gwlb1-tunnels"
set dstintf "port1"
set srcaddr "rfc-1918-subnets"
set dstaddr "all"
set action accept
set schedule "always"
set service "ALL"
set logtraffic all
set nat enable
next
edit 2
set name "2-arm-hairpin"
set srcintf "gwlb1-tunnels"
set dstintf "gwlb1-tunnels"
set srcaddr "all"
set dstaddr "all"
set action accept
set schedule "always"
set service "ALL"
set logtraffic all
next
end

### sdn connector with alt-resource-ip enabled to allow advanced filtering
config system sdn-connector
edit aws-instance-role
set status enable
set type aws
set use-metadata-iam enable
set alt-resource-ip enable
next
end
```

{{% /expand %}}