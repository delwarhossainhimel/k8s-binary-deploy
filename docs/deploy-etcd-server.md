![A computer screen shot of a program AI-generated content may be
incorrect.](media/image1.png){width="6.5in"
height="2.9604166666666667in"}

I will generate CA and create a CA server

#vim CAopenssl.cnf

\[req\]

distinguished_name = req_distinguished_name

x509_extensions = v3_ca

prompt = no

\[req_distinguished_name\]

C = BD

ST = Dhaka

L = Dhaka

O = Kubernetes

OU = CA

CN = kubernetes-ca

\[v3_ca\]

subjectKeyIdentifier = hash

authorityKeyIdentifier = keyid:always,issuer

basicConstraints = critical, CA:true

keyUsage = critical, keyCertSign, cRLSign

openssl genpkey -algorithm RSA -out ca.key -pkeyopt rsa_keygen_bits:4096

openssl req -x509 -new -nodes -key ca.key -sha256 -days 1000 -out ca.crt
-config CAopenssl.cnf -extensions v3_ca

![A screen shot of a computer screen AI-generated content may be
incorrect.](media/image2.png){width="6.5in"
height="2.1694444444444443in"}

Now I will install etcd server

\# vim openssl-etcd.cnf

\[req\]

req_extensions = v3_req

distinguished_name = req_distinguished_name

\[req_distinguished_name\]

\[ v3_req \]

basicConstraints = CA:FALSE

keyUsage = nonRepudiation, digitalSignature, keyEncipherment

subjectAltName = \@alt_names

\[alt_names\]

IP.1 = 192.168.60.12 #This my host ip

IP.2 = 127.0.0.1

openssl genpkey -algorithm RSA -out etcd-server.key -pkeyopt
rsa_keygen_bits:2048

openssl req -new -key etcd-server.key -subj
\"/CN=etcd-server/O=Kubernetes\" -out etcd-server.csr -config
openssl-etcd.cnf

openssl x509 -req -in etcd-server.csr -CA ca.crt -CAkey ca.key
-CAcreateserial -out etcd-server.crt -extensions v3_req -extfile
openssl-etcd.cnf -days 1000

openssl verify -CAfile ca.crt etcd-server.crt

![A black screen with a black background AI-generated content may be
incorrect.](media/image3.png){width="6.5in"
height="1.6541666666666666in"}

Download etcd binary

wget
<https://github.com/etcd-io/etcd/releases/download/v3.6.0-rc.3/etcd-v3.6.0-rc.3-linux-amd64.tar.gz>

tar -xvf etcd-v3.6.0-rc.3-linux-amd64.tar.gz

mv etcd-v3.6.0-rc.3-linux-amd64/etcd
etcd-v3.6.0-rc.3-linux-amd64/etcdctl /usr/local/bin/

ls /usr/local/bin/

mkdir -p /etc/etcd /var/lib/etcd

chmod 700 /var/lib/etcd

![A black background with colorful lines AI-generated content may be
incorrect.](media/image4.png){width="6.5in"
height="1.0027777777777778in"}

cp etcd-server.key etcd-server.crt ca.crt /etc/etcd/

![A black rectangle with white text AI-generated content may be
incorrect.](media/image5.png){width="6.5in"
height="1.3194444444444444in"}

#vim /etc/systemd/system/etcd.service

\[Unit\]

Description=etcd

Documentation=https://github.com/coreos

\[Service\]

ExecStart=/usr/local/bin/etcd \\

\--name controlplane01 \\

\--cert-file=/etc/etcd/etcd-server.crt \\

\--key-file=/etc/etcd/etcd-server.key \\

\--peer-cert-file=/etc/etcd/etcd-server.crt \\

\--peer-key-file=/etc/etcd/etcd-server.key \\

\--trusted-ca-file=/etc/etcd/ca.crt \\

\--peer-trusted-ca-file=/etc/etcd/ca.crt \\

\--peer-client-cert-auth \\

\--client-cert-auth \\

\--initial-advertise-peer-urls https://192.168.60.12:2380 \\

\--listen-peer-urls https://192.168.60.12:2380 \\

\--listen-client-urls https://192.168.60.12:2379,https://127.0.0.1:2379
\\

\--advertise-client-urls https://192.168.60.12:2379 \\

\--initial-cluster-token etcd-cluster-0 \\

\--initial-cluster controlplane01=https://192.168.60.12:2380 \\

\--initial-cluster-state new \\

\--data-dir=/var/lib/etcd

Restart=on-failure

RestartSec=5

\[Install\]

WantedBy=multi-user.target

systemctl daemon-reload

systemctl start etcd

systemctl enable etcd

ETCDCTL_API=3 etcdctl member list \\

\--endpoints=https://127.0.0.1:2379 \\

\--cacert=/etc/etcd/ca.crt \\

\--cert=/etc/etcd/etcd-server.crt \\

\--key=/etc/etcd/etcd-server.key

![A computer screen shot of a computer screen AI-generated content may
be incorrect.](media/image6.png){width="6.5in"
height="2.8756944444444446in"}
