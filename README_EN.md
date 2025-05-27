# NodeCrypt

🌐 **[中文版 README](README.md)**

One-click deployment to Cloudflare supported

[![Deploy to Cloudflare Workers](https://deploy.workers.cloudflare.com/button?projectName=NodeCrypt)](https://deploy.workers.cloudflare.com/?url=https://github.com/shuaiplus/NodeCrypt)

## 📝 Project Introduction

NodeCrypt is a truly end-to-end encrypted chat system that implements a complete zero-knowledge architecture. The entire system design ensures that servers, network intermediaries, and even system administrators cannot access any plaintext message content. All encryption and decryption operations are performed locally on the client side, with the server serving only as a blind relay for encrypted data.

### System Architecture
- **Frontend**: ES6+ modular JavaScript, no framework dependencies
- **Backend**: Cloudflare Workers + Durable Objects
- **Communication**: Real-time bidirectional WebSocket communication
- **Build**: Vite modern build tool

## 🔐 Zero-Knowledge Architecture Design

### Core Principles
- **Server Blind Relay**: The server can never decrypt message content, only responsible for encrypted data relay
- **No Database Storage**: The system does not use any persistent storage; all data exists only temporarily in memory
- **End-to-End Encryption**: Messages are encrypted from sender to receiver throughout the entire process; no intermediate node can decrypt them
- **Forward Secrecy**: Even if keys are compromised, historical messages cannot be decrypted because there are no historical messages at all
- **Anonymous Communication**: Users do not need to register real identities; supports temporary anonymous chat

### Privacy Protection Mechanisms

- **Real-time Member Notifications**: The room online list is completely transparent; any member joining or leaving will notify all members in real-time
- **No Historical Messages**: Newly joined users cannot see any historical chat records
- **Private Chat Encryption**: Clicking on a user's avatar can initiate end-to-end encrypted private conversations that are completely invisible to other room members

### Room Password Mechanism

Room passwords serve as **key derivation factors** in end-to-end encryption: `Final Shared Key = ECDH_Shared_Key XOR SHA256(Room Password)`

- **Password Error Isolation**: Rooms with different passwords cannot decrypt each other's messages
- **Server Blind Spot**: The server can never know the room password

### Three-Layer Security System

#### Layer 1: RSA-2048 Server Identity Authentication
- Server generates temporary RSA-2048 key pairs on startup, automatically rotated every 24 hours
- Client verifies server public key on connection to prevent man-in-the-middle attacks
- Private keys exist only in server memory and are never persistently stored

#### Layer 2: ECDH-P384 Key Agreement
- Each client generates independent elliptic curve key pairs (P-384 curve)
- Establishes shared keys through Elliptic Curve Diffie-Hellman key exchange protocol
- Each client has an independent encrypted channel with the server

#### Layer 3: Hybrid Symmetric Encryption
- **Server Communication**: Uses AES-256-CBC to encrypt control messages between client and server
- **Client Communication**: Uses ChaCha20 to encrypt actual chat content between clients
- Each message uses independent initialization vectors (IV) and nonces

## 🔄 Complete Encryption Process

```mermaid
sequenceDiagram
    participant C as Client
    participant S as Server
    participant O as Other Clients

    Note over C,S: Phase 1: Server Identity Authentication (RSA-2048)
    C->>S: WebSocket Connection
    S->>C: RSA-2048 Public Key
    
    Note over C,S: Phase 2: Client-Server Key Exchange (P-384 ECDH)
    C->>S: P-384 ECDH Public Key
    S->>C: P-384 Public Key + RSA Signature
    Note over C: Verify RSA signature and derive AES-256 key
    Note over S: Derive AES-256 key from P-384 ECDH
    
    Note over C,S: Phase 3: Room Authentication
    C->>S: Join Request (Room Hash, AES-256 encrypted)
    Note over S: Add client to room/channel
    S->>C: Member List (Other client IDs, encrypted)
    
    Note over C,O: Phase 4: Inter-Client Key Exchange (Curve25519)
    Note over C: Generate Curve25519 key pair for each member
    C->>S: Curve25519 Public Key Bundle (AES-256 encrypted)
    S->>O: Forward Client C's public key
    O->>S: Return other clients' Curve25519 public keys
    S->>C: Forward other clients' public keys
    
    Note over C,O: Phase 5: Password-Enhanced Key Derivation
    Note over C: Client Key = ECDH_Curve25519(own private key, other's public key) XOR SHA256(password)
    Note over O: Client Key = ECDH_Curve25519(own private key, other's public key) XOR SHA256(password)
    
    Note over C,O: Phase 6: Identity Authentication
    C->>S: Username (ChaCha20 encrypted with client key)
    S->>O: Forward encrypted username
    O->>S: Username (ChaCha20 encrypted with client key)
    S->>C: Forward encrypted username
    Note over C,O: Both clients now verify each other's identity
    
    Note over C,O: Phase 7: Secure Message Transmission (Double-layer encryption)
    Note over C: 1. ChaCha20 encrypt(message content)<br/>2. AES-256 encrypt(transport layer wrapper)
    C->>S: Double-layer encrypted message
    Note over S: Decrypt AES-256 transport layer<br/>Extract ChaCha20 encrypted data<br/>Cannot decrypt message content
    S->>O: Forward ChaCha20 encrypted data
    Note over O: Decrypt AES-256 transport layer<br/>ChaCha20 decrypt to get message content
```

## 🛠️ Technical Implementation

- **Web Cryptography API**: Native browser encryption implementation with hardware acceleration
- **elliptic.js**: Elliptic curve cryptography library implementing Curve25519 and P-384
- **aes-js**: Pure JavaScript AES implementation supporting multiple modes
- **js-chacha20**: JavaScript implementation of ChaCha20 stream cipher
- **js-sha256**: SHA-256 hash algorithm implementation

## 🚀 Deployment Instructions

### Local Development
After cloning the project and installing dependencies, use `npm run dev` to start the development server.

Use `npm run deploy` to deploy to Cloudflare Workers.

## 🔬 Security Verification

### Encryption Process Verification
Users can observe the complete encryption and decryption process through browser developer tools to verify that messages are indeed encrypted during transmission.

### Network Traffic Analysis
Network packet capture tools can verify that all WebSocket transmitted data is unreadable encrypted content.

### Code Security Audit
All encryption-related code is completely open source, using standard cryptographic algorithms. Security researchers are welcome to conduct independent audits.

## ⚠️ Security Recommendations

- **Use Strong Room Passwords**: Room passwords directly affect end-to-end encryption strength; complex passwords are recommended
- **Password Confidentiality**: If a room password is leaked, all communication content in that room may be decrypted
- **Use Latest Modern Browsers**: Ensure security and performance of cryptographic APIs

## 🤝 Security Contributions

Security researchers are welcome to report vulnerabilities and conduct security audits. Critical security issues will be fixed within 24 hours.

## 📄 Open Source License

This project uses the ISC open source license.

## ⚠️ Disclaimer

This project is for educational and technical research purposes only and must not be used for any illegal or criminal activities. Users should comply with the relevant laws and regulations of their country and region. The project author assumes no legal responsibility for any consequences arising from the use of this software. Please use this project legally and compliantly.

---

**NodeCrypt** - True End-to-End Encrypted Communication 🔐

*"In the digital age, encryption is the last line of defense for privacy"*
