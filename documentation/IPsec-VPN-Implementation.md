\# IPsec VPN Implementation



\## 1. Introduction



This lab was created in GNS3 to configure and test a site-to-site IPsec VPN between an HQ network and a Branch network.



The purpose was to allow devices on both LANs to communicate through an encrypted VPN tunnel.



HQ network:



`10.10.10.0/24`



Branch network:



`10.20.10.0/24`



\---



\## 2. Lab Topology



The topology contains:



\* PC-HQ

\* R1-HQ

\* ISP Router

\* R2-BRANCH

\* PC-BRANCH



Traffic path:



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



The ISP router is used only to simulate the network between the two sites.



\---



\## 3. IP Addressing



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



\---



\## 4. Routing



Static routes were used to make sure both routers could reach the remote LAN.



\### R1-HQ



```text

ip route 0.0.0.0 0.0.0.0 203.0.113.2

ip route 10.20.10.0 255.255.255.0 203.0.113.2

```



\### ISP



```text

ip route 10.10.10.0 255.255.255.0 203.0.113.1

ip route 10.20.10.0 255.255.255.0 198.51.100.2

```



\### R2-BRANCH



```text

ip route 0.0.0.0 0.0.0.0 198.51.100.1

ip route 10.10.10.0 255.255.255.0 198.51.100.1

```



\---



\## 5. NAT Configuration



NAT overload was configured on both routers.



VPN traffic was excluded from NAT because traffic between the two private networks needs to be encrypted and sent through the IPsec tunnel.



\### R1-HQ



```text

ip access-list extended NAT-HQ

&#x20;deny ip 10.10.10.0 0.0.0.255 10.20.10.0 0.0.0.255

&#x20;permit ip 10.10.10.0 0.0.0.255 any



ip nat inside source list NAT-HQ interface GigabitEthernet0/0 overload

```



\### R2-BRANCH



```text

ip access-list extended NAT-BRANCH

&#x20;deny ip 10.20.10.0 0.0.0.255 10.10.10.0 0.0.0.255

&#x20;permit ip 10.20.10.0 0.0.0.255 any



ip nat inside source list NAT-BRANCH interface GigabitEthernet0/0 overload

```



The first deny statement does not block the traffic. It simply tells NAT not to translate that traffic.



\---



\## 6. IKE Phase 1



IKE Phase 1 was configured on both routers.



Configuration used:



```text

crypto isakmp policy 10

&#x20;encr aes

&#x20;authentication pre-share

&#x20;group 5

```



A pre-shared key was configured between the two VPN routers.



```text

crypto isakmp key OmegaVPN2026 address <peer-ip>

```



R1-HQ uses:



```text

crypto isakmp key OmegaVPN2026 address 198.51.100.2

```



R2-BRANCH uses:



```text

crypto isakmp key OmegaVPN2026 address 203.0.113.1

```



\---



\## 7. IPsec Phase 2



The IPsec transform set was configured as:



```text

crypto ipsec transform-set VPN-SET esp-aes esp-sha-hmac

&#x20;mode tunnel

```



This provides encryption and authentication for the traffic going through the VPN tunnel.



\---



\## 8. VPN Traffic



An extended ACL was used to identify the traffic that should go through the VPN.



\### R1-HQ



```text

ip access-list extended VPN-TRAFFIC

&#x20;permit ip 10.10.10.0 0.0.0.255 10.20.10.0 0.0.0.255

```



\### R2-BRANCH



```text

ip access-list extended VPN-TRAFFIC

&#x20;permit ip 10.20.10.0 0.0.0.255 10.10.10.0 0.0.0.255

```



This means traffic between:



`10.10.10.0/24`



and



`10.20.10.0/24`



is considered interesting traffic for the VPN.



\---



\## 9. Crypto Map



The crypto map connects the VPN configuration together.



\### R1-HQ



```text

crypto map VPN-MAP 10 ipsec-isakmp

&#x20;set peer 198.51.100.2

&#x20;set transform-set VPN-SET

&#x20;match address VPN-TRAFFIC

```



The crypto map was applied to:



```text

interface GigabitEthernet0/0

&#x20;crypto map VPN-MAP

```



\### R2-BRANCH



```text

crypto map VPN-MAP 10 ipsec-isakmp

&#x20;set peer 203.0.113.1

&#x20;set transform-set VPN-SET

&#x20;match address VPN-TRAFFIC

```



The crypto map was applied to the WAN interface on R2-BRANCH.



\---



\## 10. VPN Verification



The following commands were used to check the VPN:



```text

show crypto isakmp sa

```



The tunnel showed:



```text

QM\_IDLE

```



This confirmed that the IKE security association was established.



The IPsec tunnel was checked using:



```text

show crypto ipsec sa

```



The packet counters increased after traffic was sent between the two LANs.



The counters showed encrypted and decrypted packets, confirming that traffic was passing through IPsec.



\---



\## 11. Final Ping Test



The final test was performed from PC-HQ.



```text

PC-HQ> ping 10.20.10.10

```



Result:



```text

5 packets transmitted

5 packets received

0% packet loss

```



This confirmed communication between the HQ and Branch LANs through the VPN.



\---



\## 12. Troubleshooting



The VPN did not come up during the first test.



After checking the configuration, I found that the peer IP address used in the pre-shared-key configuration on R1-HQ was incorrect.



The incorrect address was:



```text

192.51.100.2

```



The correct address was:



```text

198.51.100.2

```



After correcting the address, the IKE tunnel came up and the IPsec tunnel started passing traffic.



The final status showed:



```text

QM\_IDLE

```



and the ping between the two LANs was successful.



\---



\## 13. Verification Results



| Test                        | Result |

| --------------------------- | ------ |

| HQ gateway reachable        | PASS   |

| Branch gateway reachable    | PASS   |

| IKE tunnel                  | PASS   |

| IPsec tunnel                | PASS   |

| IPsec encryption/decryption | PASS   |

| HQ to Branch ping           | PASS   |

| Packet loss                 | 0%     |



\---



\## 14. What I Learned



This project helped me practice:



\* Site-to-site IPsec VPN

\* IKE Phase 1

\* IPsec Phase 2

\* Pre-shared key authentication

\* Crypto ACLs

\* Crypto maps

\* NAT exemption

\* Static routing

\* VPN troubleshooting

\* Cisco VPN verification commands



\---



\## 15. Project Result



The site-to-site IPsec VPN was successfully configured and tested in GNS3.



PC-HQ was able to communicate with PC-BRANCH across the simulated ISP network, and the IPsec tunnel successfully encrypted the traffic between the two LANs.



All router configurations were saved after testing.



