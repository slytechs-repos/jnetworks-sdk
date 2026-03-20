- - # jNetWorks SDK

    High-performance, multi-CPU network packet capture and analysis for Java.

    ## Overview

    jNetWorks provides a unified API for packet capture across multiple backends:

    - **PcapBackend** - libpcap/WinPcap (included, open source)
    - **DpdkBackend** - DPDK for 100Gbps+ capture (commercial)
    - **NtapiBackend** - Napatech SmartNICs for 800Gbps capture (commercial)

    Write once, swap backends. Same code works everywhere.

    ## Requirements

    - **JDK 22+** (FFM standard)
    - **Maven 3.8+**
    - **libpcap** (Linux/macOS) or **Npcap** (Windows)

    ## Quick Start

    Add the SDK dependency:

    ```xml
    <dependency>
        <groupId>com.slytechs.sdk</groupId>
        <artifactId>jnetworks-sdk</artifactId>
        <version>3.0.0-SNAPSHOT</version>
        <type>pom</type>
    </dependency>
    ```

    For commercial backends (DPDK, Napatech), add the repository:

    ```xml
    <repositories>
        <repository>
            <id>slytechs-commercial</id>
            <url>https://maven.slytechs.com/releases</url>
        </repository>
    </repositories>
    ```

    ## Examples

    ### Multi-CPU Capture (Recommended Pattern)

    ```java
    public class MultiCpuCapture {
    
        public static void main(String[] args) {
            new MultiCpuCapture().run();
        }
    
        public void run() {
            Net.activateLicense();
    
            try (Net net = new PcapBackend()) {
    
                ProtocolStack stack = new ProtocolStack();
                PacketStreamSettings settings = new PacketStreamSettings();
    
                // Create 4 streams for parallel processing
                PacketStream[] streams = net.createPacketStreams("stream-%d", 4, settings, stack);
    
                // Capture pipeline: capture -> filter -> distribute
                net.capture("en0", "en1")
                    .filter("icmp")
                    .pipe(streams)
                    .hash(HashType.HASH_5_TUPLE)
                    .slice(64)
                    .apply();
    
                // Fork worker threads with lifecycle management
                try (TaskScope scope = new TaskScope(net)) {
                    scope.shutdownAfter(Duration.ofMinutes(1));
                    scope.fork(streams, this::processStream)
                        .awaitCompletion();
                }
    
            } catch (NetException | InterruptedException e) {
                e.printStackTrace();
            }
        }
    
        void processStream(PacketStream stream) {
            while (stream.isActive()) {
                try {
                    Packet packet = stream.take();
    
                    // Process packet...
    
                    stream.release(packet);
    
                } catch (SessionShutdownException | InterruptedException e) {
                    // Loop exits via isActive() check
                }
            }
        }
    }
    ```

    ### Single-Threaded Capture

    ```java
    try (Net net = new PcapBackend()) {
    
        PacketStream stream = net.createPacketStream("main", settings, stack);
    
        net.capture(PortFilter.ethernet().up().first())
            .filter("tcp port 443")
            .pipe(stream)
            .apply();
    
        try (TaskScope scope = new TaskScope(net)) {
            scope.shutdownAfter(Duration.ofSeconds(30));
            scope.join(() -> {
                while (stream.isActive()) {
                    try {
                        Packet packet = stream.take();
                        System.out.println(packet);
                        stream.release(packet);
                    } catch (SessionShutdownException | InterruptedException e) {
                        // Exit gracefully
                    }
                }
            });
        }
    }
    ```

    ### Protocol Streams (Type-Safe)

    ```java
    try (Net net = new PcapBackend()) {
    
        // Type-safe protocol stream
        ProtocolStream<IpDatagram> ipStream = 
            net.createProtocolStream("ip", IpDatagram.class, settings, stack);
    
        net.capture(PortFilter.ethernet().up())
            .filter("ip")
            .pipe(ipStream)
            .apply();
    
        try (TaskScope scope = new TaskScope(net)) {
            scope.shutdownAfter(Duration.ofSeconds(10));
            scope.fork(ipStream, stream -> {
                while (stream.isActive()) {
                    try {
                        IpDatagram ip = stream.take();  // Returns IpDatagram, not Packet
                        System.out.printf("IP: %s -> %s%n", ip.src(), ip.dst());
                        stream.release(ip);
                    } catch (SessionShutdownException | InterruptedException e) {
                        // Exit gracefully
                    }
                }
            }).awaitCompletion();
        }
    }
    ```

    ### Port Discovery

    ```java
    try (Net net = new PcapBackend()) {
    
        // Chainable filters
        Port[] ports = net.getPorts(PortFilter.ethernet().up());
        String[] names = net.getPortNames(PortFilter.matching("eth?"));
    
        // Use filter directly in capture
        net.capture(PortFilter.ethernet().up().first())
            .pipe(stream)
            .apply();
    
        // Debug hook - see what was selected
        net.capture(PortFilter.active())
            .onPortSelection(port -> System.out.println("Capturing: " + port.name()))
            .filter("udp")
            .pipe(stream)
            .apply();
    }
    ```

    ### Capture Options

    ```java
    net.capture("en0", "en1")
        .filter("tcp")
        .pipe(streams)
        
        // Distribution
        .hash(HashType.HASH_5_TUPLE)
        
        // Capture options
        .slice(64)                          // Truncate to 64 bytes
        .descriptor(DescriptorType.DPDK)    // Descriptor format
        .priority(5)                        // Rule priority
        .color(7)                           // Classification color
        .tag("tcp-traffic")                 // Tag for management
        
        // Retransmit options
        .txPort(3)                          // Transmit port
        .txSlice(128)                       // Transmit slice
        .txImmediately()                    // No inter-frame gap
        
        .apply();
    ```

    ## Architecture

    ```
    Application
    ├── Net (root session)
    │   ├── capture().filter().pipe().apply()   // Routing pipeline
    │   ├── createPacketStream()                // Stream factory
    │   ├── createProtocolStream()              // Type-safe streams
    │   └── getPorts(PortFilter)                // Port discovery
    │
    ├── TaskScope (lifecycle management)
    │   ├── fork(streams, task)                 // Parallel execution
    │   ├── join(task)                          // Single-threaded
    │   ├── shutdownAfter(duration)             // Scheduled shutdown
    │   └── awaitCompletion()                   // Block until done
    │
    └── DataStream<T> (type-safe pools)
        ├── PacketStream                        // Raw packets
        └── ProtocolStream<T>                   // Protocol objects
    
    Backends
    ├── PcapBackend   (open source)   ~10 Gbps
    ├── DpdkBackend   (commercial)    ~100 Gbps
    └── NtapiBackend  (commercial)    ~800 Gbps
    ```

    ## Session Hierarchy

    Sessions form a tree with propagating shutdown:

    ```
    Net (root)
     └── TaskScope
          ├── Worker Thread 1 -> PacketStream[0]
          ├── Worker Thread 2 -> PacketStream[1]
          ├── Worker Thread 3 -> PacketStream[2]
          └── Worker Thread 4 -> PacketStream[3]
    
    net.shutdown()     -> Graceful shutdown propagates to all descendants
    net.shutdownNow()  -> Forced shutdown, interrupts all workers
    ```

    ## Licensing

    jNetWorks uses a tiered licensing model:

    | Component       | License    | Source             |
    | --------------- | ---------- | ------------------ |
    | jnetworks-api   | Apache 2.0 | Maven Central      |
    | jnetworks-pcap  | Apache 2.0 | Maven Central      |
    | jnetworks-dpdk  | Commercial | maven.slytechs.com |
    | jnetworks-ntapi | Commercial | maven.slytechs.com |

    ### License Activation

    ```java
    // Automatic resolution (env, properties, files)
    Net.activateLicense();
    
    // Or explicit key
    Net.activateLicense("YOUR-LICENSE-KEY");
    ```

    License key resolution order:

    1. `JNETWORKS_LICENSE_KEY` environment variable
    2. `JNETWORKS_LICENSE_DIR` directory containing jnetworks.lic
    3. `-Djnetworks.license.key` system property
    4. `-Djnetworks.license.dir` system property
    5. `~/.jnetworks/jnetworks.lic`
    6. `/etc/jnetworks/jnetworks.lic`
    7. Embedded community key (feature-limited)

    ## Backend Selection

    ```java
    // Open source - included
    try (Net net = new PcapBackend()) { ... }
    
    // Commercial - requires license + dependency
    try (Net net = new DpdkBackend()) { ... }
    try (Net net = new NtapiBackend()) { ... }
    ```

    Same application code works with any backend. Swap one line to upgrade performance.

    ## Documentation

    - API Javadoc: https://docs.slytechs.com/jnetworks/api
    - User Guide: https://docs.slytechs.com/jnetworks/guide

    ## Support

    - Issues: https://github.com/slytechs-repos/jnetworks-sdk/issues
    - Commercial: support@slytechs.com
    - Website: https://www.slytechs.com

    ## License

    Open source components licensed under Apache 2.0. Commercial components require a valid license key.

    Copyright 2025 Sly Technologies Inc.
