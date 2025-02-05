**TCSNP (Transmittable Common Server Network Protocol)  
Documentation Version 1.0**  
*(Protocol for Anonymous Decentralized Web 3.0 Network with Onion Routing Integration)*  

---

### **1. Introduction**  
TCSNP is a next-generation protocol that combines decentralization, cryptographic anonymity, and censorship resistance. Key improvements include:  
- **Full Anonymization**: Metadata obfuscation through Onion routing and temporary identifiers.  
- **Blockchain-Free Design**: Use of DHT with dynamic replication and blind reputation metrics.  
- **Traffic Analysis Resistance**: Packet obfuscation and node pseudonymization.  

---

### **2. Architecture**  
#### **2.1. System Components**  
- **Participant Nodes**:  
  - **Guards**: Nodes with uptime > 95% and reputation > 90 (similar to Tor Guards).  
  - **Relays**: Intermediate nodes for Onion routing.  
  - **Service Nodes**: Store data and process requests (anonymous server analogs).  
- **Onion-DHT**: A modified DHT divided into three layers:  
  - **Layer 1**: Stores public keys of Guards and Relays.  
  - **Layer 2**: Anonymous hashes of Service Nodes (no geolocation binding).  
  - **Layer 3**: Sharded metadata with obfuscated records (XOR encoding + salt).  

---

### **3. Node Joining Process**  
#### **3.1. Anonymous Initialization**  
1. **Temporary Identifier Generation**:  
   - Each node generates a one-time key pair (Ed25519) upon every restart.  
   - The public key is hashed via HMAC-BLAKE3 with a secret salt → `temp_id`.  

2. **Onion-Discovery**:  
   - Requests are broadcast through a chain of 3 random Guards (Onion-encrypted).  
   - Each Guard removes its encryption layer and forwards the packet.  

3. **DHT Registration**:  
   - The node sends a signed request to Onion-DHT with `temp_id` and encrypted metadata:  
     ```json
     {
       "temp_id": "a1b2c3...",  
       "enc_metadata": "AES-256(GCM){'protocols': ['TCP', 'UDP'], 'region': 'XX'}",
       "onion_route": ["Guard1_pubkey", "Guard2_pubkey", "Relay_pubkey"]  
     }
     ```  
   - Reputation is calculated via blind signatures: nodes prove reliability without revealing activity history.  

---

### **4. Onion Routing**  
#### **4.1. Chain Formation**  
1. **Node Selection**:  
   - The client randomly selects 3 Guards from DHT (Layer 1) and 2 Relays (Layer 2).  
   - Chain: `Client → Guard1 → Relay1 → Guard2 → Relay2 → Service Node`.  

2. **Multi-Layer Encryption**:  
   - Each packet is encrypted in reverse order (from service to client):  
     ```python
     # Example for 3 hops:
     packet = Encrypt(Relay2_pubkey, 
               Encrypt(Guard2_pubkey, 
                 Encrypt(Relay1_pubkey, 
                   Encrypt(Guard1_pubkey, payload))))
     ```  
   - Each node decrypts its layer and learns the next hop.  

#### **4.2. Metadata Protection**  
- **Fixed Packet Size**: All packets are padded to 1024 bytes with random data.  
- **Dummy Traffic**: Nodes periodically send empty encrypted packets to mask real activity.  

---

### **5. Anonymous Data Exchange**  
#### **5.1. Garlic Messaging Protocol**  
- **Nested Messages**: Multiple messages for different recipients are packed into a single packet (similar to I2P).  
- **Session Aliases**: A unique identifier (`session_id`) is generated for each transaction, unrelated to `temp_id`.  

#### **5.2. Blind DHT Queries**  
- **Intent Obfuscation**:  
  - The client sends a hashed query masked via a bloom filter to the DHT.  
  - Nodes return all records matching the filter, even if they don’t precisely match the query.  
- **Query Encryption**: Search parameters (e.g., geolocation) are replaced with ranges (`EU:*` → `[EU:00...EU:FF]`).  

---

### **6. Reputation System**  
#### **6.1. Blind Reputation Tokens**  
- **Accrual Mechanism**:  
  - Upon successful data transfer, the recipient signs a blind token (Chaumian-style).  
  - The node exchanges tokens for reputation points via a mixing service without revealing the source.  
- **Reputation Degradation**:  
  - Automatic point reduction if > 5% of requests are rejected within an hour.  
  - Nodes with reputation < 30 are excluded from the DHT for 24 hours.  

#### **6.2. Proof of Work (PoW)**  
- To prevent Sybil attacks:  
  - New nodes must solve a cryptographic puzzle (Hashcash-style) before DHT registration.  
  - Puzzle difficulty is dynamically adjusted based on network load.  

---

### **7. Security and Anonymity**  
#### **7.1. Correlation Resistance**  
- **Chain Rotation**: Onion routes are rebuilt every 10 minutes.  
- **Stream Separation**: Data and control commands are transmitted through different chains.  

#### **7.2. Anti-Exfiltration**  
- **Session Reset**: All keys (session, identifiers) are deleted after > 5 minutes of inactivity.  
- **Quantum-Safe Algorithms**: Fallback encryption using NTRUEncrypt for critical data.  

---

### **8. Optimizations**  
- **Local Micro-Networks**: Nodes within the same subnet (e.g., Wi-Fi mesh) exchange data directly via the Noise Protocol Framework.  
- **Predictable Latency**: Artificial jitter (0–100 ms) is added to mask real location.  

---

### **9. Use Cases**  
- **Anonymous Datagram Messenger**:  
  - Messages are fragmented and transmitted through different chains.  
  - The recipient reassembles fragments using `session_id`.  
- **Censorship-Resistant CDN**:  
  - Data is stored encrypted on Service Nodes with geographic distribution.  
  - Decryption keys are transmitted via Garlic Messaging.  

---
