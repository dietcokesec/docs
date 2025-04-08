# Nmap
## Port States

| State            | Description                                                                                                                                                                                           |
| ---------------- | ----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| open             | This indicates that the connection to the scanned port has been established. These connections can be TCP connections, UDP datagrams as well as SCTP associations.                                    |
| closed           | When the port is shown as closed, the TCP protocol indicates that the packet we received back contains an RST flag. This scanning method can also be used to determine if our target is alive or not. |
| filtered         | Nmap cannot correctly identify whether the scanned port is open or closed because either no response is returned from the target for the port or we get an error code from the target.                |
| unfiltered       | This state of a port only occurs during the TCP-ACK scan and means that the port is accessible, but it cannot be determined whether it is open or closed.                                             |
| open\|filtered   | If we do not get a response for a specific port, Nmap will set it to that state. This indicates that a firewall or packet filter may protect the port.                                                |
| closed\|filtered | This state only occurs in the IP ID idle scans and indicates that it was impossible to determine if the scanned port is closed or filtered by a firewall.                                             |
[source](https://academy.hackthebox.com/module/19/section/102)

## Scanning
### Timings and Speedups
If we use `-p-` it'll check literally every port. Speedups are: `-F` top 100, and `--top-ports=10` for top 10. You can use a timing modifier like `-T4` or `-T5` to get results more quickly. For long scans, it is recommended to use `--stats-every=5s` to get consistent updates. You can also press `<space>` to get it to print if you're in an interactive session.

### Finding All TCP Ports
#### Running without Root
```bash
nmap -sT -p- -T4 <host>
```

#### Running with Root
SYN scans require root.
```bash
sudo nmap -sS -p- -T4 <host>
```

#### Evasive TCP Scans
Both TCP-Handshake (`-sT`) and SYN (`-sS`) scans require initiating a connection into the server. This is typically easy to classify and block as most admins don't want random TCP connections being initiated to ports besides the usual suspects (`80`, `443`, and sometimes `x000` for things like an express app, which every tutorial puts on port `3000`).

We can use an `ACK` scan to try and circumvent this since it would be hard for a firewall or IDS to know if it originated from itself or someone else (i.e. an attacker).
```bash
sudo nmap -sA -p- -T4 <host>
```

### Finding All UDP Ports
You'll almost always want to restrict the number of ports, or have an educated guess of the list. Otherwise this will take an eternity.
```bash
sudo nmap -sU -F -T4 <host>
```

### Detecting Services
#### Version Scans
Nmap has built in service detection, this is useful for finding out versions etc using Nmap's payload database. The easiest and fastest followup after getting the list of open ports is to banner grab all of them. Note this can take awhile. It's always recommended to give a static port list from a fast scan if possible.
```bash
sudo nmap -sV -p- -T4 <host>
```
Recall also that Nmap does not always hang around, so services that reply more slowly, like SMTP, may not immediately give up their banner and other details. We can use `tcpdump` and `nc` to capture this.
```bash
sudo tcpdump -i <interface> host <your_ip_on_interface> and <host>
```
This command is telling tcpdump to:
- Capture traffic on interface <interface>
- Only show packets where either the source or destination IP is both:
- `<your_ip_on_interface>` and
- `<host>`

To confirm:
```bash
ip addr show eth0
```
Look for a line like:
```bash
inet <your_ip_on_interface> ...
```

From here, you rig up nc as
```bash
nc -nv <host> <port>
```
And tcpdump will dump the relevant information, which we can use to see if our requests went through successfully. This is useful to reverse engineer the running service.

#### Scripting Engine
The scripting engine, typically called Nmap Scripting Engine, or `NSE` lets us make and use scripts in Nmap which do purpose specific functionality.

| Category  | Description                                                                                                                             |
| --------- | --------------------------------------------------------------------------------------------------------------------------------------- |
| auth      | Determination of authentication credentials.                                                                                            |
| broadcast | Scripts, which are used for host discovery by broadcasting and the discovered hosts, can be automatically added to the remaining scans. |
| brute     | Executes scripts that try to log in to the respective service by brute-forcing with credentials.                                        |
| default   | Default scripts executed by using the -sC option.                                                                                       |
| discovery | Evaluation of accessible services.                                                                                                      |
| dos       | These scripts are used to check services for denial of service vulnerabilities and are used less as it harms the services.              |
| exploit   | This category of scripts tries to exploit known vulnerabilities for the scanned port.                                                   |
| external  | Scripts that use external services for further processing.                                                                              |
| fuzzer    | This uses scripts to identify vulnerabilities and unexpected packet handling by sending different fields, which can take much time.     |
| intrusive | Intrusive scripts that could negatively affect the target system.                                                                       |
| malware   | Checks if some malware infects the target system.                                                                                       |
| safe      | Defensive scripts that do not perform intrusive and destructive access.                                                                 |
| version   | Extension for service detection.                                                                                                        |
| vuln      | Identification of specific vulnerabilities.                                                                                             |
[source](https://academy.hackthebox.com/module/19/section/108)

We can run the default category from `NSE` with the following:
```bash
sudo nmap -sC <host>
```

To specify a category, we can use:
```bash
sudo nmap --script <category> <host>
```

To specify a specific script, you can use the following:
```bash
sudo nmap --script <script_name>,... <host>
```
All the scripts are located in `/usr/share/nmap/scripts/` on kali. Do not include the `nse` extnension. To pull back everything, we can use the addressive scan:
```bash
sudo nmap -S <host>
```
This will perform service detection (`-sV`), os detection (`-O`), traceroute, and default scripts (`-sC`).

**Fuck It**
Wanna run just fucking everything?
```bash
sudo nmap -A -T5 --script auth,broadcast,brute,default,discovery,dos,exploit,external,fuzzer,intrusive,malware,safe,version,vuln -p- <host>
```
See you in 6 hours. This will for sure fuck up your box if you aren't careful.

### Outputting Results
For each scan, it's generally good to store the output for later comparison. We can use the built-in commands in nmap for this. We should also try to append `--reason` where possible if we want to show this to anyone. Makes report writing easier.
```bash
sudo nmap -sS -p- -T4 -oX scan-<host>-syn.xml --reason <host>
```

And we can make a nice output of the report with:
```bash
xsltproc scan-<host>-syn.xml -o scan-<host>-syn.html
```

It's generally good practice to clearly label your scans so you don't accidentally override and also so you don't mixup results, especially in multi-host environments.