# Azure Network Traffic Analysis

I built a small network environment in Azure and used Wireshark to watch live traffic across several protocols. The goal was to get past theory and actually see what these protocols look like when they're running.

---

## Environment

- Microsoft Azure (Resource Groups, Virtual Networks, VMs, Network Security Groups)
- Windows 10 Pro VM and Ubuntu 22.04 VM on the same VNet/subnet
- Wireshark (packet capture and protocol analysis)
- PowerShell and Command Prompt (within Windows VM)
- Remote Desktop / Windows App for VM access

---

## Setup

I deployed two VMs in Azure (one Windows 10, one Ubuntu) and made sure both were on the same virtual network and subnet. Connected into the Windows VM via RDP, installed Wireshark, and used that as my base for every lab that followed.

---

## Labs

### ICMP: Ping and Firewall Testing

I filtered Wireshark for ICMP and pinged the Ubuntu VM's private IP from the Windows VM. Watched the request/reply pairs show up in the capture. Then pinged google.com to see external ICMP traffic alongside internal.

![ICMP request and reply packets in Wireshark alongside PowerShell ping output](Step%203e%20-%20Ping%20ICMP.png)

I also set up a continuous ping from the Windows VM to Ubuntu, went into the Ubuntu VM's Network Security Group in Azure, and blocked inbound ICMP. The replies stopped immediately, visible in both Wireshark and the command line at the same time.

![Continuous ping running before NSG rule is applied](Step%204a%20-%20Network%20Security%20Group.png)

![Wireshark showing only requests and PowerShell showing Request timed out after NSG blocks ICMP](Step%204e%20-%20Network%20Security%20Group.png)

Re-enabled the rule and traffic came back.

![Traffic resuming in Wireshark and PowerShell after re-enabling the NSG rule](Step%204g%20-%20Network%20Security%20Group.png)

---

### SSH: Remote Session Traffic

I filtered for SSH and connected into the Ubuntu VM from PowerShell using its private IP. Every command I typed produced a burst of packets in Wireshark, but none of the content was readable. Just encrypted traffic confirming the session was active.

![Wireshark showing SSH encrypted packets alongside an active Linux session in PowerShell](Step%205b%20-%20SSH.png)

---

### DHCP: IP Address Renewal

I filtered for DHCP and ran `ipconfig /renew` from an elevated PowerShell prompt. Watched the discover/offer/request/acknowledge exchange show up in the capture in sequence.

![Wireshark showing the full DHCP Release, Discover, Offer, Request, and ACK sequence](Step%206c-%20DHCP.png)

---

### DNS: Name Resolution

I filtered for DNS and used `nslookup` to query google.com and disney.com. Watched the query go out and the response come back with IP addresses. Worth noting that the queries are plaintext, so even when the connection that follows is encrypted, the lookup is still visible in the capture.

![Wireshark showing DNS query and response alongside nslookup output in PowerShell](Step%207a-%20DNS.png)

---

### RDP: Continuous Stream

I filtered for RDP traffic on tcp.port 3389. Every other protocol generated traffic in response to something: a ping, a command, a query. RDP just ran constantly with no input from me, because it streams a live image of the remote desktop the entire time the session is open.

![Wireshark showing constant RDP traffic stream on tcp.port 3389](Step%208a%20-%20RDP.png)

---

## Key Takeaways

A few things that landed differently after doing this versus just reading about it:

- **NSG rules are live.** Blocking ICMP mid-ping and watching the traffic stop in Wireshark made cloud firewall behavior feel real in a way a diagram never did.
- **Encrypted does not mean invisible.** SSH hides the content but not the activity. DNS queries are plaintext and reveal destination intent even when the connection itself is locked down.
- **Protocols have personalities.** ICMP is clean request/reply. DHCP is a four-step handshake. RDP is a constant stream. Knowing what normal looks like for each one is what lets you spot when something is wrong.
