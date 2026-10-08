# Project 4: Site-to-Site IPsec VPN

## About
This project demonstrates a site-to-site IPsec VPN lab environment simulated in GNS3. The objective was to establish a secure, encrypted tunnel between two distinct local area networks to enable cross-site communication.

**Network Details:**
* **HQ LAN:** `10.10.10.0/24`
* **Branch LAN:** `10.20.10.0/24`

Verification was successfully performed by testing connectivity between the HQ PC and the Branch PC.

## Lab Topology
The lab infrastructure consists of the following components:
* **R1-HQ:** Head office gateway router
* **ISP Router:** Simulates the public internet backbone
* **R2-BRANCH:** Branch office gateway router
* **PC-HQ:** Client machine in the HQ network
* **PC-BRANCH:** Client machine in the branch network
