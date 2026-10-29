# Network Protocols From Scratch

A collection of small, self-contained implementations and simulators that rebuild core
networking concepts in Python, from the data link layer up to the application layer.
Each module focuses on one layer or protocol family: Ethernet switching and ARP, random
multiple-access MAC protocols, routing-table convergence, a QUIC-style reliable transport
over UDP, HTTP/1.0 through HTTP/3 clients and servers, and a software-defined networking
controller. The implementations favour clarity over completeness and are meant for study
and experimentation rather than production use.

## Repository structure

| Directory | Topic | Key files |
|-----------|-------|-----------|
| `traffic-analysis/` | Traffic-analysis write-up (report only, no source) | `report.pdf` |
| `switch-arp/` | Layer-2 learning switch and ARP table simulator | `main.py`, `setting.py` |
| `mac-protocols/` | Random-access MAC protocol simulator (ALOHA, slotted ALOHA, CSMA, CSMA/CD) | `main.py`, `protocol.py`, `setting.py` |
| `routing/` | Routing convergence simulator (distance-vector RIP and link-state OSPF) | `main.py` |
| `quic-transport/` | QUIC-style reliable transport over UDP | `quic_server.py`, `quic_client.py`, `df/` (alternative implementation) |
| `http-versions/` | HTTP/1.0, HTTP/1.1, HTTP/2.0 and HTTP/3.0 clients and servers | `http_*_demo.py`, `my/`, `QUIC/` |
| `sdn-controller/` | Software-defined networking controller and topology | `controller.py`, `topo.py` |

Several modules also contain a `snapshot/` subfolder holding an earlier revision of the
same sources, kept for reference.

## Requirements

- Python 3.8 or newer.
- `matplotlib` and `numpy` for the simulators in `mac-protocols/` and `routing/`.
- The SDN controller in `sdn-controller/` depends on the [Ryu](https://ryu-sdn.org/)
  framework and [Mininet](http://mininet.org/); it is written for OpenFlow 1.3 and runs
  on Linux.
- The transport and HTTP modules use only the Python standard library (`socket`,
  `struct`, `threading`).

## Modules

### Ethernet switching and ARP (`switch-arp/`)

Models hosts and switches as Python objects on a topology defined in `setting.py`
(host names, switch names, IP/MAC maps and links). Switches perform MAC learning and
flooding; hosts maintain ARP tables. An interactive prompt accepts commands:

```bash
cd switch-arp
python main.py
```

```text
>> h1 ping h2
>> show_table all_switches
>> show_table h1
>> clear s1
```

### MAC protocols (`mac-protocols/`)

`protocol.py` simulates ALOHA, slotted ALOHA, CSMA and CSMA/CD and returns success,
idle and collision rates for a given `Setting` (host count, packet count, packet size,
link delay, resend probability and seed). `main.py` drives sweeps over these parameters
and plots the results with matplotlib.

```bash
cd mac-protocols
python main.py
```

### Routing convergence (`routing/`)

`main.py` builds a graph from a link-cost matrix and simulates how routing tables
converge under a distance-vector scheme (`run_rip`) and a link-state scheme with
flooding (`run_ospf`). It prints the resulting cost tables and the message exchange
trace for two sample topologies.

```bash
cd routing
python main.py
```

### QUIC-style transport (`quic-transport/`)

A reliable transport built on top of UDP that implements a handshake, per-stream
sliding windows, acknowledgements with retransmission timers, flow control using the
receiver's advertised window, and congestion control. `quic_server.py` and
`quic_client.py` expose `QUICServer`/`QUICClient` classes with `listen`/`accept`,
`connect`, `send`, `recv` and `close`. The `df/` directory holds an alternative
implementation of the same interface.

```bash
cd quic-transport
python quic_server.py   # listens on UDP port 30000
# in a second terminal
python quic_client.py
```

### HTTP/1.0 to HTTP/3.0 (`http-versions/`)

Clients and servers for four HTTP generations, implemented in `my/`:

- HTTP/1.0 and HTTP/1.1 over TCP, with streamed response bodies.
- HTTP/2.0 over TCP, using binary framing and stream multiplexing.
- HTTP/3.0 over the bundled QUIC package in `QUIC/`, which provides independent
  streams so a lost packet on one stream does not block the others.

Each generation has a paired demo. The server serves files from `static/`; the client
lists the index, downloads the linked files and writes them into `target/`.

```bash
cd http-versions
python http_2_0_server_demo.py
# in a second terminal
python http_2_0_client_demo.py
```

Swap the `2_0` prefix for `1_0`, `1_1` or `3_0` to run the other generations.

### SDN controller (`sdn-controller/`)

`controller.py` is an OpenFlow 1.3 controller application that extends a standard
learning-switch example with multi-table ICMP filtering rules, and `topo.py` defines a
four-host single-switch Mininet topology.

```bash
# start the controller
ryu-manager controller.py
# in another terminal, start the topology
sudo mn --custom topo.py --topo mytopo --controller remote
```

## Notes

- The learning-switch structure of the SDN controller in `sdn-controller/` is adapted
  from the OpenFlow 1.3 example that ships with the Ryu framework (Apache 2.0); the
  filtering tables are added on top of it.
- Several modules keep PDF write-ups next to the code as reference material; they are not
  needed to run anything.
