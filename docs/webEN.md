**TCSNP (Transmittable Common Server Network Protocol)**

---

### **1. Introduction**  
TCSNP is a next-generation protocol that combines decentralization, cryptographic anonymity, and censorship resistance. Key improvements include:  
- **Complete Anonymization**: Hiding metadata through Onion routing and temporary identifiers.  
- **Blockchain Abandonment**: Utilizing DHT with dynamic replication and blind reputation metrics.  
- **Traffic Analysis Resistance**: Packet obfuscation and node pseudonymization.  
- **P2P Architecture**: All nodes act as both servers and clients, ensuring full decentralization and equality among nodes.  
- **Dual Reputation System**: Reputation based on connections and reputation regulated by users.

---

### **2. Architecture**  
#### **2.1. System Components**  
- **Participant Nodes**:  
  - **Guards**: Nodes with uptime > 95%, reputation > 90 (analogous to Tor Guards).  
  - **Relays**: Intermediate nodes for Onion routing.  
  - **Service Nodes**: Store data and process requests (anonymous server analogs).  
- **Onion-DHT**: A modified DHT divided into three layers:  
  - **Layer 1**: Stores public keys of guards and relays.  
  - **Layer 2**: Anonymous hashes of service nodes (without geolocation binding).  
  - **Layer 3**: Sharded metadata with obfuscated records (XOR encoding + salt).

---

### **3. Node Joining**  
#### **3.1. Anonymous Initialization**  
1. **Temporary Identifier Generation**:  
   - Each node generates a one-time key pair (Ed25519) on every restart.  
   - The public key is hashed via HMAC-BLAKE3 with a secret salt → `temp_id`.

2. **Onion Discovery**:  
   - Requests are sent via a chain of 3 random guards (Onion encryption).  
   - Each guard removes its encryption layer and forwards the packet.

3. **DHT Registration**:  
   - The node sends a signed request to the Onion-DHT with `temp_id` and encrypted metadata:  
     ```json
     {
       "temp_id": "a1b2c3...",  
       "enc_metadata": "AES-256(GCM){'protocols': ['TCP', 'UDP'], 'region': 'XX'}",
       "onion_route": ["Guard1_pubkey", "Guard2_pubkey", "Relay_pubkey"]
     }
     ```  
   - Reputation is calculated through a blind signature: the node proves its reliability without revealing its activity history.

---

### **4. Onion Routing**  
#### **4.1. Chain Formation**  
1. **Node Selection**:  
   - The client randomly selects 3 guards from DHT (Layer 1) and 2 relays (Layer 2).  
   - The chain: `Client → Guard1 → Relay1 → Guard2 → Relay2 → Service Node`.

2. **Multilayer Encryption**:  
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
- **Fake Traffic**: Nodes periodically send empty encrypted packets to mask real activity.

---

### **5. Anonymous Data Exchange**  
#### **5.1. Garlic Messaging Protocol**  
- **Nested Messages**: Multiple messages for different recipients are packed into a single packet (similar to I2P).  
- **Session Pseudonyms**: A unique identifier (`session_id`) is generated for each transaction, unrelated to `temp_id`.

#### **5.2. Blind DHT Queries**  
- **Search Without Revealing Intentions**:  
  - The client sends a hash of the request, masked through a bloom filter.  
  - Nodes return all records matching the filter, even if they don't exactly match the query.  
- **Query Encryption**: Search parameters (e.g., geolocation) are replaced with ranges (`EU:*` → `[EU:00...EU:FF]`).

---

### **6. Reputation System**  
#### **6.1. Blind Reputation Tokens**  
- **Accrual Mechanism**:  
  - Upon successful data transfer, the recipient signs a blind token (Chaumian scheme).  
  - The node exchanges tokens for reputation points via a mixing service, without revealing the source.  
- **Reputation Degradation**:  
  - Automatic point reduction if more than 5% of requests are declined per hour.  
  - Nodes with reputation < 30 are excluded from DHT for 24 hours.

#### **6.2. User Reputation**  
- **Node Rating by Users**:  
  - Users can rate nodes based on the quality of services provided.  
  - Ratings are stored in encrypted form and factored into the node's overall reputation.  
- **Voting Mechanism**:  
  - Users can vote for or against a node using cryptographically signed tokens.  
  - Votes are weighted according to the reputation of the voting users.

#### **6.3. Proof of Work (PoW)**  
- To prevent Sybil attacks:  
  - New nodes must solve a cryptographic puzzle (Hashcash-style) before registering in DHT.  
  - The difficulty of the puzzle is dynamically adjusted based on network load.

---

### **7. Security and Anonymity**  
#### **7.1. Correlation Protection**  
- **Route Rotation**: Onion routes are restructured every 10 minutes.  
- **Stream Separation**: Data and control messages are sent through separate chains.

#### **7.2. Anti-Exfiltration**  
- **Session Reset**: All keys (session, identifiers) are deleted after 5 minutes of inactivity.  
- **Quantum-Safe Algorithms**: Backup encryption based on NTRUEncrypt for critical data.

---

### **8. Optimizations**  
- **Local Micro-Networks**: Nodes in the same subnet (e.g., Wi-Fi mesh) exchange data directly via the Noise Protocol Framework.  
- **Predictable Latency**: Artificial jitter (0–100 ms) is added to mask the real location.  
