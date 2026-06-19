# Educational ARP Poisoning Lab

This repository documents my educational lab demonstrating an ARP Poisoning (Man-in-the-Middle) attack. I conducted this lab in a controlled, isolated virtual environment to understand network vulnerabilities and packet interception.

## Lab Environment
* **Attacker Machine:** Kali Linux (`10.0.2.3`)
* **Target Machine:** Windows 7 (`10.0.2.4`)
* **Default Gateway:** `10.0.2.1`
* **Tools Used:** `ifconfig`, `ip route`, `sysctl`, `arpspoof`, Ettercap, Wireshark.

---

## Phase 1: CLI Method (arpspoof)

### 1. Network Reconnaissance
First, I verified the IP configurations of my machines to map out the local network.
* I checked my target's IP address (Windows 7), which was `10.0.2.4`.

![Windows 7 IP Configuration](Phase-1/win7_ip.png)

* I checked my Kali Linux IP address, which was `10.0.2.3` on the `eth0` interface.

![Kali Linux IP Configuration](Phase-1/kali_ip.png)

* I identified the network's default gateway as `10.0.2.1`.

![Default Gateway](Phase-1/gateway_default.png)

### 2. Enabling IP Forwarding
To ensure my target machine maintained internet access during the attack (allowing me to act as a seamless router rather than causing a Denial of Service), I enabled IPv4 packet forwarding on my Kali machine:

```bash
sudo sysctl -w net.ipv4.ip_forward=1
```

![IP Forwarding](Phase-1/packet-forwarding.png)

### 3. Launching the ARP Spoofing Attack
I used `arpspoof` to manipulate the ARP tables of both the target and the gateway. I opened two separate terminal instances on my Kali machine to send forged ARP replies continuously.

In the first terminal, I tricked the target into believing my Kali machine was the gateway:

```bash
sudo arpspoof -i eth0 -t 10.0.2.4 10.0.2.1
```

In the second terminal, I tricked the gateway into believing my Kali machine was the target:

```bash
sudo arpspoof -i eth0 -t 10.0.2.1 10.0.2.4
```

Wireshark confirmed the broadcast of these malicious ARP packets, showing my Kali MAC address claiming the IP addresses of both the gateway and the target.

![ARP Poisoning](Phase-1/arp-poisoning.png)

### 4. Traffic Generation & Interception
* On my Windows 7 target machine, I opened a web browser and navigated to YouTube to generate standard HTTPS traffic.

![Windows 7 YouTube](Phase-1/youtube_win7-browser.png)

* On my Kali machine, I ran Wireshark and captured packets on the `eth0` interface.
* Because I successfully poisoned the ARP caches, all outbound and inbound traffic for the target routed directly through my Kali machine. By filtering for `tcp.port == 443`, I successfully intercepted and analyzed the TCP and TLSv1.3 application data flowing between the target (`10.0.2.4`) and external internet servers.

![Wireshark Capture](Phase-1/win7-kali-connection.png)

---

## Phase 2: GUI Method (Ettercap)

### 1. Host Scanning
As an alternative to the command-line tools, I utilized Ettercap's graphical interface. I launched Ettercap, initiated unified sniffing on `eth0`, and scanned the local subnet to build a list of active hosts.

![Ettercap Host Scan](Phase-2/ettercap-menu.png)

### 2. Setting Targets
Once the scan completed, I opened the Host List. I identified my Windows 7 target (`10.0.2.4`) and the default gateway (`10.0.2.1`). I assigned `10.0.2.4` as **Target 1** and `10.0.2.1` as **Target 2** to prepare for the MitM attack.

![Ettercap Target Selection](Phase-2/target_arp_pois.png)

### 3. Launching ARP Poisoning
I activated the ARP poisoning plugin within Ettercap. To verify the attack was successful, I opened Wireshark and observed the continuous stream of forged ARP replies being sent to the network, confirming my Kali machine was successfully intercepting the route.

![Wireshark ARP Verification](Phase-2/wireshark1.png)

### 4. Traffic Interception
With the Ettercap MitM attack running seamlessly, I monitored the `eth0` interface in Wireshark. By applying a filter for `tcp.port == 443`, I was able to capture the target's web traffic, confirming that TCP packets originating from `10.0.2.4` were successfully routing through my machine.

![Wireshark TCP Capture](Phase-2/w2.png)

---
**Disclaimer:** This lab was performed strictly for educational purposes within a local, authorized virtual environment to study network security principles.
