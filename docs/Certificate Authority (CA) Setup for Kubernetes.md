📜 Certificate Authority (CA) Setup for Kubernetes

To securely provision Kubernetes components, we first need to generate a Certificate Authority (CA). This CA will be used to sign certificates for etcd, kube-apiserver, and other control plane and worker node components.
---
🔧 Step 1: Create OpenSSL Configuration
![image](https://github.com/user-attachments/assets/c3fec232-1c11-418c-9bef-3f8ac1025378)

### Create a file named CAopenssl.cnf with the following contents:
```bash
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
```
🔐 Step 2: Generate the Private Key and CA Certificate
```bash
openssl genpkey -algorithm RSA -out ca.key -pkeyopt rsa_keygen_bits:4096
openssl req -x509 -new -nodes -key ca.key -sha256 -days 1000   -out ca.crt -config CAopenssl.cnf -extensions v3_ca
```
![image](https://github.com/user-attachments/assets/9ab08416-9c7b-40b8-9abd-2d55936406c8)
