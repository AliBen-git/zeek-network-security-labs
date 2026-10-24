# Lab 2 – Introduction to Zeek and Processing Logs Using CLI

## Overview

This lab introduces Zeek as a network security monitoring tool and explains how to inspect and process the logs it generates. It also covers essential Linux command-line interface (CLI) tools for network investigations.

## Objectives

- Understand the difference between network monitoring and Network Security Monitoring (NSM).
- Explain the main purpose and architecture of Zeek.
- Compare Zeek and Snort.
- Understand Zeek's frameworks and log categories.
- Run Zeek as a service and analyze PCAP files.
- Inspect Zeek logs and extract fields using `zeek-cut`.
- Use Linux CLI commands to filter, sort, and analyze text data.

## 1. Introduction to Zeek

Zeek, formerly known as Bro, is an open-source network traffic analyzer used for Network Security Monitoring (NSM), traffic analysis, and investigations of suspicious or malicious activity.

Zeek produces detailed logs that help security analysts investigate network activity, perform threat hunting, and support network forensics.

## 2. Network Monitoring vs. Network Security Monitoring

### Network Monitoring

Network monitoring focuses on the availability, health, performance, and configuration of IT assets. It helps network administrators troubleshoot connectivity problems, monitor performance, and maintain reliable services.

### Network Security Monitoring

Network Security Monitoring focuses on identifying suspicious network behavior, unusual services or ports, rogue hosts, and potentially malicious traffic. It supports security analysts, incident responders, and threat hunters during investigations.

## 3. Zeek vs. Snort

Zeek and Snort both support network security, but they have different strengths and purposes.

| Feature | Zeek | Snort |
|---|---|---|
| Primary purpose | Network Security Monitoring and traffic analysis | Intrusion detection and prevention |
| Detection approach | Event analysis, scripting, and event correlation | Signature- and rule-based detection |
| Strengths | Detailed traffic visibility, threat hunting, and analysis of complex activity | Established rules and alerts for known threats |
| Limitations | Requires networking knowledge and log analysis | Complex threats may require additional context |
| Common use cases | Network monitoring, in-depth investigation, and event correlation | Intrusion detection, prevention, and known-attack detection |

## 4. Zeek Architecture

Zeek has two primary layers:

- **Event Engine:** Processes packets and identifies information such as source and destination addresses, protocols, sessions, and files.
- **Policy Script Interpreter:** Uses Zeek scripts to perform semantic analysis and describe relationships between events.

Together, these layers allow Zeek to turn network traffic into useful information for analysis.

## 5. Zeek Frameworks

Zeek includes frameworks that extend its functionality. Examples include:

- Logging
- Notice
- Input
- Configuration
- Intelligence
- Cluster
- Broker Communication
- Supervisor
- GeoLocation
- File Analysis
- Signature
- Summary
- NetControl
- Packet Analysis
- TLS Decryption

The Logging Framework is particularly important because many Zeek investigations begin by examining generated logs.

For more information, see the [Zeek documentation](https://docs.zeek.org/).

## 6. Zeek Outputs and Operation Modes

Zeek generates logs based on the network traffic it analyzes. These logs can support traffic monitoring, intrusion detection, threat hunting, and forensic investigations.

There are two common ways to run Zeek:

1. **As a service:** Monitors live network traffic when configured with an appropriate network interface and permissions.
2. **Against a PCAP file:** Analyzes previously captured traffic and generates logs in the working directory.

### Check the Zeek version

```bash
zeek -v
```

### Manage Zeek with ZeekControl

ZeekControl manages the Zeek service. The following commands may require elevated privileges, depending on your installation.

```bash
sudo zeekctl status
sudo zeekctl start
sudo zeekctl stop
```

You can also enter the interactive ZeekControl interface:

```bash
sudo zeekctl
```

Then use its commands:

```text
status
start
stop
```

### Process a PCAP file

```bash
zeek -C -r sample.pcap
```

- `-r` reads and processes a PCAP file.
- `-C` ignores checksum errors.
- `-v` displays version information.
- `zeekctl` manages the Zeek service.

After processing a PCAP file, list the generated files:

```bash
ls -l
```

Example log files include:

```text
conn.log
dhcp.log
dns.log
packet_filter.log
```

The logs produced depend on the traffic in the PCAP file and the scripts enabled.

## 7. Understanding Zeek Logs

Zeek logs are structured, tab-separated text files. Each log contains fields describing different aspects of network activity.

A unique identifier called the **UID** helps correlate related records across logs.

### Common log categories

| Category | Purpose | Example logs |
|---|---|---|
| Network | Network protocol activity | `conn.log`, `dns.log`, `http.log`, `ftp.log`, `ssh.log` |
| Files | File analysis results | `files.log`, `pe.log`, `x509.log` |
| NetControl | Network control and flow activity | `netcontrol.log`, `openflow.log` |
| Detection | Alerts, signatures, and indicators | `notice.log`, `intel.log`, `signatures.log` |
| Network Observations | Hosts, services, and software | `known_hosts.log`, `known_services.log`, `software.log` |
| Miscellaneous | Additional alerts, protocol issues, and failures | `weird.log`, `dpd.log`, `unknown_protocols.log` |
| Zeek Diagnostic | System messages and operational statistics | `loaded_scripts.log`, `stats.log`, `reporter.log` |

The exact logs available depend on the traffic analyzed and the configuration.

### Examples of commonly used logs

| Log | Purpose |
|---|---|
| `conn.log` | Records network connections |
| `files.log` | Records file analysis activity |
| `notice.log` | Records notices generated by Zeek |
| `intel.log` | Records activity matching configured intelligence data |
| `signatures.log` | Records signature matches |
| `known_hosts.log` | Records hosts observed by Zeek |
| `known_services.log` | Records services observed on the network |
| `known_certs.log` | Records observed certificates |
| `software.log` | Records software observations |

Some logs are maintained across multiple sessions, while others record activity from individual sessions. Refer to the Zeek documentation for details about each log.

## 8. Organizing Logs for an Investigation

Before investigating an incident, identify which logs are most relevant to the case and the hypothesis being tested.

| Investigation focus | Example logs | Purpose |
|---|---|---|
| Overall information | `conn.log`, `files.log`, `loaded_scripts.log`, `intel.log` | Review connections, files, scripts, and indicators |
| Protocol-specific analysis | `http.log`, `dns.log`, `ftp.log`, `ssh.log` | Investigate activity involving a specific protocol |
| Detection | `notice.log`, `signatures.log`, `traceroute.log` | Review notices, signature matches, and related indicators |
| Network observations | `known_hosts.log`, `known_services.log`, `software.log` | Understand hosts, services, and software observed |

These categories are examples, not a complete investigation checklist. Select logs based on the incident, available evidence, and investigation hypothesis.

## 9. Extracting Fields with `zeek-cut`

Zeek log files contain metadata describing their fields. The `#fields` line identifies the field names, while `#types` describes their data types.

When using `zeek-cut`, specify the field names rather than the types.

### View a connection log

```bash
cat conn.log
```

Example fields include:

```text
#fields ts uid id.orig_h id.orig_p id.resp_h id.resp_p proto service duration
```

### Extract selected fields

```bash
cat conn.log | zeek-cut uid proto id.orig_h id.orig_p id.resp_h id.resp_p
```

This command extracts:

- `uid`: Connection identifier
- `proto`: Transport protocol
- `id.orig_h`: Originating host
- `id.orig_p`: Originating port
- `id.resp_h`: Responding host
- `id.resp_p`: Responding port

Example output based on the lab data:

```text
CTMFXm1AcIsSnq2Ric  udp  192.168.121.2   51153  192.168.120.22  53
CLsSsA3HLB2N6uJwW  udp  192.168.121.10  50080  192.168.120.10  514
```

Extracting only the relevant fields makes large logs easier to inspect and compare.

## 10. Processing Zeek Logs Using the CLI

Graphical interfaces can help visualize information, but command-line tools are particularly useful when processing large amounts of data or when a graphical interface does not provide the required function.

The following commands form a basic CLI reference for working with text and log data.

### Command history

View command history:

```bash
history
```

Execute the tenth command in history:

```bash
!10
```

Repeat the previous command:

```bash
!!
```

### Read files

Display a file:

```bash
cat sample.txt
```

Display the first 10 lines:

```bash
head sample.txt
```

Display the last 10 lines:

```bash
tail sample.txt
```

### Find and filter information

Extract the first tab-separated field:

```bash
cat test.txt | cut -f 1
```

Extract the first character of each line:

```bash
cat test.txt | cut -c1
```

Search for a keyword:

```bash
cat test.txt | grep 'keywords'
```

### Sort output alphabetically

```bash
cat test.txt | sort
```

### Sort output numerically

```bash
cat test.txt | sort -n
```

### Remove consecutive duplicate lines

```bash
cat test.txt | uniq
```

Note: `uniq` removes adjacent duplicate lines. To remove duplicates regardless of their original order, sort the input first:

```bash
cat test.txt | sort | uniq
```

### Count lines

```bash
cat test.txt | wc -l
```

These commands can be combined into pipelines to filter, organize, and summarize data before investigating suspicious activity.

## Key Takeaways

- Zeek supports network security monitoring through detailed logs and event analysis.
- Snort is commonly associated with signature-based detection, while Zeek provides extensive traffic visibility and scripting capabilities.
- Zeek can analyze PCAP files or monitor live traffic when configured as a service.
- Zeek logs contain structured fields and UIDs that help correlate activity.
- `zeek-cut` extracts selected fields from Zeek logs.
- Linux CLI tools such as `grep`, `cut`, `sort`, `uniq`, and `wc` help analysts process and investigate large log files.

## References

- [Official Zeek documentation](https://docs.zeek.org/)
- [Zeek website](https://zeek.org/)
