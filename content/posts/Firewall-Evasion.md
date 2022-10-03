---
title: "Nmap Firewall Evasion"
date: 2022-10-02T20:32:05+02:00
draft: true
---


### SYN-Scan
[[Port Scanning#TCP SYN Scan]]

--- 

### ACK-Scan
[[Port Scanning#TCP ACK scan]]

used to test firewalls rules only to check if the port is filtered or unfiltered
-  the `ACK` flag are often passed by the firewall because the firewall cannot determine whether the connection was first established from the external network or the internal network.

---  

### Decoys
[[Port Scanning#Decoys]]
- the purpose of using Decoys is to add **noise** to the IDS by sending scans from spoofed ip addresses along with the **real attacker ip** and that confuses the analyst watching 

- A **Decoy** attack requires the following:
	1. All decoys are up and running (otherwise it’s easy to determine the real attacker’s IP),
	2. The real IP address should appear in random order to the IDS (otherwise it is easy to infer the real attacker’s IP),
	3. ISP’s traversed by spoofed traffic let the traffic go through

```bash
	sudo nmap –sS –D [DecoyIP_1],[DecoyIP_2],[DecoyIP_3],ME [target]
```
   - `ME` is used to define the position of our real ip address among other decoys
   - if you don't specify ME , Nmap will put your ip in a random position

*note* you can not use the Decoy attack with `sT` and `sV`  ( beacuse they use full connect scan  )

##### Decoys from hping3
- using random source ip addresses 
```bash
	sudo hping3 --rand--source -S -p 80 192.168.1.4
```
- using live hosts in the network
```bash
	sudi hping3 -a [spoffed_ip] -S -p 80 192.168.1.4
```

---  
### Source Port
it can be used to abuse poorly configured firewalls that allow traffice coming from certain ports
like some firwalls allow traffice from port **53** (DNS) or port **20** (FTP)
- `--source-port [port]`
- `-g [port]`

---

### Fragmentation
- [[Port Scanning#Fragmented Packets]]

the conept of fragmentation is to split a single packet into smaller ones , so that can disable the ability of  some firwall/IDS to apply their packet filtering rules

they may inspect single fragment but not the whole packet

**notice**
instead of using `-f` ,we can use `--mtu` to specify a custom offset size (offset must be  a multiple of eight)


--- 

### Timing
[[Port options and Rate#Rate and Performance options]]
the **timing attack** does not **modify** the packets but it just slow down the scan in order to blend with other traffic in the logs
- `-T[0~5]` set timing template
    these table show the differences  between time options and the delay time between packets
	![[Nmap_time_Template.png]]

---- 

### DNS Proxying
By using [[#Source Port]] technique
we can evasion the firewall by using dns port as our source port so the IDS/IPS trust it

```bash
	sudo nmap 192.168.1.1 -p50000 -sS -Pn -n --source-port 53
```
- `-Pn`  disable ping (skip host discovery)
- `-n` disable dns resolve
- `--source-port <port>`

##### using ncat with DNS proxy
```bash
	ncat -nv --source-port 53 10.129.2.28 50000
```

---- 
### Append random data in the TCP header
```bash
	sudo nmap 192.168.1.4 --data-length 10
```
`--data-length 10` that will append random 10 bytes data to the tcp header

- from hping3
	`--data [data_size]`
	add fixed data (xxxxxxxx)

----


### Mac Spoofing
hepls us if the firewall block our mac or  accepts only packets sepcific mac
```bash
--spoof-mac <option/mac>
```
nmap has three types of mac spoofing
1. use the mac from popular vendors
	```bash
		--spoof-mac apple
	```
2. use a complete **random** **mac**  
	```bash
	--spoof-mac 0
	```
3. Manually enter the mac
	```bash
	--spoof-mac  00:11:22:33:44:55
	```

--- 
### Randomize  scan hosts in a  range 
```bash
	--randomize-hosts
```
for more evasion techniques [Nmap_Manual For Evasion](https://nmap.org/book/man-bypass-firewalls-ids.html)
