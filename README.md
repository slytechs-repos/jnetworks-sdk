# jNetWorks SDK

[![License](https://img.shields.io/badge/License-Sly%20Technologies-green.svg)](LICENSE)
[![Maven Central](https://img.shields.io/maven-central/v/com.slytechs.jnet.jnetworks/jnetworks-sdk.svg)](https://central.sonatype.com/artifact/com.slytechs.jnet.jnetworks/jnetworks-sdk)
[![JavaDoc](https://javadoc.io/badge2/com.slytechs.jnet.jnetworks/jnetworks-sdk/javadoc.svg)](https://javadoc.io/doc/com.slytechs.jnet.jnetworks/jnetworks-sdk)

Welcome to the **jNetWorks SDK** from Sly Technologies Inc. (www.slytechs.com), your gateway to effortless, high-performance network development. Whether you're building real-time monitoring tools, capturing massive traffic volumes, or analyzing packets with precision, our SDK delivers a unified, hardware-agnostic API that makes complex network tasks feel simple and intuitive. Imagine writing code once and seamlessly switching between software-based solutions or hardware-accelerated powerhouses—no rewrites, no headaches. With three versatile backends tailored to different needs—PCAP for flexible software capture, Intel DPDK for blazing-fast throughput, and Napatech SmartNIC with Link Capture Software (NTAPI) for advanced hardware acceleration—you get the freedom to choose the perfect fit for your project. Dive in and discover why developers love our SDK: it's powerful, scalable, and designed to let you focus on innovation, not integration.

## Overview

The **jNetWorks SDK** is a modular Java framework that empowers developers to handle network monitoring, packet capture, transmission, and analysis with unparalleled ease. At its core is a consistent API that abstracts away backend complexities, ensuring your code works flawlessly across environments. Whether you're prototyping on standard hardware or scaling to enterprise-level demands, the SDK supports Linux and Windows, integrates with the Protocol API for deep packet inspection, and scales to handle millions of streams. Backed by Sly Technologies Inc., a leader in network solutions, this SDK is built for reliability, performance, and simplicity—making it the smart choice for modern network applications.

## Modules

- **jnetworks-api**: The heart of the SDK, providing the unified API for seamless operations across all backends.
- **jnetworks-pcap**: Leverages Libpcap/Npcap for versatile, software-driven packet handling—perfect for broad compatibility and massive scalability.
- **jnetworks-dpdk**: Harnesses Intel DPDK for ultra-high-speed processing, ideal for demanding, low-latency scenarios.
- **jnetworks-ntapi**: Utilizes Napatech SmartNIC with Link Capture Software (NTAPI) for hardware-accelerated features like precise timestamping.
- **protocol-tcpip**: A companion module for effortless packet header analysis, integrating smoothly for real-time insights.

## Features

- **Unified API for Effortless Development**: One intuitive interface to rule them all—capture, transmit, filter, and analyze without backend-specific tweaks.
- **Three Backends, Endless Possibilities**: Choose PCAP for everyday flexibility, DPDK for performance extremes, or NTAPI for hardware precision, all with the same easy API.
- **High-Performance at Scale**: Achieve line-rate capture and transmission, with support for zero-copy efficiency, hash-based load balancing, and dynamic traffic assignment.
- **Real-Time Adaptability**: Apply filters, assignments, and configurations on the fly, even mid-session, for responsive network management.
- **Advanced Analysis Integration**: Pair with the Protocol API to inspect protocols like Ethernet, IP, TCP, and VLAN with zero-copy speed.
- **Structured Concurrency**: Harness Java's virtual threads for parallel processing of streams and buffers, tested to handle 1 million streams effortlessly.
- **Cross-Platform Versatility**: Run smoothly on Linux and Windows, supporting a wide range of adapters from standard NICs to specialized hardware.
- **Extensible and Future-Proof**: Easily add custom file formats or backends, with built-in discovery for evolving needs.

## Capability Comparison Table

Here's how our three backends stack up, giving you the tools to match your exact requirements:

| **Capability**                     | **PCAP** | **DPDK** | **NTAPI** |
|------------------------------------|----------|----------|-----------|
| High-performance packet capture    | Yes      | Yes      | Yes       |
| Advanced filtering                 | Yes      | Yes      | Yes       |
| Real-time monitoring               | Yes      | Yes      | Yes       |
| Statistics collection              | Yes      | Yes      | Yes       |
| Time synchronization               |          | Yes      | Yes       |
| Hardware timestamping              |          | Yes      | Yes       |
| Zero-copy transmission             |          | Yes      | Yes       |
| Multi-stream capture               | Yes      | Yes      | Yes       |
| Hash-based distribution            | Yes      | Yes      | Yes       |
| Scalability (e.g., 1M streams)     | Yes      | Limited  | Limited   |
| Hardware offloading                |          | Yes      | Yes       |
| Multi-queue support                |          | Yes      | Yes       |

## Documentation

- **GitHub Wiki**: Your complete guide to everything jNetWorks, from getting started to advanced topics: [jNetWorks SDK Wiki](https://github.com/slytechs-repos/jnetworks-sdk/wiki).
- **API Documentation**: Explore detailed JavaDoc for intuitive reference: [JavaDoc](https://javadoc.io/doc/com.slytechs.jnet.jnetworks/jnetworks-sdk).

## Requirements

- **JDK 22** or later, leveraging the permanent Foreign Function API for seamless native integration.
- **Backend-Specific Needs**:
  - **PCAP**: Libpcap on Linux or Npcap on Windows.
  - **DPDK**: Intel DPDK setup with compatible NICs.
  - **NTAPI**: Napatech Link Capture Software and SmartNIC hardware.

## Installation

### Maven

Set the SDK as your parent POM for easy dependency management:

```xml
<parent>
    <groupId>com.slytechs.jnet.jnetworks</groupId>
    <artifactId>jnetworks-sdk</artifactId>
    <version>1.0.0</version>
</parent>
```

Add the API and your chosen backend:

```xml
<dependencies>
    <dependency>
        <groupId>com.slytechs.jnet.jnetworks</groupId>
        <artifactId>jnetworks-api</artifactId>
        <version>1.0.0</version>
    </dependency>
    <dependency>
        <groupId>com.slytechs.jnet.jnetworks</groupId>
        <artifactId>jnetworks-pcap</artifactId> <!-- Or -dpdk, -ntapi -->
        <version>1.0.0</version>
    </dependency>
</dependencies>
```

### Gradle

```groovy
implementation 'com.slytechs.jnet.jnetworks:jnetworks-api:1.0.0'
implementation 'com.slytechs.jnet.jnetworks:jnetworks-pcap:1.0.0' // Or -dpdk, -ntapi
```

## Quick Start

Get up and running in minutes with this simple capture example—highlighting the SDK's straightforward API:

```java
try (NetWorks networks = new PcapWorks()) { // Swap to DpdkWorks or NtapiWorks effortlessly
    PacketStream stream = networks.createRxStream("rx-stream");
    try (Capture capture = networks.openCapture("analyzer", "tcp", stream)) {
        capture.shutdownAfter(Duration.ofSeconds(10));
        while (stream.isActive()) {
            Packet packet = stream.take();
            // Analyze or process packet here
            stream.release(packet);
        }
    }
}
```

## Advanced Usage

Explore dynamic configurations, multi-stream setups, or backend-specific optimizations through our Wiki—keeping your code clean and portable.

## Building from Source

```bash
git clone https://github.com/slytechs-repos/jnetworks-sdk
cd jnetworks-sdk
mvn clean install
```

## Contributing

Join us in shaping the future! Check our [Contributing Guidelines](CONTRIBUTING.md).

## License

### Free License

- **Non-redistributing use** limited to 1-5 system installs.
- Ideal for personal projects, internal development, prototyping, demonstrations, and evaluations.
- Use it in your internal products without redistribution.

### OEM Internal-Use License (Paid)

- **Perpetual** (never expires) with 100+ to unlimited system installs.
- Includes full OEM support and pro features for larger internal deployments.

### OEM Redistributing License (Paid)

- **Perpetual or term-based**, no-royalty, unlimited installs.
- Flexible distribution without tracking or reporting—simplifying your business.

For demos or more free installs during evaluation, contact sales. Visit [www.slytechs.com](http://www.slytechs.com) or email [sales@slytechs.com](mailto:sales@slytechs.com) for details.

## Support

- **Issue Tracker**: [GitHub Issues](https://github.com/slytechs-repos/jnetworks-sdk/issues)
- **API Documentation**: [JavaDoc](https://javadoc.io/doc/com.slytechs.jnet.jnetworks/jnetworks-sdk)
- **Examples**: [GitHub Examples](https://github.com/slytechs-repos/jnetworks-sdk/tree/main/examples)
- **GitHub Wiki**: [Wiki](https://github.com/slytechs-repos/jnetworks-sdk/wiki)
- **Commercial Support**: [slytechs.com/support](http://slytechs.com/support)

Unlock the potential of your network applications with the jNetWorks SDK—easy, powerful, and ready for anything. From Sly Technologies Inc., we're here to make your development journey exciting and successful!
