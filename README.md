# Setup Openshift Private Image Registry

## Overview
Panduan berikut untuk setup private image registry untuk kebutuhan instalasi openshift secara offline, disini hanya VM registry saja yang dapat mengakses domain domain tertentu di jaringan publik untuk mendapatkan resource yang dibutuhkan pada saat instalasi

## Prasyarat
1. VM dengan spesifikasi minimal (4 vCPU, 8 GB Memory, 500 GB Storage, RHEL 9)
2. VM dapat mengakses domain domain berikut
3. VM harus diassign alamat domain tertentu, misalnya registry.example.com

| From                      | To                                                                                                                                                                                                                | Port | Transport | Notes                                                                                                                |
| ------------------------- | ----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- | ---- | --------- | -------------------------------------------------------------------------------------------------------------------- |
| REGISTRY_DOMAIN | [registry.redhat.io](http://registry.redhat.io)                                                                                                                                                                   | 443  | TCP       | Provides core container images                                                                                       |
|                           | [access.redhat.com](http://access.redhat.com)                                                                                                                                                                     | 443  | TCP       | Hosts a signature store that a container client requires for verifying images pulled from registry.access.redhat.com |
|                           | [registry.access.redhat.com](http://registry.access.redhat.com)                                                                                                                                                   | 443  | TCP       | Hosts all the container images that are stored on the Red Hat Ecosystem Catalog, including core container images.    |
|                           | [quay.io](http://quay.io)                                                                                                                                                                                         | 443  | TCP       | Provides core container images                                                                                       |
|                           | [cdn.quay.io](http://cdn.quay.io)                                                                                                                                                                                 | 443  | TCP       | Provides core container images                                                                                       |
|                           | [cdn01.quay.io](http://cdn01.quay.io)                                                                                                                                                                             | 443  | TCP       | Provides core container images                                                                                       |
|                           | [cdn02.quay.io](http://cdn02.quay.io)                                                                                                                                                                             | 443  | TCP       | Provides core container images                                                                                       |
|                           | [cdn03.quay.io](http://cdn03.quay.io)                                                                                                                                                                             | 443  | TCP       | Provides core container images                                                                                       |
|                           | [cdn04.quay.io](http://cdn04.quay.io)                                                                                                                                                                             | 443  | TCP       | Provides core container images                                                                                       |
|                           | [cdn05.quay.io](http://cdn05.quay.io)                                                                                                                                                                             | 443  | TCP       | Provides core container images                                                                                       |
|                           | [cdn06.quay.io](http://cdn06.quay.io)                                                                                                                                                                             | 443  | TCP       | Provides core container images                                                                                       |
|                           | [cert-api.access.redhat.com](http://cert-api.access.redhat.com)                                                                                                                                                   | 443  | TCP       | for Telemetry                                                                                                        |
|                           | [api.access.redhat.com](http://api.access.redhat.com)                                                                                                                                                             | 443  | TCP       | for Telemetry                                                                                                        |
|                           | [infogw.api.openshift.com](http://infogw.api.openshift.com)                                                                                                                                                       | 443  | TCP       | for Telemetry                                                                                                        |
|                           | [console.redhat.com](http://console.redhat.com)                                                                                                                                                                   | 443  | TCP       | for Telemetry                                                                                                        |
|                           | [icr.io](http://icr.io)                                                                                                                                                                                           | 443  | TCP       | Cloud Pak for Integration Container Images                                                                           |
|                           | [*.icr.io](http://*.icr.io)                                                                                                                                                                                           | 443  | TCP       | Cloud Pak for Integration Container Mirror Image                                                                           |
|                           | [github.com](http://github.com)                                                                                                                                                                                   |      | TCP       | Cloud Pak for Integration Operators and Tools                                                                        |
|                           |                                                                                                                                                                                                                   |      |           |                                                                                                                      |
|                           | [sso.redhat.com](http://sso.redhat.com)                                                                                                                                                                           | 443  | TCP       | Redhat Authentication                                                                                                |
|                           | [console.redhat.com](http://console.redhat.com)                                                                                                                                                                   | 443  | TCP       |                                                                                                                      |
|                           | [mirror.openshift.com](http://mirror.openshift.com)                                                                                                                                                               | 443  | TCP       | Openshift mirror                                                                                                     |
|                           | [quayio-production-s3.s3.amazonaws.com](http://quayio-production-s3.s3.amazonaws.com)                                                                                                                             | 443  | TCP       | mirror quay                                                                                                          |
|                           | [api.openshift.com](http://api.openshift.com)                                                                                                                                                                     | 443  | TCP       | Openshift token                                                                                                      |
|                           | [rhcos.mirror.openshift.com](http://rhcos.mirror.openshift.com)                                                                                                                                                   | 443  | TCP       | RHCOS mirror                                                                                                         |
|                           | [storage.googleapis.com/openshift-release](http://storage.googleapis.com/openshift-release)                                                                                                                       | 443  | TCP       | image signature                                                                                                      |
|                           | [registry.connect.redhat.com](http://registry.connect.redhat.com)                                                                                                                                                 | 443  | TCP       | third party images                                                                                                   |
|                           | [rhc4tp-prod-z8cxf-image-registry-us-east-1-evenkyleffocxqvofrk.s3.dualstack.us-east-1.amazonaws.com](http://rhc4tp-prod-z8cxf-image-registry-us-east-1-evenkyleffocxqvofrk.s3.dualstack.us-east-1.amazonaws.com) | 443  | TCP       | access container images                                                                                              |
|                           | [oso-rhc4tp-docker-registry.s3-us-west-2.amazonaws.com](http://oso-rhc4tp-docker-registry.s3-us-west-2.amazonaws.com/)                                                                                            | 443  | TCP       | sonatype nexus                                                                                                       |
|                           | [1.rhel.pool.ntp.org](http://1.rhel.pool.ntp.org)                                                                                                                                                                 | 123  | UDP       |                                                                                                                      |
|                           | [2.rhel.pool.ntp.org](http://2.rhel.pool.ntp.org/)                                                                                                                                                                | 123  | UDP       |                                                                                                                      |
|                           | [3.rhel.pool.ntp.org](http://3.rhel.pool.ntp.org/)                                                                                                                                                                | 123  | UDP       |                                                                                                                      |


## Case
1. Pada environment yang memperbolehkan adanya sebuah VM untuk mengakses domain domain yang telah di sebutkan diatas secara temporary anda dapat mengikuti langkah langkah dibawah ini sampai poin 3.5
2. Pada environment yang tidak memperbolehkan sama sekali untuk akses ke domain yang telah disebutkan diatas diperlukan langkah langkah yang sedikit berbeda dibandingkan dengan langkah dibawah. 

## 1. Setup Docker

### 1.1. Install Docker
```bash
sudo dnf remove docker docker-client docker-client-latest docker-common docker-latest docker-latest-logrotate docker-logrotate docker-engine podman runc
sudo dnf -y install dnf-plugins-core
sudo dnf config-manager --add-repo https://download.docker.com/linux/rhel/docker-ce.repo
sudo dnf install docker-ce docker-ce-cli containerd.io docker-buildx-plugin docker-compose-plugin
sudo systemctl enable --now docker
sudo systemctl start docker
```

*instalasi docker secara offline*
1. download docker melalui link berikut [Docker](https://download.docker.com/linux/rhel/?_gl=1*1i3ykwe*_ga*NTE2MTAyMzI0LjE3MzY3MTQ4NDU.*_ga_XJWPQMJYHQ*czE3NTY3NTk1OTkkbzczJGcxJHQxNzU2NzU5NjAwJGo1OSRsMCRoMA..)
2. masukkan package package tersebut ke VM private registry

```bash
sudo dnf install ./containerd.io-<version>.<arch>.rpm \
  ./docker-ce-<version>.<arch>.rpm \
  ./docker-ce-cli-<version>.<arch>.rpm \
  ./docker-buildx-plugin-<version>.<arch>.rpm \
  ./docker-compose-plugin-<version>.<arch>.rpm
```

**Note** : refer to [Install Docker for RHEL](https://docs.docker.com/engine/install/rhel/)

## 2. Setup Harbor

### 2.1. Download Harbor
```bash
export HOME=/mnt/ocp
export CERTS_DIR=/mnt/ocp/certs
export IMAGES_DIR=/mnt/ocp/images
export HARBOR_VERSION=v2.13.2
export DOMAIN=YOUR_DOMAIN
export REGISTRY_DOMAIN=registry.${DOMAIN}

mkdir -p /mnt/ocp && mkdir -p /mnt/ocp/certs && mkdir -p /mnt/ocp/image && cd /mnt/ocp
wget https://github.com/goharbor/harbor/releases/download/${HARBOR_VERSION}/harbor-offline-installer-${HARBOR_VERSION}.tgz && tar -xvf harbor-offline-installer-${HARBOR_VERSION}.tgz
```
### 2.1. Buat Root CA Untuk Harbor
```bash
cd certs
openssl genrsa -out ca.key 4096
openssl req -x509 -new -nodes -sha512 -days 3650 -subj "/C=ID/ST=Jakarta/L=Jakarta/O=Digital/OU=Infrastructure/CN=${DOMAIN}" -key ca.key -out ca.crt
```

### Tips: sesuaikan subject Organization, Organization Unit, dan DN, kemudian selalu validasi konten Root CA dengan command berikut
```bash
openssl x509 -in ca.crt -text -noout
```

**Contoh Output :** 
```text
Certificate:
    Data:
        Version: 3 (0x2)
        Serial Number:
            4d:9b:51:e5:be:81:53:69:4a:67:5e:54:db:f2:57:27:46:1a:15:cc
        Signature Algorithm: sha512WithRSAEncryption
        Issuer: C=ID, ST=Jakarta, L=Jakarta, O=Digital, OU=Infrastructure, CN=asrori.id
        Validity
            Not Before: Aug 30 13:46:57 2025 GMT
            Not After : Aug 28 13:46:57 2035 GMT
        Subject: C=ID, ST=Jakarta, L=Jakarta, O=Digital, OU=Infrastructure, CN=asrori.id
        Subject Public Key Info:
            Public Key Algorithm: rsaEncryption
                Public-Key: (4096 bit)
                Modulus:
                    00:d2:4b:1a:35:52:08:65:c8:d0:75:6a:d3:8c:c1:
                    ...
                Exponent: 65537 (0x10001)
        X509v3 extensions:
            X509v3 Subject Key Identifier:
                80:B1:02:39:4B:8B:44:97:E7:2D:07:84:A0:30:ED:68:54:95:AF:A5
            X509v3 Authority Key Identifier:
                80:B1:02:39:4B:8B:44:97:E7:2D:07:84:A0:30:ED:68:54:95:AF:A5
            X509v3 Basic Constraints: critical
                CA:TRUE
    Signature Algorithm: sha512WithRSAEncryption
    Signature Value:
        61:92:1c:8d:73:f5:a3:f9:2c:a1:4c:90:96:2d:84:49:d4:6d:
        ...
```

### 2.2. Buat Extension Untuk Self Signed Certificate
```bash
cat > v3.ext <<-EOF
authorityKeyIdentifier=keyid,issuer
basicConstraints=CA:FALSE
keyUsage = digitalSignature, nonRepudiation, keyEncipherment, dataEncipherment
extendedKeyUsage = serverAuth
subjectAltName = @alt_names

[alt_names]
DNS.1=${REGISTRY_DOMAIN}
DNS.2=registry
EOF
``` 

### 2.3. Sign Domain Menggunakan Root CA
```bash
openssl genrsa -out ${REGISTRY_DOMAIN}.key 4096
openssl req -sha512 -new -subj "/C=ID/ST=Jakarta/L=Jakarta/O=Digital/OU=Infrastructure/CN=${REGISTRY_DOMAIN}" -key ${REGISTRY_DOMAIN}.key -out ${REGISTRY_DOMAIN}.csr

openssl x509 -req -sha512 -days 3650 -extfile v3.ext -CA ca.crt -CAkey ca.key -CAcreateserial -in ${REGISTRY_DOMAIN}.csr -out ${REGISTRY_DOMAIN}.crt
```

Note : sesuaikan subject Organization, Organization Unit, dan DN. ketika anda melakukan signing terhadap domain sebuah domain menggunakan Root CA. output yang dihasilnya seperti ini misalnya :

```bash
Certificate request self-signature ok
subject=C=ID, ST=Jakarta, L=Jakarta, O=Digital, OU=Infrastructure, CN=registry.asrori.id``` 
```


### 2.4. Konversi Certificate ke Format PEM 
```bash
openssl x509 -inform PEM -in ${REGISTRY_DOMAIN}.crt -out ${REGISTRY_DOMAIN}.cert
```

### Tips: selalu lihat isi konten self signed certificate dengan command berikut
```bash
openssl x509 -in ${REGISTRY_DOMAIN}.crt -text -noout
```

contoh output 
```bash
Certificate:
    Data:
        Version: 3 (0x2)
        Serial Number:
            34:52:fd:af:57:3a:6b:2e:40:e7:21:ce:cf:e8:c8:47:54:95:2b:eb
        Signature Algorithm: sha512WithRSAEncryption
        Issuer: C=ID, ST=Jakarta, L=Jakarta, O=Digital, OU=Infrastructure, CN=asrori.id
        Validity
            Not Before: Aug 30 14:03:15 2025 GMT
            Not After : Aug 28 14:03:15 2035 GMT
        Subject: C=ID, ST=Jakarta, L=Jakarta, O=Digital, OU=Infrastructure, CN=registry.asrori.id
        Subject Public Key Info:
            Public Key Algorithm: rsaEncryption
                Public-Key: (4096 bit)
                Modulus:
                    00:c4:8d:8f:53:19:86:0f:79:f4:2f:c7:a6:f5:4e:
                    ...
                Exponent: 65537 (0x10001)
        X509v3 extensions:
            X509v3 Authority Key Identifier:
                AE:A8:3F:1C:89:EB:BA:8E:C9:34:5B:44:8F:37:C8:25:1B:5E:96:AF
            X509v3 Basic Constraints:
                CA:FALSE
            X509v3 Key Usage:
                Digital Signature, Non Repudiation, Key Encipherment, Data Encipherment
            X509v3 Extended Key Usage:
                TLS Web Server Authentication
            X509v3 Subject Alternative Name:
                DNS:registry.asrori.id, DNS:registry
            X509v3 Subject Key Identifier:
                2D:DC:E2:EF:C8:A3:92:07:3A:B2:28:8A:99:41:B4:79:58:C7:42:B5
    Signature Algorithm: sha512WithRSAEncryption
    Signature Value:
        a0:6e:49:cd:13:aa:05:e8:d7:cb:84:2f:94:81:55:1e:fc:84:
        ...
```


### 2.5. Pasang Certificate Format PEM Untuk Docker
```bash
mkdir -p /etc/docker/certs.d/${REGISTRY_DOMAIN}
cp ${REGISTRY_DOMAIN}.cert ${REGISTRY_DOMAIN}.key ca.crt /etc/docker/certs.d/${REGISTRY_DOMAIN}
```

### 2.6. Pasang Root CA ke VM Private Image Registry atau VM Bastion 
```bash
cp ca.crt /etc/pki/ca-trust/source/anchors/
update-ca-trust
```

### 2.7. Restart Docker
```bash
sudo systemctl restart docker
cd && cd harbor
```

### 2.8. Setup Harbor
```bash
cat > harbor.yml <<-EOF
hostname: ${REGISTRY_DOMAIN}

http:
  port: 80

https:
  port: 443
  certificate: ${CERTS_DIR}/${REGISTRY_DOMAIN}.cert
  private_key: ${CERTS_DIR}/${REGISTRY_DOMAIN}.key
  
harbor_admin_password: Harbor12345

database:
  password: root123
  max_idle_conns: 100
  max_open_conns: 900
  conn_max_lifetime: 5m
  conn_max_idle_time: 0

data_volume: ${IMAGES_DIR}

trivy:
  ignore_unfixed: false
  skip_update: false
  skip_java_db_update: false
  offline_scan: false
  security_check: vuln
  insecure: false
  timeout: 5m0s

jobservice:
  max_job_workers: 10
  max_job_duration_hours: 24
  job_loggers:
    - STD_OUTPUT
    - FILE
  logger_sweeper_duration: 1 #days

notification:
  webhook_job_max_retry: 3
  webhook_job_http_client_timeout: 3 #seconds

log:
  level: info
  local:
    rotate_count: 50
    rotate_size: 200M
    location: /var/log/harbor

_version: 2.13.0

proxy:
  http_proxy:
  https_proxy:
  no_proxy:
  components:
    - core
    - jobservice
    - trivy

upload_purging:
  enabled: true
  age: 168h
  interval: 24h
  dryrun: false

cache:
  enabled: false
  expire_hours: 24
  
EOF
```

### 2.8. Jalankan Harbor
```bash
docker load -i harbor.${HARBOR_VERSION}.tar.gz
```
**output**
```bash
.....
d41a7971d79a: Loading layer [==================================================>]  40.92MB/40.92MB
23085cdcf9d9: Loading layer [==================================================>]  102.8MB/102.8MB
2dbea2922ddd: Loading layer [==================================================>]  51.84MB/51.84MB
fc57623bb37d: Loading layer [==================================================>]  13.98MB/13.98MB
47b3b7df2746: Loading layer [==================================================>]  66.05kB/66.05kB
c9bd41405cfa: Loading layer [==================================================>]   2.56kB/2.56kB
0f62a2991247: Loading layer [==================================================>]  1.536kB/1.536kB
7c11e8815785: Loading layer [==================================================>]  12.29kB/12.29kB
1ca462c05d95: Loading layer [==================================================>]  3.414MB/3.414MB
dbce63374990: Loading layer [==================================================>]  561.7kB/561.7kB
Loaded image: goharbor/prepare:v2.13.2
b8d9ed7472a1: Loading layer [==================================================>]  11.62MB/11.62MB
d636a318bc3e: Loading layer [==================================================>]  3.584kB/3.584kB
ae4e509b0959: Loading layer [==================================================>]   2.56kB/2.56kB
4e44bd229445: Loading layer [==================================================>]  74.19MB/74.19MB
969d405abe51: Loading layer [==================================================>]  5.632kB/5.632kB
679eaed82f02: Loading layer [==================================================>]    128kB/128kB
acc66297eb18: Loading layer [==================================================>]  209.9kB/209.9kB
c528a6ed9282: Loading layer [==================================================>]  75.32MB/75.32MB
74c02e7052f0: Loading layer [==================================================>]   2.56kB/2.56kB
Loaded image: goharbor/harbor-core:v2.13.2
95b0a8775e8b: Loading layer [==================================================>]  16.48MB/16.48MB
23cda48793f9: Loading layer [==================================================>]  175.3MB/175.3MB
59077178faae: Loading layer [==================================================>]  26.56MB/26.56MB
ae34cce00dbc: Loading layer [==================================================>]  18.57MB/18.57MB
17278233736b: Loading layer [==================================================>]   5.12kB/5.12kB
d1e500c86487: Loading layer [==================================================>]  6.144kB/6.144kB
b161ec810c64: Loading layer [==================================================>]  3.072kB/3.072kB
6b969541a9ab: Loading layer [==================================================>]  2.048kB/2.048kB
ef7b74187540: Loading layer [==================================================>]   2.56kB/2.56kB
d9cb02dd1d79: Loading layer [==================================================>]  14.85kB/14.85kB
Loaded image: goharbor/harbor-db:v2.13.2
59571a26a4ca: Loading layer [==================================================>]  11.62MB/11.62MB
6728a8a69d88: Loading layer [==================================================>]  3.584kB/3.584kB
d7762407b524: Loading layer [==================================================>]   2.56kB/2.56kB
8ebfb551a501: Loading layer [==================================================>]  63.68MB/63.68MB
a1ca2c4dae85: Loading layer [==================================================>]  64.47MB/64.47MB
Loaded image: goharbor/harbor-jobservice:v2.13.2
784874945d46: Loading layer [==================================================>]  11.62MB/11.62MB
4ee71defc051: Loading layer [==================================================>]  40.65MB/40.65MB
a67d1114c14f: Loading layer [==================================================>]  4.608kB/4.608kB
04ed750f9c05: Loading layer [==================================================>]  41.44MB/41.44MB
Loaded image: goharbor/harbor-exporter:v2.13.2
b99f63f14406: Loading layer [==================================================>]  112.3MB/112.3MB
ded14cff346a: Loading layer [==================================================>]  6.835MB/6.835MB
c8c0275e2b2f: Loading layer [==================================================>]  252.9kB/252.9kB
d857eca5ce6a: Loading layer [==================================================>]  1.539MB/1.539MB
Loaded image: goharbor/harbor-portal:v2.13.2
65102c38b1ca: Loading layer [==================================================>]  125.2MB/125.2MB
a03d6655921c: Loading layer [==================================================>]  3.584kB/3.584kB
9c79a543bfb3: Loading layer [==================================================>]  3.072kB/3.072kB
72268863b0e2: Loading layer [==================================================>]   2.56kB/2.56kB
6fa267ebb810: Loading layer [==================================================>]  3.072kB/3.072kB
ebe14c87ab04: Loading layer [==================================================>]  3.584kB/3.584kB
ab9c78054127: Loading layer [==================================================>]  20.48kB/20.48kB
Loaded image: goharbor/harbor-log:v2.13.2
569aa44b0ecf: Loading layer [==================================================>]  8.665MB/8.665MB
732b123a1de1: Loading layer [==================================================>]  4.096kB/4.096kB
59de1aec3aaf: Loading layer [==================================================>]  18.96MB/18.96MB
ab52bfe859a0: Loading layer [==================================================>]  3.072kB/3.072kB
044c3c7f0c6b: Loading layer [==================================================>]  39.09MB/39.09MB
9bb956bf9466: Loading layer [==================================================>]  58.85MB/58.85MB
Loaded image: goharbor/harbor-registryctl:v2.13.2
4af0b05a15c4: Loading layer [==================================================>]  16.48MB/16.48MB
b7d082b0d116: Loading layer [==================================================>]  110.5MB/110.5MB
09294173d458: Loading layer [==================================================>]  3.072kB/3.072kB
c186056b4c24: Loading layer [==================================================>]   59.9kB/59.9kB
4a81bc50f73a: Loading layer [==================================================>]  61.95kB/61.95kB
Loaded image: goharbor/redis-photon:v2.13.2
9ff850898dcc: Loading layer [==================================================>]  112.3MB/112.3MB
Loaded image: goharbor/nginx-photon:v2.13.2
ef761ee04cd8: Loading layer [==================================================>]  8.665MB/8.665MB
6a5c6fb7e587: Loading layer [==================================================>]  4.096kB/4.096kB
f4348053411c: Loading layer [==================================================>]  3.072kB/3.072kB
4dbd04f6119b: Loading layer [==================================================>]  18.96MB/18.96MB
ed7e7803eb40: Loading layer [==================================================>]  19.76MB/19.76MB
Loaded image: goharbor/registry-photon:v2.13.2
a3c6016bfc77: Loading layer [==================================================>]  9.158MB/9.158MB
7e6cf43e2227: Loading layer [==================================================>]  4.096kB/4.096kB
71bd290cafa1: Loading layer [==================================================>]  3.072kB/3.072kB
9dae772d7025: Loading layer [==================================================>]  153.5MB/153.5MB
d55c9ebec784: Loading layer [==================================================>]  16.49MB/16.49MB
efa2d12a453e: Loading layer [==================================================>]  170.8MB/170.8MB
Loaded image: goharbor/trivy-adapter-photon:v2.13.2
...
```

```bash
./prepare
./install.sh
```

Note: ketika menjalankan command ./install.sh, pastikan output yang dihasilkan seperti berikut

```text
......
[Step 5]: starting Harbor ...
[+] Running 10/10
 ✔ Network harbor_harbor        Created                                                                                          0.0s
 ✔ Container harbor-log         Started                                                                                          0.3s
 ✔ Container harbor-portal      Started                                                                                          0.8s
 ✔ Container harbor-db          Started                                                                                          0.6s
 ✔ Container registryctl        Started                                                                                          0.6s
 ✔ Container registry           Started                                                                                          0.7s
 ✔ Container redis              Started                                                                                          0.7s
 ✔ Container harbor-core        Started                                                                                          0.9s
 ✔ Container nginx              Started                                                                                          1.3s
 ✔ Container harbor-jobservice  Started                                                                                          1.2s
✔ ----Harbor has been installed and started successfully.----
```

### 2.9. Setup Harbor Project

*login ke harbor, user : admin, password : Harbor12345*
![alt text](images/1.%20harbor-login.png)

*anda akan diarahkan ke halaman dashboard*
![alt text](images/2.%20harbor-dashboard.png)

*pergi ke menu users dan tekan tombol +new users*
![alt text](images/3.%20harbor-user.png)

*masukkan informasi user yang anda ingin tambahkan*
![alt text](images/4.%20harbor-user.png)

*hasil*
![alt text](images/5.%20harbor-user.png)

*klik menu projects, klik tombol +new projects*
![alt text](images/6.%20create-project.png)

*masukkan nama project dan tekan tombol save*
![alt text](images/7.%20create-project-ocp.png)

*hasil*
![alt text](images/8.%20create-project-ocp.png)


*klik nama project yang telah dibuat*
![alt text](images/9.%20create-project-ocp.png)

*klik tab member dan klik tombol +user*
![alt text](images/10.%20create-project-ocp.png)

*masukkan nama user yang telah ditambahkan dan klik tombol save*
![alt text](images/11.%20create-project-ocp.png)

*hasil*
![alt text](images/12.%20create-project-ocp.png)


*jika anda ingin melakukan mirroring image terhadap project cloud pak, buat project dan tambahkan *
![alt text](images/13.%20create-project-cp.png)



## 3. Setup Openshift Mirror

### 3.1. Download Openshift Client dan Openshift Mirror
```bash
export WORKSPACE_FOLDER=mirror-workspace
export OCP_VERSION=4.19.9
export OCP_PROJECT=ocp
cd
wget https://mirror.openshift.com/pub/openshift-v4/x86_64/clients/ocp/${OCP_VERSION}/openshift-client-linux-amd64-rhel9-${OCP_VERSION}.tar.gz
wget https://mirror.openshift.com/pub/openshift-v4/x86_64/clients/ocp/${OCP_VERSION}/oc-mirror.rhel9.tar.gz
tar -xvf oc-mirror.rhel9.tar.gz
tar -xvf openshift-client-linux-amd64-rhel9-${OCP_VERSION}.tar.gz
chmod +x oc-mirror oc kubectl
mv oc-mirror oc kubectl /usr/bin
```

### 3.2. Download Pull Secret Dari [Redhat Console](https://console.redhat.com/openshift/install/pull-secret)

simpan file pull-secret di direktori /mnt/ocp

### 3.3. Konfigurasi Akses Registry
```bash
cd
mkdir .docker
cat pull-secret | jq . > config.json
mv config.json .docker
docker login ${REGISTRY_DOMAIN}
```

Tips : pastikan didalam file config.json yang ada direktori .docker terdapat domain registry

### 3.4. Konfigurasi ImageSet
```yaml
cat > ImageSetConfiguration.yaml <<-EOF
kind: ImageSetConfiguration
apiVersion: mirror.openshift.io/v2alpha1
mirror:
  platform:
    channels:
    - name: stable-4.19
      minVersion: ${OCP_VERSION}
      maxVersion: ${OCP_VERSION}
    graph: true
  operators:
    - catalog: registry.redhat.io/redhat/certified-operator-index:v4.19
      packages: 
       - name: gpu-operator-certified
    - catalog: registry.redhat.io/redhat/redhat-operator-index:v4.19
      packages:
       - name: nfd
       - name: openshift-cert-manager
       - name: openshift-builds-operator
       - name: openshift-gitops-operator
       - name: multicluster-engine
       - name: multicluster-global-hub-operator-rh
       - name: node-observability-operator
       - name: kernel-module-management
       - name: kernel-madule-management-hub
       - name: lightspeed-operator
       - name: rhods-operator
       - name: ocs-operator
       - name: odf-operator
       - name: mcg-operator
       - name: odf-csi-addons-operator
       - name: ocs-client-operator
       - name: odf-prometheus-operator
       - name: recipe
       - name: rook-ceph-operator
       - name: cephcsi-operator
       - name: odf-dependencies
       - name: odr-cluster-operator
       - name: odr-hub-operator
       - name: rhacs-operator
       - name: rhsso-operator
       - name: servicemeshoperator3
       - name: local-storage-operator
  additionalImages:
   - name: registry.redhat.io/ubi8/ubi:latest
   - name: registry.redhat.io/ubi9/ubi@sha256:20f695d2a91352d4eaa25107535126727b5945bff38ed36a3e59590f495046f0
   - name: registry.redhat.io/rhel9/python-312@sha256:95ec8d3ee9f875da011639213fd254256c29bc58861ac0b11f290a291fa04435
   - name: registry.redhat.io/ubi9/openjdk-21-runtime@sha256:75fa8df8118eaa1710b5676192a10711768f82f02dd9303009aa2c8d23de5cd5
   - name: registry.redhat.io/rhel9/postgresql-13@sha256:504233a6d4309115cee4712f14a121e048d27aac74c0620896244bf91794446a
   - name: registry.redhat.io/ubi9/nodejs-22@sha256:1fc87050383d51ce796732449113f18dfd30b9e59de7aafeb87633ce43f40b79
   - name: registry.redhat.io/rhel9/mysql-84@sha256:8d67c8546cf54613dc8c58cb81d341d83427e43708576164b7c26245f24f6074
EOF
```
### 3.5. Pull Image Dari Redhat Ke Private Image Registry
```bash
cd
mkdir ${WORKSPACE_FOLDER}
oc mirror -c ImageSetConfiguration.yaml --workspace file:///${HOME}/${WORKSPACE_FOLDER} docker://${REGISTRY_DOMAIN}/${OCP_PROJECT} --v2
```

Output : 
```bash
[root@registry ~]# oc mirror -c ImageSetConfiguration.yaml --workspace file://mnt/ocp/mirror-workspace docker://${REGISTRY_DOMAIN}/ocp --v2
2025/08/30 [INFO]   : 👋 Hello, welcome to oc-mirror
2025/08/30 [INFO]   : ⚙️  setting up the environment for you...
2025/08/30 [INFO]   : 🔀 workflow mode: mirrorToMirror
WARN[0000] Ignoring unrecognized environment variable REGISTRY_DOMAIN
2025/08/30 [INFO]   : 🕵  going to discover the necessary images...
2025/08/30 [INFO]   : 🔍 collecting release images...
2025/08/30 [INFO]   : 🔍 collecting operator images...
 ✓   (23s) Collecting catalog registry.redhat.io/redhat/certified-operator-index:v4.19
 ✓   (20s) Collecting catalog registry.redhat.io/redhat/redhat-operator-index:v4.19
2025/08/30 [INFO]   : 🔍 collecting additional images...
2025/08/30 [INFO]   : 🔍 collecting helm images...
2025/08/30 [INFO]   : 🔂 rebuilding catalogs
 ✓   (15s) Rebuilding catalog docker://registry.redhat.io/redhat/certified-operator-index:v4.19
 ✓   (16s) Rebuilding catalog docker://registry.redhat.io/redhat/redhat-operator-index:v4.19
2025/08/30 [INFO]   : 🚀 Start copying the images...
2025/08/30 [INFO]   : 📌 images to copy 1317
...
```

Tunggu sampai proses pull image selesai. lama proses pull image tergantung pada kecepatan atau bandwidth jaringan anda

hasil : 
```bash
...
2025/08/30 [INFO]   : === Results ===
2025/08/30 [INFO]   :  ✓  191 / 191 release images mirrored successfully
2025/08/30 [INFO]   :  ✓  1119 / 1119 operator images mirrored successfully
2025/08/30 [INFO]   :  ✓  7 / 7 additional images mirrored successfully
2025/08/30 [INFO]   : 📄 Generating IDMS file...
2025/08/30 [INFO]   : mirror-workspace/working-dir/cluster-resources/idms-oc-mirror.yaml file created
2025/08/30 [INFO]   : 📄 Generating ITMS file...
2025/08/30 [INFO]   : mirror-workspace/working-dir/cluster-resources/itms-oc-mirror.yaml file created
2025/08/30 [INFO]   : 📄 Generating CatalogSource file...
2025/08/30 [INFO]   : mirror-workspace/working-dir/cluster-resources/cs-redhat-operator-index-v4-19.yaml file created
2025/08/30 [INFO]   : mirror-workspace/working-dir/cluster-resources/cs-certified-operator-index-v4-19.yaml file created
2025/08/30 [INFO]   : 📄 Generating ClusterCatalog file...
2025/08/30 [INFO]   : mirror-workspace/working-dir/cluster-resources/cc-redhat-operator-index-v4-19.yaml file created
2025/08/30 [INFO]   : mirror-workspace/working-dir/cluster-resources/cc-certified-operator-index-v4-19.yaml file created
2025/08/30 [INFO]   : 📄 Generating Signature Configmap...
2025/08/30 [INFO]   : mirror-workspace/working-dir/cluster-resources/signature-configmap.json file created
2025/08/30 [INFO]   : mirror-workspace/working-dir/cluster-resources/signature-configmap.yaml file created
2025/08/30 [INFO]   : 📄 Generating UpdateService file...
2025/08/30 [INFO]   : mirror-workspace/working-dir/cluster-resources/updateService.yaml file created
2025/08/30 [INFO]   : mirror time     : 1h10m13.222771756s
2025/08/30 [INFO]   : 👋 Goodbye, thank you for using oc-mirror
```

*tampilan di project ocp*
![alt text](images/14.%20hasil-pull-image.png)

setelah menjalankan command diatas menghasilkan beberapa file yang harus diperhatikan

```bash
cd ${WORKSPACE_FOLDER}/working-dir/cluster-resources
```

output :

```text
[root@registry cluster-resources]# ls
cc-certified-operator-index-v4-19.yaml  cs-certified-operator-index-v4-19.yaml  idms-oc-mirror.yaml  signature-configmap.json  updateService.yaml
cc-redhat-operator-index-v4-19.yaml     cs-redhat-operator-index-v4-19.yaml     itms-oc-mirror.yaml  signature-configmap.yaml
```

perhatikan file cs-redhat-operator-index-v4-19.yaml, modifikasi file tersebut yang semula

```yaml
apiVersion: operators.coreos.com/v1alpha1
kind: CatalogSource
metadata:
  annotations:
    createdAt: Saturday, 30-Aug-25 19:34:57 UTC
    createdBy: oc-mirror v2
    oc-mirror_version: 4.19.0-202508130123.p2.ge91ec32.assembly.stream.el9-e91ec32
  name: cs-redhat-operator-index-v4-19
  namespace: openshift-marketplace
spec:
  image: registry.asrori.id/ocp/redhat/redhat-operator-index:v4.19
  sourceType: grpc
status: {}
```

modifikasi menjadi 

```yaml
apiVersion: operators.coreos.com/v1alpha1
kind: CatalogSource
metadata:
  annotations:
    createdAt: Saturday, 30-Aug-25 19:34:57 UTC
    createdBy: oc-mirror v2
    oc-mirror_version: 4.19.0-202508130123.p2.ge91ec32.assembly.stream.el9-e91ec32
  name: redhat-operators 			#rename
  namespace: openshift-marketplace
spec:
  image: registry.asrori.id/ocp/redhat/redhat-operator-index:v4.19
  sourceType: grpc
  # tambah konfigurasi dibawah ini
  grpcPodConfig:
    securityContextConfig: restricted
  displayName: Red Hat Operators
  publisher: Red Hat
  updateStrategy:
    registryPoll: 
      interval: 30m
status: {}
```
perhatikan file cs-certified-operator-index-v4-19.yaml, modifikasi file tersebut yang semula
```yaml
apiVersion: operators.coreos.com/v1alpha1
kind: CatalogSource
metadata:
  annotations:
    createdAt: Saturday, 30-Aug-25 19:34:57 UTC
    createdBy: oc-mirror v2
    oc-mirror_version: 4.19.0-202508130123.p2.ge91ec32.assembly.stream.el9-e91ec32
  name: cs-certified-operator-index-v4-19
  namespace: openshift-marketplace
spec:
  image: registry.asrori.id/ocp/redhat/certified-operator-index:v4.19
  sourceType: grpc
status: {}
```

modifikasi menjadi 
```yaml
apiVersion: operators.coreos.com/v1alpha1
kind: CatalogSource
metadata:
  annotations:
    createdAt: Saturday, 30-Aug-25 19:34:57 UTC
    createdBy: oc-mirror v2
    oc-mirror_version: 4.19.0-202508130123.p2.ge91ec32.assembly.stream.el9-e91ec32
  name: redhat-certified-operators
  namespace: openshift-marketplace
spec:
  image: registry.asrori.id/ocp/redhat/certified-operator-index:v4.19
  sourceType: grpc
  grpcPodConfig:
    securityContextConfig: restricted
  displayName: Red Hat Certified Operators
  publisher: Red Hat
  updateStrategy:
    registryPoll: 
      interval: 30m
status: {}
```


pada saat cluster akan diinstall maka ketika anda membuat file install-config.yaml, jangan lupa tambahkan baris berikut

```yaml
...
additionalTrustBundle: |
  -----BEGIN CERTIFICATE-----
  ISI_FILE ca.crt
  -----END CERTIFICATE-----
imageContentSources:
  - mirrors:
    - ${REGISTRY_DOMAIN}/${OCP_PROJECT}/openshift/release
    source: quay.io/openshift-release-dev/ocp-v4.0-art-dev
  - mirrors:
    - ${REGISTRY_DOMAIN}/${OCP_PROJECT}/openshift/release-images
    source: quay.io/openshift-release-dev/ocp-release
```

jika cluster sudah terinstall, jangan lupa untuk mendisable default operator catalog dan jalankan perintah berikut
```bash
oc apply -f ${HOME}/${WORKSPACE_FOLDER}/working-dir/cluster-resources/*.yaml
```


# Pull Image Dari Redhat Ke Disk (Untuk Environment Yang Sangat Privat)
```bash
cd
mkdir ${WORKSPACE_FOLDER}
oc mirror -c ImageSetConfiguration.yaml --workspace file://${HOME}/${WORKSPACE_FOLDER}  --v2
```

setelah proses download selesai archive folder workspace menggunakan perintah erbikut
```bash
tar -czvf -r ocp-images.tar.gz 
```

kemudian tranfer file tersebut ke private docker registry di environment anda

dan jalankan perintah berikut 
```bash
oc mirror -c ImageSetConfiguration.yaml --from file://${WORKSPACE_FOLDER} docker://${REGISTRY_DOMAIN} --v2
```

# Day 1 (Installation)

## Operator Catalog
untuk lingkungan yang tidak dapat mengakses jaringan publik anda perlu menonaktifkan default catalog source

```bash
oc patch OperatorHub cluster --type json -p '[{"op": "add", "path": "/spec/disableAllDefaultSources", "value": true}]'
```

# Day 2 (Maintenance)
## 1. Update Cluster
### 1.1. Buat CA ConfigMap
```bash
cat > cm-registry.yaml <<-EOF
apiVersion: v1
kind: ConfigMap
metadata:
  name: private-registry-ca
data:
  updateservice-registry: | 
    -----BEGIN CERTIFICATE-----
    isi file ca.crt
    -----END CERTIFICATE-----
EOF
```
```bash
oc apply cm-registry.yaml
``` 
### 1.2. Update Global Pull Secret
```bash
oc get secret/pull-secret -n openshift-config --template='{{index .data ".dockerconfigjson" | base64decode}}' > .
oc registry login --registry=${REGISTRY_DOMAIN} --auth-basic="<username>:<password>" --to=.
oc set data secret/pull-secret -n openshift-config --from-file=.dockerconfigjson=.
```
Note : sesuaikan username dan passwordnya

### 1.3. Install Update Service Operator via OperatorHub

### 1.4. Konfigurasi CVO
```bash
export NAMESPACE=openshift-update-service
export NAME=update-service-oc-mirror
POLICY_ENGINE_GRAPH_URI="$(oc -n "${NAMESPACE}" get -o jsonpath='{.status.policyEngineURI}/api/upgrades_info/v1/graph{"\n"}' updateservice "${NAME}")"
export POLICY_ENGINE_GRAPH_URI=hasil_dari_command_diatas
export PATCH="{\"spec\":{\"upstream\":\"${POLICY_ENGINE_GRAPH_URI}\"}}"
oc patch clusterversion version -p  $PATCH  --type merge
```
