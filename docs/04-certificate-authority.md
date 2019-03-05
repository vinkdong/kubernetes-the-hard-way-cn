# 准备证书

这里需要准备一堆证书


## 先创个ca

```

cat > ca-config.json <<EOF
{
  "signing": {
    "default": {
      "expiry": "8760h"
    },
    "profiles": {
      "kubernetes": {
        "usages": ["signing", "key encipherment", "server auth", "client auth"],
        "expiry": "8760h"
      }
    }
  }
}
EOF

cat > ca-csr.json <<EOF
{
  "CN": "Kubernetes",
  "key": {
    "algo": "rsa",
    "size": 2048
  },
  "names": [
    {
      "C": "US",
      "L": "Portland",
      "O": "Kubernetes",
      "OU": "CA",
      "ST": "Oregon"
    }
  ]
}
EOF

cfssl gencert -initca ca-csr.json | cfssljson -bare ca

```

创建完成之后有好几个文件，关注下面两个文件即可

```
ca-key.pem
ca.pem
```

### 创建admin客户端证书

```bash
cat > admin-csr.json <<EOF
{
  "CN": "admin",
  "key": {
    "algo": "rsa",
    "size": 2048
  },
  "names": [
    {
      "C": "US",
      "L": "Portland",
      "O": "system:masters",
      "OU": "Kubernetes The Hard Way",
      "ST": "Oregon"
    }
  ]
}
EOF

配置和ca一样就可以

cfssl gencert \
  -ca=ca.pem \
  -ca-key=ca-key.pem \
  -config=ca-config.json \
  -profile=kubernetes \
  admin-csr.json | cfssljson -bare admin

```

同样会生成两个文件

```bash
admin-key.pem
admin.pem
```

### 生成kubelet证书（worker节点就可以）

假如我有两个worker节点

vk004, 外网ip:39.98.175.114, 内网ip:172.26.192.64
vk005, 外网ip:39.98.180.57, 内网ip:172.26.192.63

```bash
vk004="vk004,39.98.175.114,172.26.192.64"
vk005="vk005,39.98.180.57,172.26.192.63"
for x in $vk004 $vk005; do
IFS=',' read -r instance EXTERNAL_IP INTERNAL_IP <<< "$x"

cat > ${instance}-csr.json <<EOF
{
  "CN": "system:node:${instance}",
  "key": {
    "algo": "rsa",
    "size": 2048
  },
  "names": [
    {
      "C": "US",
      "L": "Portland",
      "O": "system:nodes",
      "OU": "Kubernetes The Hard Way",
      "ST": "Oregon"
    }
  ]
}
EOF

cfssl gencert \
  -ca=ca.pem \
  -ca-key=ca-key.pem \
  -config=ca-config.json \
  -hostname=${instance},${EXTERNAL_IP},${INTERNAL_IP} \
  -profile=kubernetes \
  ${instance}-csr.json | cfssljson -bare ${instance}
done
```

会生成一组证书

```bash
vk004-key.pem
vk004.pem
vk005-key.pem
vk005.pem
```

### 生成Controller的证书

```
cat > kube-controller-manager-csr.json <<EOF
{
  "CN": "system:kube-controller-manager",
  "key": {
    "algo": "rsa",
    "size": 2048
  },
  "names": [
    {
      "C": "US",
      "L": "Portland",
      "O": "system:kube-controller-manager",
      "OU": "Kubernetes The Hard Way",
      "ST": "Oregon"
    }
  ]
}
EOF
cfssl gencert \
  -ca=ca.pem \
  -ca-key=ca-key.pem \
  -config=ca-config.json \
  -profile=kubernetes \
  kube-controller-manager-csr.json | cfssljson -bare kube-controller-manager
```

同样会生成两个文件

```
kube-controller-manager-key.pem
kube-controller-manager.pem
```

### 生成Proxy的证书

```bash
cat > kube-proxy-csr.json <<EOF
{
  "CN": "system:kube-proxy",
  "key": {
    "algo": "rsa",
    "size": 2048
  },
  "names": [
    {
      "C": "US",
      "L": "Portland",
      "O": "system:node-proxier",
      "OU": "Kubernetes The Hard Way",
      "ST": "Oregon"
    }
  ]
}
EOF
cfssl gencert \
  -ca=ca.pem \
  -ca-key=ca-key.pem \
  -config=ca-config.json \
  -profile=kubernetes \
  kube-proxy-csr.json | cfssljson -bare kube-proxy

```

还是生成两个文件

```bash
kube-proxy-key.pem
kube-proxy.pem
```

### 生成Scheduler的证书

```bash
cat > kube-scheduler-csr.json <<EOF
{
  "CN": "system:kube-scheduler",
  "key": {
    "algo": "rsa",
    "size": 2048
  },
  "names": [
    {
      "C": "US",
      "L": "Portland",
      "O": "system:kube-scheduler",
      "OU": "Kubernetes The Hard Way",
      "ST": "Oregon"
    }
  ]
}
EOF
cfssl gencert \
  -ca=ca.pem \
  -ca-key=ca-key.pem \
  -config=ca-config.json \
  -profile=kubernetes \
  kube-scheduler-csr.json | cfssljson -bare kube-scheduler

```

生成两个文件：

```bash
kube-scheduler-key.pem
kube-scheduler.pem
```

### 生成ApiServer的证书

```bash
cat > kubernetes-csr.json <<EOF
{
  "CN": "kubernetes",
  "key": {
    "algo": "rsa",
    "size": 2048
  },
  "names": [
    {
      "C": "US",
      "L": "Portland",
      "O": "Kubernetes",
      "OU": "Kubernetes The Hard Way",
      "ST": "Oregon"
    }
  ]
}
EOF

KUBERNETES_PUBLIC_ADDRESS="39.98.167.186" # master节点或者lb的公网ip
MASTER_NODE_IPS="172.26.192.62,172.26.192.65,172.26.192.66,172.26.192.67" #master节点的内网ip和slb ip
cfssl gencert \
  -ca=ca.pem \
  -ca-key=ca-key.pem \
  -config=ca-config.json \
  -hostname=10.32.0.1,${MASTER_NODE_IPS},${KUBERNETES_PUBLIC_ADDRESS},127.0.0.1,kubernetes.default \
  -profile=kubernetes \
  kubernetes-csr.json | cfssljson -bare kubernetes

```

生成两个文件

```bash
kubernetes-key.pem
kubernetes.pem
```

### 最后再创一个sa的证书

```bash
cat > service-account-csr.json <<EOF
{
  "CN": "service-accounts",
  "key": {
    "algo": "rsa",
    "size": 2048
  },
  "names": [
    {
      "C": "US",
      "L": "Portland",
      "O": "Kubernetes",
      "OU": "Kubernetes The Hard Way",
      "ST": "Oregon"
    }
  ]
}
EOF
cfssl gencert \
  -ca=ca.pem \
  -ca-key=ca-key.pem \
  -config=ca-config.json \
  -profile=kubernetes \
  service-account-csr.json | cfssljson -bare service-account

```

两个文件：

```bash
service-account-key.pem
service-account.pem
```

# 分发证书

## work节点

将ca.pem分发到每个worker节点，将vink4.pem,vink4-key.pem给vink4节点，vink5.pem,vink5-key.pem给vink5节点，依次类推

## master节点

将ca.pem ca-key.pem kubernetes-key.pem kubernetes.pem service-account-key.pem service-account.pem 给每个master节点。
kube-proxy, kube-controller-manager, kube-scheduler, and kubelet暂时不用理会

下一节[生成k8s配置文件](05-kubernetes-configuration-files)