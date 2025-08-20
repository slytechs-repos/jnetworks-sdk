# Napatech NTAPI and NTPL Command Reference

## Basic Assignment Commands

| Command Type | Command Example | Explanation |
|-------------|----------------|-------------|
| NTPL Basic Assignment | ```Assign[StreamId=1] = Port == 0``` | Assigns all packets from Port 0 to Stream ID 1 |
| NTPL Port Range | ```Assign[StreamId=2] = Port == (0,1,2)``` | Assigns packets from Ports 0, 1, and 2 to Stream ID 2 |
| NTPL with VLAN | ```Assign[StreamId=3] = VLAN == 100``` | Assigns all packets with VLAN ID 100 to Stream ID 3 |
| NTPL with Protocol | ```Assign[StreamId=4] = TCP``` | Assigns all TCP packets to Stream ID 4 |
| NTPL with IP | ```Assign[StreamId=5] = IP.Src == 192.168.1.100``` | Assigns packets from source IP 192.168.1.100 to Stream ID 5 |

## Hash Operations

| Command Type | Command Example | Explanation |
|-------------|----------------|-------------|
| Basic Hash Mode | ```Assign[StreamId=6] = HashMode == 1``` | Uses basic hash mode 1 for stream assignment |
| IP Hash | ```Assign[StreamId=7] = Hash(IP.Src, IP.Dst) % 4``` | Distributes based on IP address pair hash across 4 streams |
| TCP/IP Hash | ```Assign[StreamId=8] = Hash(TCP.Port, IP) % 8``` | Load balances TCP flows across 8 streams |
| Full Tuple Hash | ```Assign[StreamId=9] = Hash(IP.Src, IP.Dst, TCP.SrcPort, TCP.DstPort, IP.Proto) % 16``` | Complete 5-tuple hash distribution |
| VLAN Hash | ```Assign[StreamId=10] = Hash(VLAN.Id, IP.Src) % 4``` | Hash based on VLAN ID and source IP |
| Custom Hash Key | ```Assign[StreamId=11] = Hash(Slice(Frame[26:42])) % 8``` | Custom hash using frame slice |
| GTP Hash | ```Assign[StreamId=12] = Hash(GTP.TEID) % 4``` | Load balance GTP tunnels |
| Inner Packet Hash | ```Assign[StreamId=13] = Hash(Inner.IP.Src, Inner.IP.Dst) % 8``` | Hash based on tunneled packet IPs |
| Symmetric Hash | ```Assign[StreamId=14] = SymmetricHash(IP.Src, IP.Dst) % 4``` | Bidirectional flow to same stream |

## Advanced Operations

| Command Type | Command Example | Explanation |
|-------------|----------------|-------------|
| Color with Hash | ```Assign[StreamId=15; Color=Blue] = Hash(IP.Src) % 4``` | Combines coloring with hash distribution |
| Priority Hash | ```Assign[Priority=1; StreamId=16] = Hash(TCP.Port) % 8``` | High-priority hash-based distribution |
| Conditional Hash | ```Assign[StreamId=17] = TCP AND Hash(IP.Src, TCP.Port) % 4``` | Hash distribution for TCP only |
| Layer-2 Hash | ```Assign[StreamId=18] = Hash(MAC.Src, MAC.Dst) % 4``` | Distribution based on MAC addresses |
| Multi-field Slice | ```Assign[StreamId=19] = Hash(Slice(Frame[14:34],Frame[42:46])) % 8``` | Hash multiple frame sections |
| Payload Hash | ```Assign[StreamId=20] = Hash(Frame[TCP.DataOffset:TCP.DataOffset+16]) % 4``` | Hash based on TCP payload |
| Custom Offset Hash | ```Assign[StreamId=21] = Hash(Slice(Frame[EtherOffset+16:EtherOffset+20])) % 8``` | Hash using dynamic offset |
| RSS Hash | ```Assign[StreamId=22] = RSSHash() % 8``` | Uses RSS algorithm for distribution |
| Weight-based Hash | ```Assign[StreamId=23] = WeightedHash(IP.Src, IP.Dst)[1,2,1,2]``` | Hash with weighted distribution |
| Tunnel-aware Hash | ```Assign[StreamId=24] = TunnelHash(IP, GTP) % 4``` | Smart tunnel-aware distribution |

## Usage Notes

### Hash Modes
- Basic hash modes (1-4) provide predefined combinations
- Custom hash functions allow specific field selection
- Symmetric hashing ensures bidirectional flows map to same stream
- RSS hash compatible with NIC RSS configurations

### Hash Distribution
- Modulo (%) determines number of streams for distribution
- Values should be power of 2 for optimal distribution
- Consider using weighted distribution for uneven capacity

### Best Practices
- Use symmetric hash for bidirectional analysis
- Combine with colors for advanced classification
- Consider priority when using multiple hash rules
- Frame slicing allows custom protocol support

### Performance Considerations
- Hash calculations impact throughput
- Simpler hash combinations perform better
- Consider hardware offload capabilities
- Balance between distribution and processing overhead