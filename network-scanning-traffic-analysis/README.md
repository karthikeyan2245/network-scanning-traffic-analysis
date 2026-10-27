# Network Scanning and Traffic Analysis Lab

## Project Overview

This project demonstrates basic network discovery and traffic analysis
using **Nmap** and **Wireshark** on Windows. The lab focuses on
identifying listening TCP ports on the local computer and observing DNS,
TCP, and TLS traffic.

## Objectives

-   Perform a TCP connect scan against the local computer.
-   Identify open TCP ports and investigate the processes listening on
    selected ports.
-   Capture and filter DNS and TCP packets in Wireshark.
-   Document findings from a small, authorized local lab.

## Tools and Environment

-   **Operating system:** Windows
-   **Nmap:** 7.991
-   **Wireshark:** Used to capture and inspect network packets
-   **Target:** `127.0.0.1` (localhost --- this computer)

## 1. Nmap Port Scan

### Command

``` cmd
nmap -sT -p 135,445,7070 127.0.0.1
```

### Results

  -----------------------------------------------------------------------
  Port              State             Nmap service      Notes
                                      label             
  ----------------- ----------------- ----------------- -----------------
  `135/tcp`         Open              `msrpc`           Commonly used by
                                                        Microsoft Windows
                                                        RPC.

  `445/tcp`         Open              `microsoft-ds`    Commonly
                                                        associated with
                                                        SMB / Windows
                                                        file and printer
                                                        sharing.

  `7070/tcp`        Open              `realserver`      Nmap's service
                                                        label is a guess;
                                                        process
                                                        investigation
                                                        identified
                                                        `AnyDesk.exe`
                                                        listening on this
                                                        port.
  -----------------------------------------------------------------------

The host responded as up. The scan checked only the three specified TCP
ports.

## 2. Process Investigation

The following Windows commands were used to inspect listeners and
identify processes:

``` cmd
netstat -ano | findstr ":7070"
netstat -ano | findstr ":1434"
tasklist /FI "PID eq 5028"
tasklist /FI "PID eq 21660"
```

Observed results:

-   TCP port `7070` was listening on `0.0.0.0` and `[::]`, with PID
    `5028`. `tasklist` identified the process as `AnyDesk.exe`.
-   TCP port `1434` was listening on `127.0.0.1` and `::1`, with PID
    `21660`. `tasklist` identified the process as `sqlservr.exe`.

**Interpretation:** `0.0.0.0` means the service is bound to all local
IPv4 interfaces; it does not by itself prove that the service is
reachable from the internet. The SQL Server listener shown here was
bound to loopback addresses.

## 3. Wireshark DNS Analysis

The Wi-Fi interface was captured while the computer generated normal
network traffic. The display filter used was:

``` text
dns
```

The capture showed DNS queries and responses, including **A** and
**AAAA** lookups.

-   **A record:** requests an IPv4 address.
-   **AAAA record:** requests an IPv6 address.
-   The laptop's IPv4 address observed in the capture was
    `192.168.29.215`.
-   The DNS resolver shown for some queries was `192.168.29.1`.

## 4. Wireshark TCP and TLS Analysis

The display filter used was:

``` text
tcp
```

The capture showed TCP communication between the laptop and remote
servers, including traffic to TCP port `443`. Some packets were decoded
as `TLSv1.2` and `Application Data`.

A further useful filter is:

``` text
tcp.flags.syn == 1
```

This displays TCP packets with the SYN flag set, which can help identify
connection attempts.

**Capture note:** A Wi-Fi capture may not show traffic sent to
`127.0.0.1`, because localhost traffic uses the loopback interface. To
inspect a local Nmap scan in Wireshark, capture on the Npcap Loopback
Adapter if it is available.

## 5. Key Findings

1.  The local computer had TCP ports `135`, `445`, and `7070` open at
    the time of the scan.
2.  Process inspection associated port `7070` with AnyDesk and port
    `1434` with SQL Server.
3.  Wireshark captured DNS queries/responses and TCP/TLS traffic on the
    Wi-Fi interface.
4.  Nmap's service name is an identification hint, not definitive proof
    of the application. The process list provided additional evidence
    for port `7070`.

## 6. Evidence and Screenshots

Place your screenshots and saved capture in the repository using a
structure like this:

``` text
network-scanning-traffic-analysis/
├── README.md
├── screenshots/
│   ├── nmap-full-scan.png
│   ├── nmap-selected-ports.png
│   ├── process-identification.png
│   ├── wireshark-dns.png
│   └── wireshark-tcp-tls.png
└── captures/
    └── network-analysis.pcapng
```

Rename the screenshot files to match the names above, or update this
list to match your actual filenames. Add the `.pcapng` capture only if
you are comfortable sharing it; packet captures can contain private
network metadata.

## 7. Conclusion

This lab provided hands-on practice with Nmap TCP connect scanning,
Windows process identification, and Wireshark packet filtering. It
demonstrated how to combine port-scan results with local process
information and packet-level observations to better understand network
activity.

## Safety and Scope

All scans in this project targeted the local computer (`127.0.0.1`).
Scan only systems and networks you own or have explicit permission to
test. Review packet captures before publishing them because they may
contain device addresses, hostnames, and other private metadata.
