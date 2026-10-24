# Lab 4 - Zeek Scripting and Signatures

## Overview

This lab explores Zeek's event-driven scripting model and demonstrates how custom scripts can be combined with signatures to analyze network traffic and investigate suspicious activity.

Zeek scripts can add context to signature matches, extract relevant connection details, and help analysts investigate events using structured logs.

## Objectives

- Understand Zeek's event-driven architecture.
- Use `zeek_init`, `zeek_done`, and `new_connection` events.
- Extract source and destination details from connection records.
- Write and execute basic Zeek scripts.
- Combine Zeek scripts with custom signatures.
- Load local scripts and selected built-in scripts.
- Explore FTP authentication failures and brute-force detection.

## 1. Why Zeek Scripting Matters

Signatures identify whether a specific traffic pattern occurs. However, a signature match alone may not explain the broader context of an event.

Zeek scripting allows analysts to investigate questions such as:

- What happened before and after an event?
- How frequently did an event occur?
- Which host initiated the connection?
- Does the activity appear suspicious in context?

**Key concept:** Signatures detect events; scripts interpret and add context to those events.

## 2. Zeek's Event-Driven Architecture

Zeek generates events while analyzing network traffic. Examples include new connections, HTTP requests, FTP login failures, DNS queries, and signature matches.

Events can trigger scripts, provide structured information, and be correlated with other events.

### Important lifecycle events

`zeek_init` runs when Zeek begins processing.

```zeek
event zeek_init()
{
    print "Zeek engine started";
}
```

`zeek_done` runs when Zeek finishes processing.

```zeek
event zeek_done()
{
    print "Zeek engine stopped";
}
```

These events are useful for startup messages, initialization, summaries, and cleanup.

## 3. Inspecting Network Connections

The `new_connection` event is triggered when Zeek observes a new network connection.

### Print the complete connection record

Create a file named `102.zeek`:

```zeek
event new_connection(c: connection)
{
    print c;
}
```

Run the script against a packet capture:

```bash
zeek -C -r sample.pcap 102.zeek
```

The output contains connection information, including addresses, ports, protocol details, timing, and connection history. The complete record can be verbose, so extracting relevant fields is often more practical.

### Extract source and destination details

Create a file named `103.zeek`:

```zeek
event new_connection(c: connection)
{
    print "New Connection";

    print fmt("Source: %s:%s",
        c$id$orig_h, c$id$orig_p);

    print fmt("Destination: %s:%s",
        c$id$resp_h, c$id$resp_p);
}
```

Run it with:

```bash
zeek -C -r sample.pcap 103.zeek
```

This script displays the source and destination addresses and ports for each observed connection.

### Understanding connection fields

| Field | Meaning |
|---|---|
| `c$id$orig_h` | Originating host IP address |
| `c$id$orig_p` | Originating host port |
| `c$id$resp_h` | Responding host IP address |
| `c$id$resp_p` | Responding host port |

Zeek uses typed values such as `string`, `addr`, `port`, `count`, `interval`, and `time`. Understanding data types helps when writing comparisons, thresholds, and time-based logic.

## 4. Combining Signatures and Scripts

Signatures detect configured patterns in traffic. Scripts can respond to signature matches and customize how those matches are handled.

### Example signature

The following example looks for an FTP username pattern containing `admin`:

```text
signature ftp-admin {
    ip-proto == tcp
    ftp /.*USER.*admin.*/
    event "FTP Username Input Found!"
}
```

Save the signature as `ftp-admin.sig`.

### Respond to a signature match

Create `201.zeek`:

```zeek
event signature_match(
    state: signature_state,
    msg: string,
    data: string)
{
    if (state$sig_id == "ftp-admin")
    {
        print "Signature hit! --> #FTP-Admin";
    }
}
```

Run Zeek with the packet capture, signature, and script:

```bash
zeek -C -r ftp.pcap -s ftp-admin.sig 201.zeek
```

When the configured signature matches, the script prints a message to the terminal. The exact number of matches depends on the traffic in the packet capture.

## 5. Loading Local Scripts

Zeek can load local policy scripts using the `local` script entry point.

For example:

```bash
zeek -C -r ftp.pcap local
```

The available scripts depend on the Zeek installation and configuration. Loading additional scripts can generate further logs, such as `loaded_scripts.log`, `capture_loss.log`, `notice.log`, and `stats.log`.

A log file does not necessarily contain an alert or record for every script loaded. Its contents depend on the events and results produced during processing.

## 6. Loading a Specific Built-In Script

Zeek also provides scripts for particular detection tasks. The lab demonstrates loading the FTP brute-force detection script directly.

Example command from the lab material:

```bash
zeek -C -r ftp.pcap /opt/zeek/share/zeek/policy/protocols/ftp/detect-bruteforcing.zeek
```

Inspect the generated notice log:

```bash
cat notice.log | zeek-cut ts note msg
```

The lab material shows an example notice describing 20 failed FTP logins against one FTP server within one second.

**Important:** This is an example from the lab instructions. Actual output depends on the packet capture, Zeek installation path, and detection conditions.

## 7. Key Takeaways

- Zeek uses an event-driven model to analyze network activity.
- Lifecycle events help organize script execution.
- The `new_connection` event provides access to connection records.
- Selecting relevant fields makes output easier to understand.
- Signatures detect configured traffic patterns, while scripts can add context.
- Local and built-in scripts extend Zeek's analysis capabilities.
- FTP brute-force detection depends on the observed traffic and the script's detection conditions.

## References

- [Official Zeek Documentation](https://docs.zeek.org/)
- [Zeek Website](https://zeek.org/)
- [Try Zeek](https://try.zeek.org/)
