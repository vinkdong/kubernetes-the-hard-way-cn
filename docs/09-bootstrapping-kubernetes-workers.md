# 安装Worker节点

本节将配置两个work节点

## 安装依赖

```bash
apt-get update
apt-get -y install socat conntrack ipset
```

## 下载资源

```bash
wget https://dl.k8s.io/v1.13.0/kubernetes-server-linux-amd64.tar.gz
wget https://github.com/containernetworking/plugins/releases/download/v0.7.4/cni-plugins-amd64-v0.7.4.tgz

tar -xvzf kubernetes-server-linux-amd64.tar.gz
cd kubernetes/server/bin/
mv kubectl kube-proxy kubelet /usr/local/bin/
cd ../../../

tar -xvf cni-plugins-amd64-v0.7.4.tgz -C /opt/cni/bin/
```

- 创建存放配置文件的文件夹

```bash
mkdir -p \
  /etc/cni/net.d \
  /opt/cni/bin \
  /var/lib/kubelet \
  /var/lib/kube-proxy \
  /var/lib/kubernetes \
  /var/run/kubernetes
```

## 安装docker 

```bash
curl -fsSL https://get.docker.com | bash -s docker --mirror Aliyun
```

## 配置cni网络(每个work分别指定)

注意此处$i在每个work中替换成不同的数字

```bash
POD_CIDR=10.200.$i.0/24
cat <<EOF | sudo tee /etc/cni/net.d/10-bridge.conf
{
    "cniVersion": "0.3.1",
    "name": "bridge",
    "type": "bridge",
    "bridge": "cnio0",
    "isGateway": true,
    "ipMasq": true,
    "ipam": {
        "type": "host-local",
        "ranges": [
          [{"subnet": "${POD_CIDR}"}]
        ],
        "routes": [{"dst": "0.0.0.0/0"}]
    }
}
EOF

cat <<EOF | sudo tee /etc/cni/net.d/99-loopback.conf
{
    "cniVersion": "0.3.1",
    "type": "loopback"
}
EOF
```

## 配置Kubelet

```bash
HOSTNAME=$(hostname -s)
mv ${HOSTNAME}-key.pem ${HOSTNAME}.pem /var/lib/kubelet/
mv ${HOSTNAME}.kubeconfig /var/lib/kubelet/kubeconfig
mv ca.pem /var/lib/kubernetes/
```

注意此处$i在每个work中替换成前文中的数字

```bash
HOSTNAME=$(hostname -s)
POD_CIDR=10.200.$i.0/24
cat <<EOF | sudo tee /var/lib/kubelet/kubelet-config.yaml
kind: KubeletConfiguration
apiVersion: kubelet.config.k8s.io/v1beta1
authentication:
  anonymous:
    enabled: false
  webhook:
    enabled: true
  x509:
    clientCAFile: "/var/lib/kubernetes/ca.pem"
authorization:
  mode: Webhook
clusterDomain: "cluster.local"
clusterDNS:
  - "10.32.0.10"
podCIDR: "${POD_CIDR}"
resolvConf: "/run/systemd/resolve/resolv.conf"
runtimeRequestTimeout: "15m"
tlsCertFile: "/var/lib/kubelet/${HOSTNAME}.pem"
tlsPrivateKeyFile: "/var/lib/kubelet/${HOSTNAME}-key.pem"
EOF
```

- 创建kubelet服务

```bash
cat <<EOF | sudo tee /etc/systemd/system/kubelet.service
[Unit]
Description=Kubernetes Kubelet
Documentation=https://github.com/kubernetes/kubernetes
After=containerd.service
Requires=containerd.service

[Service]
ExecStart=/usr/local/bin/kubelet \\
  --config=/var/lib/kubelet/kubelet-config.yaml \\
  --image-pull-progress-deadline=2m \\
  --kubeconfig=/var/lib/kubelet/kubeconfig \\
  --network-plugin=cni \\
  --register-node=true \\
  --v=2
Restart=on-failure
RestartSec=5

[Install]
WantedBy=multi-user.target
EOF
```

## 配置Kube-proxy

```bash
mv kube-proxy.kubeconfig /var/lib/kube-proxy/kubeconfig

cat <<EOF | sudo tee /var/lib/kube-proxy/kube-proxy-config.yaml
kind: KubeProxyConfiguration
apiVersion: kubeproxy.config.k8s.io/v1alpha1
clientConnection:
  kubeconfig: "/var/lib/kube-proxy/kubeconfig"
mode: "iptables"
clusterCIDR: "10.200.0.0/16"
EOF

```

- 创建Kube-proxy服务

```bash
cat <<EOF | sudo tee /etc/systemd/system/kube-proxy.service
[Unit]
Description=Kubernetes Kube Proxy
Documentation=https://github.com/kubernetes/kubernetes

[Service]
ExecStart=/usr/local/bin/kube-proxy \\
  --config=/var/lib/kube-proxy/kube-proxy-config.yaml
Restart=on-failure
RestartSec=5

[Install]
WantedBy=multi-user.target
EOF
```

## 禁用swap

- 执行 `swapoff -a`
- 注释 `/etc/fstab`中关于swap的行

## 启动服务

```bash
systemctl daemon-reload
systemctl enable docker kubelet kube-proxy
systemctl start docker kubelet kube-proxy
```

## 测试

回到任意master节点执行

```bash
kubectl get no --kubeconfig admin.kubeconfig
```

```log
NAME    STATUS   ROLES    AGE     VERSION
vk004   Ready    <none>   64s     v1.13.0
vk005   Ready    <none>   7m38s   v1.13.0
```

下节[部署网络](10-pod-network-routes.md)