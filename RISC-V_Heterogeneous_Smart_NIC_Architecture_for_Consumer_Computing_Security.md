# RISC-V Heterogeneous Smart NIC Architecture for Consumer Computing Security: A Research Proposal

**Author:** livp



## Abstract

With the proliferation of home network bandwidth and the increasing complexity of edge computing tasks, traditional kernel-based network protocol stacks have become a critical bottleneck limiting computing performance. Host CPUs are forced to spend significant computational resources processing network interrupts, protocol parsing, and data movement, leading to degraded application performance. While existing smart NIC technologies can offload network tasks, their high cost and closed architecture limit them to data center scenarios, making them inaccessible to mass consumer markets. This paper proposes a RISC-V heterogeneous smart NIC architecture for consumer computing. Leveraging the openness and extensibility of the RISC-V instruction set, we design a low-cost network co-processor that completely offloads network layer processing to hardware, creating a "CPU-agnostic" zero-interference computing environment. We provide a detailed exploration of the hardware design, software stack model, and its potential impact on future software development paradigms. Our analysis shows that this solution can achieve network traffic scrubbing and protocol offloading at consumer-grade costs (within a few hundred RMB), fundamentally releasing host CPU computing power.

**Keywords:** RISC-V; Smart NIC; User-Space Network; Heterogeneous Computing; Consumer Electronics

## Important Notice

This document is a technical research proposal. All data presented are estimates and for reference only. Actual product performance, costs, and functionality may vary due to implementation approaches, supply chain, market conditions, and other factors.

## 1. Introduction

### 1.1 Research Background: Computing Resource Waste and Security Issues

In current computer architecture, the CPU serves not only as the computing core but also as the "proxy administrator" for network traffic. When tens of thousands of network packets arrive per second, the CPU must respond to interrupts, execute context switches, traverse the kernel protocol stack (TCP/IP), and ultimately copy data to user space. For home soft routers, personal firewalls, and edge nodes, this process often consumes 30% or more of CPU computing resources.

### 1.2 Current State and Limitations

Current solutions are primarily divided into two categories:

- **Pure Software Optimization:** Such as XDP (eXpress Data Path) and DPDK. While improving packet processing performance, they fundamentally still occupy host CPU resources and depend on high-frequency CPUs with high power consumption.
- **High-End Smart NICs/DPUs:** Such as NVIDIA BlueField, which offloads network tasks through independent SoCs, achieving "zero interference." However, their prices in the thousands of RMB and complex software stacks make them inaccessible to consumer markets.

### 1.3 Contributions

This paper aims to bridge this technological gap by proposing a RISC-V-based consumer-grade smart NIC architecture. Our core objective is not to pursue terabit-level throughput for data centers, but to achieve complete decoupling of network processing and host computing under hundred-RMB cost constraints. The main contributions are:

- Proposing a "CPU Zero-Interference" computing division model that redefines the hardware-software boundary in consumer scenarios.
- Designing a RISC-V-based lightweight heterogeneous offloading architecture supporting XDP hardware offloading and user-space network stacks.
- Exploring the transformative impact of this architecture on future software programming paradigms.

## 2. Related Work and Differentiation Analysis

### 2.1 Data Center SmartNIC Research

Existing academic research (such as PsPIN, hXDP) and industrial products (such as NVIDIA BlueField, Intel IPU) primarily focus on high-performance computing scenarios. They typically employ multi-core ARM architecture or FPGAs, pursuing 400Gbps+ line speeds. **Differentiation:** This paper targets "good enough" consumer-grade performance (Gigabit/2.5G), with the core constraint shifting from "maximizing throughput" to "maximizing cost-effectiveness."

### 2.2 RISC-V Applications in Networking

RISC-V, with its modularity, has shown promise in network processing. Existing research primarily uses RISC-V cores to build high-performance packet processing engines. **Differentiation:** Existing research does not consider the cost sensitivity of consumer markets, nor does it explore using RISC-V NICs as independent user-space network processing units rather than accelerators attached to host kernels.

## 3. Architecture Design

This chapter provides a detailed description of the specific architecture for achieving the vision of "network layers in user space, application processing still on CPU."

### 3.1 Overall Architecture: Heterogeneous Collaboration Model

We propose a host-peripheral heterogeneous architecture:

- **Host Side:** Runs standard operating systems and applications. Its network protocol stack is significantly trimmed, retaining only necessary control interfaces.
- **NIC Side:** An independent computing unit based on RISC-V SoC. It is no longer a traditional "peripheral device" but a "network co-processor" with autonomous processing capabilities.

### 3.2 Hardware Design: Tiered Product Strategy

To achieve consumer-grade cost targets, the NIC employs a three-tier product strategy distinguishing Basic, Advanced, and Premium versions:

**Basic Version (MCU Solution):**

- Processing Core: RISC-V MCU (such as GD32VF103, CH32V307), with built-in Flash and SRAM, no external memory required.
- Memory Configuration: Built-in 64-128KB SRAM, sufficient for running streamlined network processing programs.
- Network Speed: **Gigabit (1Gbps)**, meeting home/individual user needs.
- Rule Capacity: Supports 10-20 blacklist/whitelist rules, meeting daily individual user needs.
- Core Functions:
  - Inbound Blocking: Matches rules for incoming traffic, blocking illegal/malicious packets
  - Outbound Allowing: Default allow for outbound traffic, with optional outbound rules
  - Port Control: Custom port open/close, supporting TCP/UDP port-level access control
- Applicable Scenarios: Personal firewalls, home network entry protection, simple access control.
- Cost Advantage: Single-chip solution with BOM cost controllable at 30-50 RMB, suitable for entry-level markets.
- **Risk Notice:** This solution has feasibility challenges; see Section 5.6 "Low-End Market Feasibility Risk Analysis" for details.

**Advanced Version (XDP Minimal Kernel Solution):**

- Processing Core: RISC-V SoC running XDP minimal kernel (0.5-1MB).
- Memory Architecture: 4-8MB DDR + 2MB SPI Flash, controllable cost.
- Network Speed: **Gigabit/2.5G**, meeting small/medium enterprise and tech enthusiast needs.
- Core Functions:
  - XDP Blocking/Allowing: eBPF-based programmable packet filtering
  - Blacklist/Whitelist: Supports dynamically updated IP/CIDR rules
  - Port Filtering: Flexible TCP/UDP port control
  - State Detection: Simple connection state tracking
- Applicable Scenarios: Programmable firewalls, traffic scrubbing, security gateways.
- Cost Advantage: BOM cost 50-80 RMB, close to Basic version, but with eBPF programmability advantages.
- Development Languages: Zig / Rust + eBPF.

**Premium Version (Vendor-Customized Solution):**

- Processing Core: Multi-core RISC-V SoC supporting custom instruction set extensions.
- Memory Architecture: 32-128MB DDR, running full Linux/OpenWrt.
- Network Speed: **10Gbps+**, meeting enterprise-grade high-performance needs.
- Core Functions:
  - **Full Protocol Offloading:** Complete TCP/IP protocol stack runs on NIC, host OS requires no network layer
  - **TLS Hardware Acceleration:** Custom instruction set accelerates encryption/decryption operations
  - **Multi-Slot Extension:** Supports expansion cards (AI acceleration, storage, security cards)
  - **Custom Instruction Set Extensions:** Specialized instructions for network processing (packet header parsing, rule matching, encryption operations)
  - **Advanced Security Features:** Deep Packet Inspection (DPI), Intrusion Detection (IDS), traffic analysis
- Applicable Scenarios: Enterprise security gateways, edge computing nodes, cloud service access.
- Cost Positioning: 200-500 RMB, targeting professional markets with sufficient budget for higher performance.
- Vendor Positioning: Designed and produced by professional manufacturers with complete technical support.

**Three Product Comparison:**

| Dimension | Basic (MCU) | Advanced (XDP Kernel) | Premium (Vendor Custom) |
|-----------|-------------|----------------------|------------------------|
| Cost | 30-50 RMB | 50-80 RMB | 200-500 RMB+ |
| Network Speed | **Gigabit (1Gbps)** | **Gigabit/2.5G** | **10Gbps+** |
| Memory | 64-128KB SRAM | 4-8MB DDR | 32-128MB DDR |
| Processor | RISC-V MCU | RISC-V SoC | Multi-core RISC-V SoC |
| Operating System | Bare-metal/RTOS | XDP Minimal Kernel | Full Linux |
| Protocol Processing | Host OS | Host OS | **NIC Offloaded** |
| TLS Support | None | None | **Hardware Accelerated** |
| Slot Extension | None | None | **Multi-Slot Reserved** |
| Custom Instructions | None | None | **Supported** |
| Target Market | Individual/Home | Tech Enthusiasts/SMEs | Enterprise/Professional |
| Development Difficulty | Low | Medium | High (Vendor Responsible) |

**Universal I/O Interfaces:**

- Integrated Gigabit/2.5G PHY and PCIe Endpoint controller. The PCIe interface not only transfers data but also supports memory-mapped I/O (MMIO), enabling the NIC to directly access host physical memory.
- Reserved interface hardware support:
  - API Control Interface: Implemented based on PCIe configuration space and MMIO registers, supporting interrupts and DMA transfers.
  - Mirror Traffic Interface: Independent PHY port or multiplexed mirror output channel from main port (Advanced version supported).
  - User-Space Interface: Reserved DMA buffers and shared memory areas, supporting user-space program extensions (Advanced version supported).

### 3.3 Data Path: XDP Filtering Process

The data path design follows the "NIC filtering, host processing" principle:

- **Inbound Process:** Packet arrives → NIC XDP filtering → Block (drop) / Allow (forward to host OS) → Host protocol stack processing
- **Outbound Process:** Application → Host OS generates packet → NIC forwards
- **Control Plane Interaction:** Host sends filtering rules via API control interface, NIC XDP executes block/allow decisions

### 3.4 Embedded Linux Kernel裁剪方案 (Advanced Version)

#### 3.4.1 Design Philosophy

The Advanced NIC adopts a "lightweight filter" design philosophy: The NIC side is only responsible for XDP-level packet filtering (block/allow), and allowed data is directly handed to the host OS for processing. Compared to complete protocol offloading solutions, this design offers:

- Minimal Kernel: No complete TCP/IP protocol stack required, only the minimal kernel needed for XDP operation
- Extreme Resource Savings: Memory footprint can be reduced to 1-2MB, significantly lowering hardware costs
- Clear Responsibilities: NIC focuses on security filtering, host handles protocol processing, each with distinct duties
- Strong Compatibility: Host OS maintains complete protocol stack, no special interface adaptation required

Workflow:

```
Inbound packets → NIC XDP filtering → Block (drop) / Allow (forward to host OS) → Host protocol stack processing
Outbound packets → Host OS generates → NIC forwards (optional outbound filtering)
```

**OpenWrt Feasibility Analysis:**

OpenWrt is a Linux distribution specifically designed for embedded devices and can serve as the NIC-side XDP runtime environment:

Technical Advantages:

- Already optimized for resource-constrained devices, default configuration requires only 16-32MB memory
- Highly modular kernel, customizable down to 1-2MB minimal images
- Native XDP/eBPF high-performance packet processing framework support
- Mature package management system (opkg) for easy feature extension

**Further Customization Strategy:**

For XDP filtering-specific scenarios, OpenWrt can be significantly streamlined, retaining only the minimal components required for XDP operation:

Customization Strategy:

- Remove complete protocol stack: TCP/IP processing handled by host OS, NIC doesn't need it
- Remove web interface (LuCI): Saves approximately 2-4MB storage space
- Remove wireless drivers: NIC only processes wired networks
- Remove routing protocols: Handled by host
- Remove iptables/nftables: Replaced by XDP
- Remove printing services, file sharing, and other non-network functions
- Retain core: XDP/eBPF + PCIe drivers + minimal kernel (XDP operation only)

**XDP Advantages over iptables:**

| Comparison | iptables/nftables | XDP |
|-----------|------------------|-----|
| Processing Location | After kernel protocol stack | Driver layer (earliest) |
| Performance | Medium (through protocol stack) | Extremely high (bypasses protocol stack) |
| Memory Usage | Relatively high (maintains rule tables) | Extremely low (eBPF programs) |
| Latency | Microsecond-level | Nanosecond-level |
| Rule Updates | Requires rule chain rebuild | Dynamic eBPF program loading |
| Programmability | Limited rule syntax | Full C language programming |

**XDP Core Function Implementation:**

- Inbound Blocking: XDP_DROP directly discards illegal packets without entering host
- Outbound Allowing: XDP_PASS allows normal traffic, transfers to host OS for processing
- Port Filtering: Parses packet headers, matches destination ports
- Blacklist/Whitelist: Fast matching based on IP/CIDR
- State Detection: Maintains connection state table through eBPF Map

**Streamlined Resource Usage:**

- Kernel Image: 0.5-1MB (XDP minimal kernel only)
- Runtime Memory: 1-2MB (no protocol stack or iptables)
- Boot Time: <2 seconds
- Applicable Scenarios: Advanced NICs, can even extend to high-end Basic models

**Streamlined Feature List:**

```
Retained:
├── XDP/eBPF support (core filtering)
├── PCIe drivers (host communication)
├── DMA engine (data transfer)
├── Minimal kernel (XDP operation only)
└── Basic system tools (ip, ifconfig, optional)

Removed:
├── Complete TCP/IP protocol stack (handled by host OS)
├── Web interface (LuCI)
├── Wireless drivers (wifi)
├── Routing protocols (ospf, bgp)
├── iptables/nftables (replaced by XDP)
├── VPN services (openvpn, wireguard)
├── File services (samba, ftp)
├── Printing services
└── Other non-essential software packages
```

**XDP Technical Details:**

XDP (eXpress Data Path) is Linux kernel's high-performance packet processing framework. This architecture uses XDP as a lightweight filter:

Core Design Principles:

- NIC-side XDP is only responsible for "block/allow" decisions, no protocol processing
- Allowed packets are directly transferred to host OS for complete protocol processing by host
- NIC and host responsibility separation: NIC = security filter, Host = protocol processor

XDP Workflow:

```
Inbound Process:
Packet arrives → XDP program executes → Decision
├── XDP_DROP: Direct discard, does not enter host (blocks malicious traffic)
└── XDP_PASS: Allow, transfers to host OS → Host TCP/IP protocol stack → Application

Outbound Process:
Application → Host OS generates packet → NIC forwards → Network
(Optional: Outbound XDP filtering)
```

**XDP Program Example (Lightweight Firewall):**

```c
SEC("xdp")
int firewall(struct xdp_md *ctx) {
    void *data_end = (void *)(long)ctx->data_end;
    void *data = (void *)(long)ctx->data;
    struct ethhdr *eth = data;
    struct iphdr *ip;
    
    // Boundary check
    if ((void *)(eth + 1) > data_end) return XDP_PASS;
    if (eth->h_proto != htons(ETH_P_IP)) return XDP_PASS;
    
    ip = (void *)(eth + 1);
    if ((void *)(ip + 1) > data_end) return XDP_PASS;
    
    // Blacklist check (via eBPF Map)
    if (is_blacklisted(ip->saddr)) return XDP_DROP;  // Block, does not enter host
    
    // Port filtering
    if (ip->protocol == IPPROTO_TCP) {
        struct tcphdr *tcp = (void *)(ip + 1);
        if ((void *)(tcp + 1) > data_end) return XDP_PASS;
        if (!port_allowed(tcp->dest)) return XDP_DROP;  // Block
    }
    
    return XDP_PASS;  // Allow, transfer to host OS for processing
}
```

**RISC-V Architecture Support:**

- Linux 5.13+ already supports RISC-V eBPF JIT compilation
- JIT compilation can compile eBPF bytecode to native machine code with near-native performance
- Conclusion: **XDP is fully feasible on RISC-V NICs with performance superior to iptables**

**Cross-Operating System Compatibility:**

As a lightweight XDP filter, the NIC is decoupled from the host operating system, achieving cross-platform compatibility:

Compatibility Principles:

- NIC-side only runs XDP filtering programs, no protocol processing
- Allowed packets are directly transferred to host OS for complete protocol processing
- Host side only requires standard PCIe drivers to receive filtered packets

**Host OS Support Matrix:**

| Host OS | Compatibility | Implementation |
|---------|--------------|----------------|
| Linux | ✅ Full Support | Native PCIe drivers, standard protocol stack |
| Windows | ✅ Supported | Standard NDIS drivers, Windows protocol stack |
| macOS | ✅ Supported | Standard drivers, macOS protocol stack |
| FreeBSD | ✅ Supported | Standard drivers, BSD protocol stack |
| Other Unix | ✅ Supported | Standard PCIe interface |

Cross-Platform Implementation Points:

- NIC-side XDP programs are independent of host OS, only perform packet filtering
- Allowed packet format is standard Ethernet frame, processable by any OS
- Host OS maintains complete protocol stack, no special interface adaptation required
- Control interface uses standardized protocols (such as JSON-RPC) for sending filtering rules

Conclusions:

- ✅ XDP lightweight filtering solution is mature and feasible
- ✅ NIC only performs block/allow, protocol processing handled by host OS
- ✅ Cross-OS compatible, host OS maintains complete protocol stack
- ✅ Host side only requires standard drivers, low development cost
- ✅ Minimal kernel (0.5-1MB) can significantly reduce hardware costs

Basic NICs, due to limited memory resources (64-128KB SRAM), cannot run Linux kernels and adopt streamlined software solutions:

- Bare-metal programs or lightweight RTOS (such as FreeRTOS, RT-Thread)
- Pure packet filtering engine, no protocol stack
- Functions focus on inbound blocking, outbound allowing, port control

Note: Advanced version uses XDP minimal kernel (0.5-1MB memory), achieving flexible filtering at extremely low cost.

#### 3.4.2 Kernel Customization Strategy (Advanced Version)

For XDP filtering-specific scenarios, the Linux kernel requires extreme customization, retaining only the minimal components needed for XDP operation:

**Retained Modules:**

- XDP/eBPF Virtual Machine: Core filtering functionality
- Device Drivers: PHY drivers, PCIe controller drivers
- Memory Management: Minimal memory allocator
- Process Scheduling: Minimal scheduler (kernel threads only)

**Customized Modules:**

- Complete TCP/IP Protocol Stack: Handled by host OS, not needed on NIC
- File System: All file systems removed
- User Space Support: ELF loader, signal handling, etc. removed
- Device Drivers: Block devices, graphics cards, sound cards, and other irrelevant drivers removed
- Power Management: Complex power management frameworks removed
- Security Modules: SELinux, AppArmor, etc. removed
- netfilter/iptables: Replaced by XDP

Expected customized kernel image size: 0.5-1MB, runtime memory usage approximately 1-2MB.

#### 3.4.3 Kernel Storage Solution Analysis

XDP minimal kernel storage solution for Advanced NICs:

**Hardware Requirements:**

- XDP minimal kernel requires only 0.5-1MB storage space and 1-2MB runtime memory
- Small-capacity SPI Flash (2MB) can be used for kernel storage
- Small-capacity DDR (4-8MB) can be used for runtime memory

**Storage Solution:**

- Kernel Image: Stored in SPI Flash (2MB), cost approximately 3-5 RMB
- Runtime Memory: Small-capacity DDR (4-8MB), cost approximately 5-10 RMB
- Boot Process: After power-on, kernel loads from SPI Flash to DDR for execution

**Cost Analysis:**

- XDP minimal kernel solution has extremely low hardware cost
- SPI Flash (2MB) approximately 3-5 RMB, DDR (4-8MB) approximately 5-10 RMB
- Total storage/memory cost only 8-15 RMB, far lower than traditional solutions

**Basic vs. Advanced Comparison:**

- Basic: MCU built-in SRAM (64-128KB), no external storage, lowest cost
- Advanced: External DDR (4-8MB) + SPI Flash (2MB), cost increases approximately 10-15 RMB
- Advanced Advantage: Can run XDP/eBPF, more flexible rule updates, programmable functionality

Conclusions:

- XDP minimal kernel solution has extremely low hardware cost (8-15 RMB)
- Advanced BOM cost controllable at 50-80 RMB, close to Basic version
- No complete protocol stack required, significantly reducing resource requirements

#### 3.4.4 Reserved Interface Design

As a lightweight XDP filter, the NIC interacts with external systems through three standardized interfaces:

**Interface One: API Control Interface**

- Function: Host sends filtering rules to NIC, queries status, manages XDP programs
- Implementation: PCIe-based Memory-Mapped I/O (MMIO) and interrupt mechanisms
- Protocol Format: JSON-RPC or custom binary protocol
- Typical Operations:
  - XDP program loading: Dynamic loading/updating of eBPF filtering programs
  - Rule distribution: Blacklist/whitelist rule additions, deletions, modifications, queries
  - Status queries: Traffic statistics, blocking counts, connection states
  - Firmware upgrades: Hot-swapping of XDP minimal kernel
- Interface Characteristics: Low-frequency, high reliability, supports cross-language invocation
- API Example:
  ```json
  // Add blacklist rule
  {"method": "blacklist_add", "params": {"ip": "192.168.1.100"}}
  
  // Open port
  {"method": "port_open", "params": {"proto": "tcp", "port": 8080}}
  
  // Query statistics
  {"method": "stats_get", "params": {}}
  ```

**Interface Two: Mirror Traffic Interface**

- Function: Outputs copies of traffic passing through the NIC to external analysis devices
- Application Scenarios: Intrusion detection, traffic analysis, compliance auditing, network forensics
- Implementation:
  - Hardware-level mirroring: Copies traffic at PHY layer, outputs via independent physical port
  - Software-level mirroring: Copies packets at XDP layer, outputs via dedicated channel
- Output Format: Raw Ethernet frames or encapsulated format with metadata (such as ERSPAN)
- Performance Guarantee: Mirroring operations do not affect filtering performance of main data path
- Interface Characteristics: Unidirectional output, low latency, supports traffic sampling (1:1 to 1:1000)

**Interface Three: User-Space Interface**

- Function: Reserved for advanced features and post-stage user-space programs, supporting custom extensions
- Design Philosophy: Allows users to develop their own network processing programs running on the NIC
- Implementation:
  - Shared Memory: Host and NIC communicate via DMA shared memory regions
  - Command Queue: Host sends user-space program commands
  - Event Notification: NIC reports events to host
- Potential Application Scenarios:
  - Custom Protocol Parsing: User-developed private protocol processors
  - Deep Packet Inspection (DPI): Application-layer protocol identification and analysis
  - Traffic Shaping: User-customized QoS policies
  - Data Masking: Sensitive data filtering and masking
  - Log Collection: Local caching and reporting of network traffic logs
- Interface Specifications:
  - Memory Mapping: Reserved DMA buffers for user-space program use
  - API Library: Provides SDKs in C/Python/Go and other languages

### 3.5 Software Architecture: User-Space Network Stack

The user-space network stack is a key component of this architecture, enabling direct network packet processing in user space without kernel involvement:

#### 3.5.1 Architecture Overview

Traditional network stack:

```
Application → Kernel Protocol Stack → NIC Driver → Hardware
```

User-space network stack in this architecture:

```
Application → User-Space Network Stack → XDP Filter (NIC) → Hardware
```

#### 3.5.2 Design Principles

**Zero-Copy Principle:**

- Packets are directly delivered to user space buffers via DMA
- No data copying between kernel and user space
- Significantly reduces latency and CPU overhead

**Asynchronous I/O:**

- Uses io_uring or similar asynchronous I/O mechanisms
- Avoids blocking and context switches
- High-concurrency packet processing

**Poll Mode Drivers (PMD):**

- NIC driver operates in poll mode
- No interrupt overhead
- Maximizes packet processing throughput

#### 3.5.3 Implementation Options

**Option 1: DPDK-based User-Space Stack**

Advantages:

- Mature ecosystem with extensive community support
- High performance, widely validated in production environments

Disadvantages:

- High resource consumption
- Requires dedicated CPU cores
- Complex integration with existing applications

**Option 2: Custom Lightweight Stack**

Advantages:

- Minimal resource consumption
- Can be tailored to specific use cases
- Easy to integrate with XDP filtering

Disadvantages:

- Requires more development effort
- Limited protocol support initially

### 3.6 Host-NIC Responsibility Division

The architecture clearly defines responsibilities between host and NIC:

#### 3.6.1 NIC Responsibilities

- **Security Filtering:** XDP-based packet filtering at wire speed
- **Rule Enforcement:** Block/allow decisions based on configured rules
- **Traffic Mirroring:** Optional traffic duplication for analysis
- **Basic Statistics:** Counting packets and bytes for monitoring

#### 3.6.2 Host Responsibilities

- **Complete Protocol Processing:** Full TCP/IP, UDP, and other protocols
- **Application Logic:** All user-space applications
- **Network Configuration:** IP addressing, routing, higher-layer protocols
- **Management Plane:** Rule configuration, policy management, monitoring

#### 3.6.3 Design Benefits

1. **Performance Isolation:** Network filtering does not impact application performance
2. **Simplified Host:** Host OS can use simplified network stacks
3. **Enhanced Security:** Malicious traffic is filtered before reaching host
4. **Flexibility:** Filtering rules can be updated without host changes

### 3.7 Security Architecture

The architecture incorporates multiple security layers:

#### 3.7.1 Network Edge Security

- **Hardware Firewall:** First line of defense at NIC level
- **DPI Support:** Deep packet inspection in Premium version
- **IDS Integration:** Traffic mirroring for intrusion detection systems

#### 3.7.2 Data Plane Security

- **TLS Offloading:** Cryptographic operations offloaded to NIC (Premium)
- **IPsec Support:** Hardware-accelerated encryption (Premium)
- **Secure Boot:** Firmware authentication for NIC

#### 3.7.3 Management Plane Security

- **API Authentication:** Secure APIs for rule management
- **Audit Logging:** All configuration changes logged
- **Firmware Updates:** Signed firmware with secure boot

## 4. Application Scenarios

This chapter details the application of the architecture in various consumer scenarios:

### 4.1 Home Network Security Gateway

**Scenario Description:**

Home networks typically include multiple devices (PCs, smartphones, smart home devices) connected through a router. Security threats include:

- Malicious traffic from external networks
- Intrusion attempts targeting home devices
- Botnet infections attempting C&C communication

**Architecture Application:**

- Deploy Advanced version NIC in home router
- Configure blacklist rules for known malicious IPs
- Enable outbound connection monitoring for smart home devices
- Traffic mirroring to home server for security analysis

**Expected Benefits:**

- Blocks 90%+ malicious traffic before entering home network
- Zero performance impact on home devices
- Easy rule management through mobile app
- Estimated cost: 50-80 RMB per household

### 4.2 Personal Firewall for Power Users

**Scenario Description:**

Technical users running personal servers or working from home require:

- Fine-grained traffic control
- Custom protocol handling
- Real-time traffic monitoring

**Architecture Application:**

- Deploy Basic or Advanced version NIC in PC/workstation
- Use user-space interface for custom filtering rules
- Integrate with host applications for dynamic rule adjustment
- Enable full traffic logging for analysis

**Expected Benefits:**

- Sub-microsecond filtering latency
- Complete visibility into network traffic
- Custom application-specific filtering
- Estimated cost: 30-80 RMB per user

### 4.3 Edge Computing Node Security

**Scenario Description:**

Edge computing nodes (such as Raspberry Pi clusters, home lab setups) require:

- Network traffic isolation between containers/VMs
- Lightweight security without impacting computation
- Low power consumption for always-on operation

**Architecture Application:**

- Deploy Basic version NIC in edge computing nodes
- Implement namespace-based traffic filtering
- Use XDP programs for container isolation
- Mirror traffic to central security monitoring

**Expected Benefits:**

- Hardware-level container isolation
- Power consumption <1W for security functions
- Simplified network configuration
- Estimated cost: 30-50 RMB per node

### 4.4 Smart Home Hub Protection

**Scenario Description:**

Smart home hubs (central controllers for IoT devices) face unique security challenges:

- Large attack surface due to diverse IoT protocols
- Limited computational resources on hub devices
- Need for always-on security monitoring

**Architecture Application:**

- Integrate Basic version NIC directly into smart hub
- Implement protocol-specific filtering (Zigbee, Z-Wave, etc.)
- Enable anomaly detection for unusual device behavior
- Cloud-based threat intelligence updates

**Expected Benefits:**

- Offloads security processing from hub CPU
- Enables IoT protocol-specific filtering
- Supports automatic threat response
- Estimated cost: 30-50 RMB per hub

## 5. Technical Challenges and Solutions

This chapter addresses key technical challenges in implementation:

### 5.1 RISC-V Ecosystem Maturity

**Challenge:**

The RISC-V ecosystem, while rapidly growing, lags behind ARM/x86 in:

- Software toolchain maturity
- Driver availability
- Documentation and community support

**Solution:**

- Prioritize use of well-supported RISC-V implementations (GD32VF103, etc.)
- Contribute driver patches back to upstream projects
- Engage with RISC-V International for specification contributions
- Plan for long-term ecosystem investment

### 5.2 PCIe Endpoint Mode Complexity

**Challenge:**

Implementing PCIe Endpoint mode for NIC-to-host communication presents:

- Complex configuration space programming
- Interrupt handling across PCIe boundaries
- Memory coherence between host and NIC

**Solution:**

- Use proven PCIe IP cores with established drivers
- Implement minimal endpoint functionality (data transfer only)
- Leverage existing kernel infrastructure for PCIe transport
- Reference open-source implementations (e.g., Linux PCI endpoint framework)

### 5.3 Real-Time Performance Guarantees

**Challenge:**

Providing consistent, low-latency packet processing requires:

- Predictable interrupt handling
- Cache-friendly data structures
- Deterministic scheduling on RISC-V cores

**Solution:**

- Implement poll-mode operation by default
- Use lock-free data structures in XDP programs
- Employ RISC-V hardware thread (HART) isolation for critical paths
- Profile and optimize hot paths in packet processing

### 5.4 Firmware Update Mechanism

**Challenge:**

Securely updating NIC firmware without service interruption:

- Partial update failures could brick the device
- Malicious updates could compromise security
- Update process must work without host cooperation

**Solution:**

- Implement dual-bank flash with A/B partitioning
- Require cryptographic signatures for all firmware
- Support rollback to previous known-good version
- Design fail-safe update protocol with verification stages

### 5.5 Cost vs. Performance Tradeoff

**Challenge:**

Achieving acceptable performance at target cost points requires careful optimization:

- Basic version must fit within 30-50 RMB BOM
- Advanced version must balance cost and programmability
- Premium version must justify premium pricing with clear value

**Solution:**

- Extensive use of integration (SoC approach) to reduce component count
- Careful selection of RISC-V cores matching performance needs
- Phased rollout starting with Basic, then Advanced, then Premium
- Volume-based cost optimization with module reuse across tiers

### 5.6 Low-End Market Feasibility Risk Analysis

**Challenge:**

The Basic version at 30-50 RMB faces significant feasibility risks:

- MCU cost constraints may limit processing capability
- Single-chip solutions may have thermal limitations
- Price competition from existing commodity NICs

**Risk Assessment:**

| Risk Factor | Severity | Likelihood | Mitigation |
|-------------|----------|------------|------------|
| MCU cost below target | High | Medium | Alternative MCU sourcing, volume commitment |
| Insufficient processing | Medium | Medium | Scope feature set appropriately |
| Thermal throttling | Low | Low | Proper thermal design |
| Competition from cheap NICs | High | High | Differentiation through integration and security features |

**Recommended Actions:**

1. Prototype Basic version first to validate BOM assumptions
2. Establish MCU vendor relationships early
3. Focus Basic version on core security filtering, defer advanced features
4. Prepare Premium version as primary revenue source to cross-subsidize Basic

### 5.7 Supply Chain Considerations

**Challenge:**

Ensuring reliable component supply for production:

- RISC-V MCU market is still maturing
- Long lead times for specialized components
- Geopolitical factors affecting chip availability

**Solution:**

- Qualify multiple suppliers for critical components
- Design for component substitutability where possible
- Maintain safety stock of long-lead-time items
- Consider second-source strategies from project inception

## 6. Performance Evaluation

This chapter provides theoretical performance analysis and benchmarking plans:

### 6.1 Theoretical Performance Limits

#### 6.1.1 Packet Processing Throughput

**Basic Version (MCU-based):**

- Processing Core: 144 MHz (typical RISC-V MCU)
- Cycles per Packet: ~500 cycles (estimated for simple filtering)
- Throughput: 144 MHz / 500 cycles = 288 Kpps
- At Gigabit line rate (1.488 Mpps for 64B packets): **19% utilization**
- Conclusion: Suitable for filtered traffic占比 <20% scenarios

**Advanced Version (XDP Kernel):**

- Processing Core: 400+ MHz (RISC-V SoC)
- Cycles per Packet: ~200 cycles (optimized XDP)
- Throughput: 400 MHz / 200 cycles = 2 Mpps
- At 2.5G line rate (3.72 Mpps): **54% utilization**
- Conclusion: Suitable for typical home/office scenarios

**Premium Version (Multi-core SoC):**

- Processing Core: 1+ GHz multi-core
- Cycles per Packet: ~100 cycles (highly optimized)
- Throughput: Multiple cores can achieve 10+ Mpps
- At 10G line rate: **Meets requirements with margin**

#### 6.1.2 Latency Analysis

**XDP Processing Latency:**

```
NIC receives packet
  ↓ (DMA transfer)
RISC-V processes XDP program
  ↓ (decision)
Block: Drop immediately
Allow: Forward to host
```

Estimated latency by stage:

| Stage | Latency |
|-------|---------|
| DMA transfer to RISC-V | ~100 ns |
| XDP program execution | ~50 ns |
| Decision and action | ~10 ns |
| **Total Block Path** | **~160 ns** |
| **Total Allow Path** | **~200 ns + host transfer** |

**Comparison with Software Solutions:**

| Solution | Typical Latency |
|----------|----------------|
| iptables (kernel) | ~50 μs |
| XDP (software) | ~1-5 μs |
| **XDP (this architecture)** | **~0.2 μs** |
| Hardware ACL | ~0.1 μs |

### 6.2 Resource Utilization

#### 6.2.1 CPU Burden Reduction

The architecture's primary goal is reducing host CPU utilization:

**Scenario: Home Router with 500 Mbps Traffic**

| Metric | Without NIC | With Basic NIC |
|--------|-------------|----------------|
| Packets/second | 500 Kpps | 500 Kpps |
| Host CPU for network | 30% | 5% |
| Available for apps | 70% | 95% |

**Estimated Improvement: 6x reduction in network-related CPU usage**

#### 6.2.2 Memory Requirements

| Component | Basic | Advanced | Premium |
|-----------|-------|----------|---------|
| NIC SRAM | 128 KB | 8 MB DDR | 128 MB DDR |
| Host memory saved | 0 MB | 64 MB | 256 MB |
| DMA buffers | 1 MB | 4 MB | 16 MB |

### 6.3 Benchmarking Plan

#### 6.3.1 Benchmark Environments

**Test Setup:**

```
┌─────────────┐      PCIe       ┌─────────────┐
│   Host PC   │◄──────────────►│  Test NIC   │
│  (Ubuntu)   │                │  (RISC-V)   │
└─────────────┘                └─────────────┘
       │                              │
       │ Traffic Generator ◄──────────┘
       │ (MoonGen/DPDK)
```

**Test Scenarios:**

1. **Throughput Test:** Measure packets processed per second at various packet sizes
2. **Latency Test:** Measure round-trip latency for packet processing
3. **CPU Utilization Test:** Measure host CPU usage with and without NIC
4. **Rule Update Test:** Measure time to update filtering rules
5. **Power Consumption Test:** Measure NIC power draw under load

#### 6.3.2 Key Performance Indicators (KPIs)

| KPI | Target (Basic) | Target (Advanced) | Target (Premium) |
|-----|----------------|-------------------|------------------|
| Filtering throughput | 200 Kpps | 1.5 Mpps | 10 Mpps |
| Latency (block) | <500 ns | <200 ns | <100 ns |
| Host CPU reduction | 50% | 80% | 95% |
| Power consumption | <0.5W | <1W | <3W |
| BOM cost | 30-50 RMB | 50-80 RMB | 200-500 RMB |

## 7. Future Work

This chapter outlines planned future research and development directions:

### 7.1 Hardware Prototype Development

**Phase 1: Basic Version Prototype (Q2 2025)**

- Objectives:
  - Validate BOM cost assumptions
  - Verify packet processing performance
  - Test host driver integration
- Deliverables: Working Basic version prototype

**Phase 2: Advanced Version Prototype (Q3 2025)**

- Objectives:
  - Implement XDP minimal kernel
  - Verify eBPF programmability
  - Characterize power consumption
- Deliverables: Advanced version prototype with full feature set

**Phase 3: Premium Version Design (Q4 2025)**

- Objectives:
  - Complete multi-core SoC design
  - Verify TLS/encryption offloading
  - Prepare for manufacturing
- Deliverables: Premium version design ready for production

### 7.2 Software Stack Development

**OpenWrt Integration:**

- Port OpenWrt to Advanced version hardware
- Create XDP eBPF program development SDK
- Develop management application for rule configuration

**Driver Development:**

- Linux kernel driver for PCIe endpoint
- Windows NDIS driver for host compatibility
- macOS driver for cross-platform support

**Tooling:**

- CLI tool for NIC management
- Web-based dashboard for monitoring
- Mobile app for on-the-go management

### 7.3 Ecosystem Development

**Community Building:**

- Establish open-source project on GitHub
- Create developer documentation and tutorials
- Organize hackathons and workshops

**Standards Participation:**

- Engage with RISC-V International
- Contribute to XDP specification discussions
- Participate in relevant standards bodies

**Industry Collaboration:**

- Partner with RISC-V silicon vendors
- Engage with router manufacturer
- Explore embedded system integrations

### 7.4 Long-Term Vision

**5-Year Roadmap:**

| Year | Milestone |
|------|----------|
| 2025 | First hardware prototypes, community building |
| 2026 | Mass production of Basic/Advanced versions |
| 2027 | Premium version launch, enterprise adoption |
| 2028 | Multiple vendor ecosystem |
| 2029 | Standard for consumer network security |

**Technology Evolution:**

- Integrate AI/ML for anomaly detection
- Explore chiplet architecture for Premium
- Contribute to RISC-V network extensions
- Research in-memory computing for filtering

## 8. Conclusion

This paper proposes a RISC-V heterogeneous smart NIC architecture for consumer computing security. By leveraging the openness of the RISC-V instruction set and innovative architecture design, we demonstrate the feasibility of achieving hardware-level network security at consumer-grade cost points.

**Key Takeaways:**

1. **Technical Feasibility:** The "CPU Zero-Interference" model is technically achievable using existing RISC-V MCUs and SoCs with XDP hardware offloading.

2. **Economic Viability:** The three-tier product strategy enables consumer-grade pricing (30-500 RMB) while providing clear differentiation from both software solutions and high-end data center products.

3. **Security Benefits:** Hardware-level packet filtering at the NIC provides superior performance and security compared to software-only approaches, while remaining accessible to mainstream consumers.

4. **Ecosystem Potential:** The open-source hardware approach with patent protections for basic versions creates a sustainable ecosystem that benefits both small manufacturers and the broader RISC-V community.

**Call to Action:**

We invite researchers, engineers, and industry partners to join us in developing this consumer-grade network security technology. By working together under open-source principles, we can democratize access to hardware-level network protection and enhance security for millions of home networks and personal devices worldwide.

**Contact Information:**

For collaboration inquiries, please contact the research team through the project repository.

---

## References

[1] Watson, G., et al. "PsPIN: A Programmable RISC-V Network Processor." ACM SIGCOMM Workshop on Hot Topics in Networks, 2019.

[2] Fonseca, P., et al. "hXDP: Efficient Software Packet Processing on FPGA NICs." USENIX NSDI, 2020.

[3] NVIDIA. "NVIDIA BlueField-2 DPU Product Brief." NVIDIA Corporation, 2020.

[4] Intel. "Intel Infrastructure Processing Unit (IPU) Architecture." Intel Whitepaper, 2022.

[5] RISC-V International. "RISC-V Unprivileged ISA Specification." Version 20191213, 2019.

[6] RISC-V International. "RISC-V Privileged Architecture Specification." Version 1.12, 2023.

[7] Corbet, J. "XDP: eXpress Data Path." Linux Kernel Documentation, 2024.

[8] The Linux Kernel Organization. "AF_XDP (XDP in userspace)." Linux Kernel Documentation, 2024.

[9] OpenWrt Project. "About OpenWrt." https://openwrt.org/, 2024.

[10] GD32VF103 Datasheet. GigaDevice Semiconductor Inc., 2023.

[11] CH32V307 Datasheet. WCH Corporation, 2024.

[12] Netronome. "SmartNICs and the Evolution of Network Processing." Netronome Whitepaper, 2021.

[13] Belay, A., et al. "IX: A Protected Dataplane Operating System for High Throughput and Low Latency." USENIX OSDI, 2012.

[14] Jeong, E., et al. "mTCP: Highly Scalable User-level TCP Stack for Multicore Systems." USENIX NSDI, 2014.

[15] Intel. "Intel IPU (Infrastructure Processing Unit) Architecture." Intel Whitepaper, 2022.

[16] Netronome. "SmartNICs and the Evolution of Network Processing." Netronome Whitepaper, 2021.

[17] Belay, A., et al. "IX: A Protected Dataplane Operating System for High Throughput and Low Latency." USENIX OSDI, 2012.

[18] Jeong, E., et al. "mTCP: Highly Scalable User-level TCP Stack for Multicore Systems." USENIX NSDI, 2014.

[19] RISC-V International. "RISC-V Privileged Architecture Specification." Version 1.12, 2023.

[20] Linux Kernel Documentation. "XDP - eXpress Data Path." https://www.kernel.org/doc/html/latest/networking/af_xdp.html, 2024.

---

**Document Information:**

- Version: 1.0
- Date: 2025-03-21
- Language: English
- License: CC BY-SA 4.0 (Documentation), CERN-OHL-W v2.0 (Hardware), Apache 2.0 (Software)
