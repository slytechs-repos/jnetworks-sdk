# jnetworks-sdk

[![License](https://img.shields.io/badge/License-Sly%20Technologies-green.svg)](LICENSE)
[![Maven Central](https://img.shields.io/maven-central/v/com.slytechs.jnet.jnetworks/jnetworks-sdk.svg)](https://central.sonatype.com/artifact/com.slytechs.jnet.jnetworks/jnetworks-sdk)
[![JavaDoc](https://javadoc.io/badge2/com.slytechs.jnet.jnetworks/jnetworks-sdk/javadoc.svg)](https://javadoc.io/doc/com.slytechs.jnet.jnetworks/jnetworks-sdk)

A comprehensive SDK that serves as the parent POM and documentation project for the **JNetworks** suite. It includes full GitHub Wiki pages detailing all aspects of the API. The primary public API is located in the **jnetworks-api** module, with current implementations provided by **jnetworks-pcap**, **jnetworks-ntapi**, and **jnetworks-dpdk** modules.

## Overview

**jnetworks-sdk** is the top-level module of the JNetworks suite, acting as the parent POM and central documentation hub. It integrates all **jnetworks-*** modules into a cohesive SDK, providing extensive documentation through GitHub Wiki pages. The SDK offers a unified interface for network monitoring, packet analysis, and traffic manipulation, abstracting the underlying packet capture engines. Whether you're using Libpcap for general-purpose capture, NTAPI for Napatech hardware acceleration, or DPDK for high-throughput packet processing, **jnetworks-sdk** ensures seamless integration and consistent API usage.

## Modules

- **jnetworks-api**: The primary public API defining the core interfaces and abstractions.
- **jnetworks-pcap**: Implementation of the API using Libpcap/Npcap for packet capture.
- **jnetworks-ntapi**: Implementation using Napatech's NTAPI for hardware-accelerated capture.
- **jnetworks-dpdk**: Implementation leveraging Intel's DPDK for high-performance packet processing.

## Features

### Core Functionality:

- **Unified API**: Interact with different packet capture engines using a consistent interface defined in **jnetworks-api**.
- **High-Performance Packet Capture**: Support for line-rate packet capture with minimal CPU usage.
- **Advanced Filtering**: Manage complex packet filters across different backends.
- **Real-Time Monitoring**: Monitor network traffic in real-time with precise timestamping.
- **Statistics Collection**: Gather detailed statistics per adapter, port, and queue.
- **Time Synchronization**: Synchronize time using PTP or other supported methods.
- **Hardware Timestamping**: Achieve high-precision timestamping with supported hardware.
- **Zero-Copy Transmission**: Send and receive packets efficiently with zero-copy mechanisms.
- **Multi-Stream and Multi-Queue Support**: Capture and process multiple data streams or queues simultaneously.
- **Alarm and Event Handling**: Monitor hardware alarms and events from supported adapters.
- **Hardware Offloading**: Leverage hardware-based features like checksum offloading and RSS.
- **NUMA Awareness**: Optimize performance on multi-socket systems.
- **Cross-Platform Support**: Compatible with Linux, Windows, and other supported operating systems.
- **Ease of Integration**: Simplify application development with a high-level API.

### Supported Backends:

- **jnetworks-pcap**: For general-purpose packet capture using Libpcap/Npcap.
- **jnetworks-ntapi**: For Napatech hardware acceleration using NTAPI.
- **jnetworks-dpdk**: For high-throughput packet processing using Intel DPDK.

## Capability Comparison Table

Below is a comparison of the capabilities supported by each backend integrated within **jnetworks-sdk**:

| **Capability**                           | **Libpcap** | **NTAPI** | **DPDK** |
|------------------------------------------|-------------|-----------|----------|
| High-performance packet capture          | Yes         | Yes       | Yes      |
| Advanced filtering                       | Yes         | Yes       | Yes      |
| Real-time monitoring                     | Yes         | Yes       | Yes      |
| Statistics collection                    | Yes         | Yes       | Yes      |
| Time synchronization                     |             | Yes       | Yes      |
| Hardware timestamping                    |             | Yes       | Yes      |
| Zero-copy transmission                   |             | Yes       | Yes      |
| Multi-stream capture                     |             | Yes       | Yes      |
| Alarm and event handling                 |             | Yes       | Yes      |
| Hardware offloading                      |             | Yes       | Yes      |
| Multi-queue support                      |             | Yes       | Yes      |
| NUMA awareness                           |             | Yes       | Yes      |
| Memory management                        |             | Yes       | Yes      |
| Virtualization support                   | Yes         | Yes       | Yes      |
| Hardware acceleration                    |             | Yes       | Yes      |
| Support for hardware NICs                | Yes         | Yes       | Yes      |
| Software-based capture                   | Yes         |           |          |
| Cross-platform support                   | Yes         | Yes       | Yes      |
| Ease of integration                      | Yes         | Yes       |          |
| Support for multi-threading              | Limited     | Yes       | Yes      |

## Documentation

- **GitHub Wiki**: Comprehensive documentation is available on our [GitHub Wiki pages](https://github.com/slytechs-repos/jnetworks-sdk/wiki), covering all aspects of the API, usage examples, and advanced topics.
- **API Documentation**: Detailed API references are provided through JavaDoc for each module.

## Requirements

- **JDK 21 LTS** or later.
- **Backend-specific requirements**:
  - **jnetworks-pcap**:
    - **Linux**: libpcap installed.
    - **Windows**: Npcap installed.
  - **jnetworks-ntapi**:
    - **Napatech NTAPI SDK** installed.
    - **Compatible Napatech Network Adapter Hardware**.
  - **jnetworks-dpdk**:
    - **Intel DPDK** installed and configured.
    - **Compatible Network Adapter Hardware** (e.g., Intel, Mellanox NICs supported by DPDK).

## Installation

### Maven

Include the SDK parent POM in your project to manage dependencies:

```xml
<parent>
    <groupId>com.slytechs.jnet.jnetworks</groupId>
    <artifactId>jnetworks-sdk</artifactId>
    <version>1.0.0</version>
</parent>
```

To use the API and specific implementations, include dependencies for **jnetworks-api** and the desired backend(s):

```xml
<dependencies>
    <!-- Include the API module -->
    <dependency>
        <groupId>com.slytechs.jnet.jnetworks</groupId>
        <artifactId>jnetworks-api</artifactId>
        <version>1.0.0</version>
    </dependency>
    
    <!-- Include the desired backend implementation -->
    <dependency>
        <groupId>com.slytechs.jnet.jnetworks</groupId>
        <artifactId>jnetworks-pcap</artifactId>
        <version>1.0.0</version>
    </dependency>
</dependencies>
```

### Gradle

```groovy
implementation 'com.slytechs.jnet.jnetworks:jnetworks-api:1.0.0'
implementation 'com.slytechs.jnet.jnetworks:jnetworks-pcap:1.0.0' // or ntapi/dpdk
```

## Quick Start

```java
// Use the primary public API from jnetworks-api
import com.slytechs.jnet.jnetworks.api.*;

public class NetworkApp {
    public static void main(String[] args) {
        // Choose backend implementation (e.g., PcapFramework)
        try (NetworkFramework framework = new PcapFramework()) {
            framework.initialize();

            // Configure network interface
            ConfigManager config = framework.createConfigManager();
            InterfaceConfig ifaceConfig = config.getInterfaceConfig();
            ifaceConfig.setInterfaceEnabled("eth0", true); // Enable interface eth0

            // Set up packet filter
            FilterConfig filter = config.getFilterConfig();
            filter.setFilterExpression("tcp port 80"); // Capture HTTP traffic

            // Capture packets
            try (NetCapture capture = framework.createNetCapture()) {
                // Set up packet handler
                capture.setPacketHandler(packet -> {
                    System.out.printf("Received packet: length=%d%n", packet.getLength());
                });

                capture.startCapture();

                // Wait for packets
                Thread.sleep(10000);  // Capture for 10 seconds

                // Get statistics
                StatisticsManager stats = framework.createStatisticsManager();
                InterfaceStats ifaceStats = stats.getInterfaceStats();
                System.out.printf("Packets received: %d%n", ifaceStats.getRxPacketCount("eth0"));
            }
        } catch (Exception e) {
            e.printStackTrace();
        }
    }
}
```

## Advanced Usage

### Accessing Documentation

- Visit the **GitHub Wiki** for detailed guides, API documentation, and usage examples: [JNetworks SDK Wiki](https://github.com/slytechs-repos/jnetworks-sdk/wiki).

### Selecting a Backend at Runtime

```java
// Use a factory or configuration to select the backend
String backendType = "DPDK"; // Could be "Pcap", "NTAPI", or "DPDK"

NetworkFramework framework;
switch (backendType) {
    case "Pcap":
        framework = new PcapFramework();
        break;
    case "NTAPI":
        framework = new NtapiFramework();
        break;
    case "DPDK":
        framework = new DpdkFramework(new String[]{"--file-prefix", "jnet_dpdk"});
        break;
    default:
        throw new IllegalArgumentException("Unsupported backend type");
}

framework.initialize();
// Proceed with common API usage
```

### Utilizing Backend-Specific Features

```java
if (framework instanceof NtapiFramework) {
    NtapiFramework ntapiFramework = (NtapiFramework) framework;
    // Access NTAPI-specific methods
    NtapiConfig ntapiConfig = ntapiFramework.getNtapiConfig();
    ntapiConfig.enableHardwareTimestamping(true);
}
```

## Building from Source

```bash
git clone https://github.com/slytechs-repos/jnetworks-sdk
cd jnetworks-sdk
mvn clean install
```

## Contributing

We welcome contributions! Please see our [Contributing Guidelines](CONTRIBUTING.md) for details.

## License

### Free License

- **Non-redistributing use**
- **Limited to 1-5 system installs**
- **Can be used in internal products**
- Perfect for:
  - Personal use
  - Internal development
  - Prototyping
  - Demonstrations
  - Evaluations

### OEM Internal-Use License (Paid)

- **Perpetual license** (never expires)
- Supports **100 to unlimited system installs**
- Includes:
  - Full OEM support
  - Pro features
- Ideal for larger internal deployments

### OEM Redistributing License (Paid)

- Available as **perpetual or term licenses**
- **No-royalty model**
- **Unlimited system installs**
- Includes:
  - Distribution rights
  - No asset tracking required
  - No reporting requirements

For commercial licensing options and pricing, please contact:

- **Website**: [slytechs.com](http://slytechs.com)
- **Email**: [sales@slytechs.com](mailto:sales@slytechs.com)

## Support

- **Issue Tracker**: [https://github.com/slytechs-repos/jnetworks-sdk/issues](https://github.com/slytechs-repos/jnetworks-sdk/issues)
- **API Documentation**: [https://javadoc.io/doc/com.slytechs.jnet.jnetworks/jnetworks-sdk](https://javadoc.io/doc/com.slytechs.jnet.jnetworks/jnetworks-sdk)
- **Examples**: [https://github.com/slytechs-repos/jnetworks-sdk/tree/main/examples](https://github.com/slytechs-repos/jnetworks-sdk/tree/main/examples)
- **GitHub Wiki**: [https://github.com/slytechs-repos/jnetworks-sdk/wiki](https://github.com/slytechs-repos/jnetworks-sdk/wiki)
- **Commercial Support**: [http://slytechs.com/support](http://slytechs.com/support)

---

Feel free to explore the **jnetworks-sdk** module, utilize the extensive documentation available on our GitHub Wiki, and leverage the power of multiple backends for your network applications using the familiar JNetworks API. If you have any questions or need assistance, our support channels are always open!
