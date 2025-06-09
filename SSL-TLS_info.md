#  What is SSL/TLS?

### **SSL (Secure Sockets Layer) and TLS (Transport Layer Security) are cryptographic protocols that:**

* Provide **secure communication** over a network.
* Encrypt data between **client (e.g., browser)** and **server (e.g., website)**.
* Protect against **eavesdropping**, **tampering**, and **forgery**.

> 🔁 SSL is the old name. **TLS** is the modern protocol. TLS 1.0 was released as an upgrade to SSL 3.0. SSL is deprecated.

---

##  Why Do We Need SSL/TLS?

Before SSL/TLS:

* **Data was sent in plaintext** across the internet.
* Anyone sniffing the traffic (like on public Wi-Fi or through ISP routers) could:

  * Read login credentials
  * Steal credit card numbers
  * Alter the data
  * Impersonate websites (phishing)

SSL/TLS was introduced to:

1. **Encrypt** communication.
2. **Authenticate** the server (and optionally the client).
3. **Ensure data integrity** (detect tampering).

---

## ⚙️ How Does SSL/TLS Work (High-Level)?

### 1. **TLS Handshake** (Before actual data transfer)

This is a negotiation phase between the client and server:

1. **Client Hello**

   * Client sends a request to server with:

     * TLS version supported
     * List of supported ciphers
     * A random number

2. **Server Hello**

   * Server responds with:

     * TLS version
     * Chosen cipher suite
     * Server’s **digital certificate** (contains public key)
     * Another random number

3. **Certificate Verification**

   * Client checks the server's certificate:

     * Issued by a trusted Certificate Authority (CA)?
     * Not expired?
     * Correct domain name?

4. **Key Exchange**

   * Using **RSA** or **ECDHE**, both parties derive a **shared session key** (symmetric key).
   * This key is used for encrypting the actual data.

5. **Finished**

   * A message is sent encrypted with the shared key to confirm setup.
   * If everything is valid, communication starts using encryption.

---

##  SSL/TLS Concepts & Terms

| Term                                | Description                                                                         |
| ----------------------------------- | ----------------------------------------------------------------------------------- |
| **Certificate Authority (CA)**      | Trusted third party that issues digital certificates. e.g., Let's Encrypt, DigiCert |
| **Digital Certificate**             | Identity proof of the server. Contains server name, public key, CA signature        |
| **Public Key / Private Key**        | Asymmetric encryption pair (used in handshake phase)                                |
| **Symmetric Key**                   | Shared secret key used to encrypt/decrypt actual data                               |
| **Cipher Suite**                    | A set of algorithms used for key exchange, encryption, MAC, etc.                    |
| **PKI (Public Key Infrastructure)** | System of CA, certificates, keys, revocation, etc.                                  |
| **HTTPS**                           | HTTP over TLS (secure HTTP)                                                         |
| **Handshake Protocol**              | Initial phase of TLS where negotiation and key exchange happen                      |
| **Record Protocol**                 | Protocol to transmit encrypted data                                                 |
                 
---

##  TLS Versions (History)

| Version           | Notes                                                                    |
| ----------------- | ------------------------------------------------------------------------ |
| SSL 2.0 / 3.0     | Deprecated (insecure)                                                    |
| **TLS 1.0 / 1.1** | Deprecated (as of 2020)                                                  |
| **TLS 1.2**       | Still widely used                                                        |
| **TLS 1.3**       | Latest version. Simpler, faster, more secure. Removes old, weak ciphers. |

---

##  Types of Certificates

* **DV (Domain Validation)** – basic, automated (e.g., Let's Encrypt)
* **OV (Organization Validation)** – manual verification of org details
* **EV (Extended Validation)** – highest trust level (green bar in some browsers)
* **Wildcard Certificate** – covers all subdomains (e.g., `*.example.com`)
* **SAN (Subject Alternative Name)** – covers multiple domains in one cert

---

<div align="center">

# **SSL/TLS Overview & Tools**

</div>

---

##  1. **OpenSSL**

### 🔹 What is it?

* A **command-line toolkit** and a **cryptographic library** used to generate private keys, CSRs (Certificate Signing Requests), self-signed certs, verify certificates, and more.

### 🔹 Common Use Cases

* Creating private keys and CSRs for getting certs from a CA.
* Generating self-signed certificates (useful in dev/testing).
* Testing TLS/SSL connections.
* Inspecting and converting certificate formats (PEM, DER, PKCS12, etc.)

### 🔹 Common Commands

```bash
# Generate a private key
openssl genrsa -out server.key 2048

# Create a CSR
openssl req -new -key server.key -out server.csr

# Generate a self-signed cert
openssl req -x509 -nodes -days 365 -newkey rsa:2048 \
  -keyout server.key -out server.crt

# Check certificate
openssl x509 -in server.crt -text -noout

# Test TLS connection
openssl s_client -connect example.com:443
````

### 🔹 Use in Kubernetes

* Used manually to generate TLS secrets for internal services.
* Sometimes used in scripts for creating local CA and certs for mTLS or custom services.

---

##  2. **Certbot**

### 🔹 What is it?

* A **CLI tool** from the Electronic Frontier Foundation (EFF) to automate getting SSL certificates from **Let’s Encrypt**.

### 🔹 What it does

* Handles:

  * Domain validation (HTTP-01 or DNS-01)
  * Issuing certs from Let’s Encrypt
  * Auto-renewal (via cron or systemd timers)

### 🔹 Example Usage

```bash
# Automatically get and install cert for Apache/Nginx
sudo certbot --nginx
sudo certbot --apache

# Just obtain the cert (no auto config)
sudo certbot certonly --manual --preferred-challenges dns -d example.com

# Renew all installed certs
sudo certbot renew
```

### 🔹 Use in Kubernetes

* Not typically used directly in production K8s, but useful:

  * In development
  * For non-K8s services like a legacy reverse proxy
* Inside K8s, **cert-manager** (see below) is preferred.

---

## 🛡️ 3. **Let’s Encrypt**

### 🔹 What is it?

* A **free, automated, open Certificate Authority (CA)**.
* Issues **Domain Validation (DV)** certificates.

### 🔹 How it works

* Validates domain ownership via:

  * **HTTP-01 challenge** (serve a file over HTTP)
  * **DNS-01 challenge** (create a TXT DNS record)

### 🔹 Key Characteristics

* Certs are **free**, **valid for 90 days**, and **renewable**.
* API-driven and used with Certbot or cert-manager.

### 🔹 Use in Kubernetes

* Combined with **cert-manager** to automatically issue and renew TLS certs for services exposed via Ingress.
* Ideal for public-facing apps with HTTPS via Ingress controllers.

---

##  4. **cert-manager (Kubernetes)**

### 🔹 What is it?

* A **Kubernetes-native controller** that automates:

  * Certificate issuance
  * Renewal
  * Managing secrets containing TLS certs

### 🔹 Key Features

* Supports **Let’s Encrypt**, **HashiCorp Vault**, **AWS ACM**, **private CAs**, etc.
* Works via `Certificate`, `Issuer`, and `ClusterIssuer` CRDs.
* Monitors expiration and auto-renews certs.

### 🔹 Use Cases in Kubernetes

| Scenario          | cert-manager Role                                        |
| ----------------- | -------------------------------------------------------- |
| HTTPS via Ingress | Automatically issues certs for domains via Let’s Encrypt |
| mTLS              | Manages certs for client/server mutual auth              |
| Internal services | Provides certs via internal PKI or CA                    |
| Wildcard certs    | Supports DNS-01 challenge with DNS providers             |

### 🔹 How It Works in K8s

1. Install cert-manager using Helm or `kubectl apply`.
2. Create a `ClusterIssuer` or `Issuer` (e.g., Let’s Encrypt HTTP-01 or DNS-01).
3. Create a `Certificate` resource.
4. cert-manager handles issuance & stores the cert in a Kubernetes Secret.

---

## ☁️ 5. **AWS Certificate Manager (ACM)**

### 🔹 What is it?

* A **managed service by AWS** that provisions and manages SSL/TLS certs for AWS resources.

### 🔹 Use Cases

* Assigning HTTPS to:

  * **Elastic Load Balancers (ELBs)**
  * **CloudFront**
  * **API Gateway**
* Integrated with **Route 53** for DNS validation.

### 🔹 Limitations

* Certificates **can’t be exported**, so can’t be directly used with:

  * In-cluster pods
  * Custom K8s services that require private key access

### 🔹 How It Works in EKS

* Typical usage:

  1. Create a cert in ACM.
  2. Attach to **AWS LoadBalancer** via Ingress annotations in EKS.
  3. ACM handles renewals automatically.

### 🔹 Example in Kubernetes (with ALB Ingress Controller)

```yaml
annotations:
  alb.ingress.kubernetes.io/certificate-arn: arn:aws:acm:region:acct:certificate/xyz
```

##  Typical Kubernetes TLS Workflows

### ✅ Public HTTPS Ingress with Let’s Encrypt

* Use cert-manager with a Let’s Encrypt `ClusterIssuer`
* Automatically secures apps via NGINX or ALB Ingress

### ✅ Internal mTLS between services

* Use cert-manager with a custom CA or Vault
* Enables secure service-to-service communication

### ✅ TLS with AWS Load Balancer (ACM)

* Use AWS ACM + ALB Ingress
* TLS terminated at the AWS load balancer

---
<div align="center">
  
# **Most Common SSL/TLS File Types and Their Roles**

</div>


| Extension                  | File Type                   | Role                                                                                                         |
| -------------------------- | --------------------------- | ------------------------------------------------------------------------------------------------------------ |
| **.crt** or **.cer**       | Certificate (X.509)         | The **public certificate** issued by a CA (or self-signed). Shared with clients.                             |
| **.key**                   | Private Key                 | The **private key** that pairs with the `.crt`. Must be kept secret. Used to decrypt incoming data.          |
| **.csr**                   | Certificate Signing Request | Generated when requesting a certificate. Contains public key + domain info (no private key). Sent to CA.     |
| **.tls.key**, **.tls.crt** | Kubernetes TLS secret files | Used in `tls`-type Kubernetes Secrets. These are just renamed `.key` and `.crt`.                             |

---

##  Role of Each File in SSL/TLS Setup

| File                      | Description                     | Usage Example                                               |
| ------------------------- | ------------------------------- | ----------------------------------------------------------- |
| `server.key`              | Private Key                     | Used by server to decrypt the symmetric session key         |
| `server.csr`              | CSR file                        | Sent to a CA to issue a certificate                         |
| `server.crt`              | Server Certificate              | Sent to client during TLS handshake                         |
| `ca.crt`                  | CA Certificate                  | Used to verify the authenticity of the server’s certificate |



---
<div align="center">
  
#  **FULL WORKFLOW**

</div>

---

You want:

* Secure communication (HTTPS)
* Authentication (users know they’re talking to *your* server)
* Confidentiality (nobody can read the data in transit)

You’ll use:

* `example.com.key` → the **private key**
* `example.com.crt` → the **public certificate** issued by a CA

---

##  Step-by-Step Flow: From Certificate Creation to HTTPS in Action

---

###  **1. Generate the Private Key (`.key`)**

This key stays on the **server only**. It’s used to:

* Decrypt messages
* Prove ownership of the domain

```bash
openssl genrsa -out example.com.key 2048
```

---

###  **2. Create a Certificate Signing Request (CSR)**

This file is generated using your private key and contains:

* Your public key
* Your domain name (e.g., `example.com`)
* Some metadata (organization, country, etc.)

```bash
openssl req -new -key example.com.key -out example.com.csr
```

 **You send this `.csr` file to a Certificate Authority (CA)** like Let’s Encrypt, AWS ACM, or DigiCert.

---

### 🏛️ **3. CA Verifies and Issues the Certificate (`.crt`)**

The CA:

* Verifies that you own `example.com` (via DNS or HTTP challenge)
* Signs your CSR with their **trusted private key**
* Gives you back a **signed certificate** → `example.com.crt`

This file contains:

* Your public key
* Your domain name
* CA’s digital signature
* Expiration date

---

###  **4. You Configure Your Web Server (e.g., Nginx, Apache)**

Example: **Nginx config**

```nginx
server {
    listen 443 ssl;
    server_name example.com;

    ssl_certificate     /etc/ssl/certs/example.com.crt;
    ssl_certificate_key /etc/ssl/private/example.com.key;

    location / {
        proxy_pass http://app;
    }
}
```

* `example.com.crt` = proves your identity to visitors
* `example.com.key` = decrypts client messages (never shared)

---

### 🌐 **5. Client Connects (TLS Handshake Happens)**

Here's what happens under the hood:

#### 🔄 TLS Handshake Flow:

1. **Client (Browser)**:
   Sends `ClientHello` with supported ciphers + TLS version.

2. **Server**:
   Sends back `ServerHello` + its `example.com.crt`

3. **Client Validates Certificate**:

   * Checks expiration, domain, and CA trust
   * Verifies the signature of `.crt` using CA’s public key (which is in OS/browser trust store)

4. **Client Generates Session Key** (for symmetric encryption)

   * Encrypts it using server’s **public key** (from `.crt`)
   * Sends it to the server

5. **Server Decrypts Session Key** using **`example.com.key`**

6. ✅ **Secure channel established**

   * From now on, client and server communicate using symmetric encryption (fast and secure)

---

### 🛡 Summary: Roles of `.crt` and `.key`

| File              | Who Uses It                  | Role                                                 |
| ----------------- | ---------------------------- | ---------------------------------------------------- |
| `example.com.key` | Server only (must be secret) | Decrypts data sent by client, proves server identity |
| `example.com.crt` | Sent to client (browser)     | Contains public key + identity, signed by trusted CA |

---

###  Optional: Test It Locally with OpenSSL

You can run a simple test server:

```bash
openssl s_server -cert example.com.crt -key example.com.key -accept 4433
```

Then connect with:

```bash
openssl s_client -connect localhost:4433
```

---

##  BONUS: In Kubernetes

You'd create a TLS secret:

```bash
kubectl create secret tls my-tls \
  --cert=example.com.crt \
  --key=example.com.key
```

Use it in an Ingress:

```yaml
tls:
  - hosts:
      - example.com
    secretName: my-tls
```

---



