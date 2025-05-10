#📜 Certificate Authority (CA) Setup for Kubernetes

To securely provision Kubernetes components, we first need to generate a Certificate Authority (CA). This CA will be used to sign certificates for etcd, kube-apiserver, and other control plane and worker node components.
---
##🔧 Step 1: Create OpenSSL Configuration

### Create a file named CAopenssl.cnf with the following contents:
```ini
[req]
distinguished_name = req_distinguished_name
x509_extensions = v3_ca
prompt = no

[req_distinguished_name]
C = BD
ST = Dhaka
L = Dhaka
O = Kubernetes
OU = CA
CN = kubernetes-ca

[v3_ca]
subjectKeyIdentifier = hash
authorityKeyIdentifier = keyid:always,issuer
basicConstraints = critical, CA:true
keyUsage = critical, keyCertSign, cRLSign
---
You can use the following command to quickly create it:

cat <<EOF > CAopenssl.cnf
[req]
distinguished_name = req_distinguished_name
x509_extensions = v3_ca
prompt = no

[req_distinguished_name]
C = BD
ST = Dhaka
L = Dhaka
O = Kubernetes
OU = CA
CN = kubernetes-ca

[v3_ca]
subjectKeyIdentifier = hash
authorityKeyIdentifier = keyid:always,issuer
basicConstraints = critical, CA:true
keyUsage = critical, keyCertSign, cRLSign
EOF

🔐 Step 2: Generate the Private Key

Generate a 4096-bit RSA private key for the CA:

openssl genpkey -algorithm RSA -out ca.key -pkeyopt rsa_keygen_bits:4096

📄 Step 3: Generate the Self-Signed CA Certificate

Using the key and config file, generate a self-signed certificate valid for 1000 days:

openssl req -x509 -new -nodes -key ca.key \
  -sha256 -days 1000 \
  -out ca.crt \
  -config CAopenssl.cnf \
  -extensions v3_ca

✅ Output Files

File

Description

ca.key

Private key of the CA

ca.crt

Self-signed root certificate

CAopenssl.cnf

Configuration used for CA creation
