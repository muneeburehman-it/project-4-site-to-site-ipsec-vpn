\# Project 4 - Site-to-Site IPsec VPN



\## About



This project is a site-to-site IPsec VPN lab created in GNS3.



The main goal was to connect two different LAN networks through an encrypted VPN tunnel.



HQ LAN: `10.10.10.0/24`



Branch LAN: `10.20.10.0/24`



The VPN was tested by sending traffic from the HQ PC to the Branch PC.



\## Lab Topology



The lab contains:



\* R1-HQ

\* ISP Router

\* R2-BRANCH

\* PC-HQ

\* PC-BRANCH



Connection:



```text

PC-HQ

&#x20; |

R1-HQ

&#x20; |

&#x20;ISP

&#x20; |

R2-BRANCH

&#x20; |

PC-BRANCH

```



\## IP Addressing



| Device    | Interface | IP Address      |

| --------- | --------- | --------------- |

| PC-HQ     | Ethernet  | 10.10.10.10/24  |

| R1-HQ     | G0/1      | 10.10.10.1/24   |

| R1-HQ     | G0/0      | 203.0.113.1/30  |

| ISP       | G0/0      | 203.0.113.2/30  |

| ISP       | G0/1      | 198.51.100.1/30 |

| R2-BRANCH | G0/0      | 198.51.100.2/30 |

| R2-BRANCH | G0/1      | 10.20.10.1/24   |

| PC-BRANCH | Ethernet  | 10.20.10.10/24  |



\## VPN Configuration



The VPN uses:



\* IKE Phase 1

\* Pre-shared key authentication

\* AES encryption

\* SHA hashing

\* DH Group 5

\* IPsec ESP

\* AES encryption

\* SHA-HMAC authentication



The pre-shared key used in the lab was:



`OmegaVPN2026`



The VPN interesting traffic was:



```text

10.10.10.0/24 <----> 10.20.10.0/24

```



\## NAT



NAT overload was configured on both edge routers.



VPN traffic was excluded from NAT so that traffic between the two LANs could use the IPsec tunnel.



Example:



```text

deny ip 10.10.10.0 0.0.0.255 10.20.10.0 0.0.0.255

permit ip 10.10.10.0 0.0.0.255 any

```



The same idea was used on the Branch router.



\## Verification



After configuring the VPN, I checked the tunnel using:



```text

show crypto isakmp sa

show crypto ipsec sa

```



The IKE status showed:



```text

QM\_IDLE

```



IPsec counters also increased when traffic was sent through the tunnel.



The final test was:



```text

PC-HQ> ping 10.20.10.10

```



Result:



```text

5 packets transmitted

5 packets received

0% packet loss

```



This confirmed that traffic between the HQ and Branch LANs was passing through the VPN.



\## Troubleshooting



The VPN did not come up on the first attempt.



The problem was a wrong peer IP address in the pre-shared-key configuration on R1.



I originally used:



```text

192.51.100.2

```



The correct address was:



```text

198.51.100.2

```



After correcting the address, the IKE tunnel came up and the IPsec tunnel started passing traffic.



\## What I Learned



During this lab I practiced:



\* Site-to-site IPsec VPN configuration

\* IKE Phase 1

\* IPsec Phase 2

\* Pre-shared keys

\* Crypto ACLs

\* Crypto maps

\* NAT exemption

\* Static routing

\* VPN troubleshooting

\* Verifying IPsec tunnels using Cisco commands



\## Tools



\* GNS3

\* Cisco IOS routers

\* VPCS



\## Project Status



Completed and tested successfully.



The configuration was saved on all devices after testing.



