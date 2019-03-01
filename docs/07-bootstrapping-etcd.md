# 安装ETCD

kubernetes的集群相关数据主要存储在etcd中。下边配置一个高可用的etcd集群。

## 安装

### 使用iterm2或者tmux同时进入到三个master节点中

同时执行下述命令安装etcd

```bash
# 下载会有点慢 注意保持连接
wget -q --show-progress --https-only --timestamping https://github.com/etcd-io/etcd/releases/download/v3.3.12/etcd-v3.3.12-linux-amd64.tar.gz

tar -xvf etcd-v3.3.12-linux-amd64.tar.gz
mv etcd-v3.3.12-linux-amd64/etcd* /usr/local/bin/
```

创建配置文件

```bash
mkdir -p /etc/etcd /var/lib/etcd
cp ca.pem kubernetes-key.pem kubernetes.pem /etc/etcd/
```

配置service

```bash
INTERNAL_IP=$(ip addr show eth0 | grep 'inet ' | awk '{print $2}' | cut -f1 -d'/') 
# 上边这一句只是为了获取内网ip.
ETCD_NAME=$(hostname -s)
ETCD_CLUSTER="vk001=https://172.26.192.62:2380,vk002=https://172.26.192.65:2380,vk003=https://172.26.192.66:2380"

cat <<EOF | sudo tee /etc/systemd/system/etcd.service
[Unit]
Description=etcd
Documentation=https://github.com/coreos

[Service]
ExecStart=/usr/local/bin/etcd \\
  --name ${ETCD_NAME} \\
  --cert-file=/etc/etcd/kubernetes.pem \\
  --key-file=/etc/etcd/kubernetes-key.pem \\
  --peer-cert-file=/etc/etcd/kubernetes.pem \\
  --peer-key-file=/etc/etcd/kubernetes-key.pem \\
  --trusted-ca-file=/etc/etcd/ca.pem \\
  --peer-trusted-ca-file=/etc/etcd/ca.pem \\
  --peer-client-cert-auth \\
  --client-cert-auth \\
  --initial-advertise-peer-urls https://${INTERNAL_IP}:2380 \\
  --listen-peer-urls https://${INTERNAL_IP}:2380 \\
  --listen-client-urls https://${INTERNAL_IP}:2379,https://127.0.0.1:2379 \\
  --advertise-client-urls https://${INTERNAL_IP}:2379 \\
  --initial-cluster-token etcd-cluster-0 \\
  --initial-cluster ${ETCD_CLUSTER} \\
  --initial-cluster-state new \\
  --data-dir=/var/lib/etcd
Restart=on-failure
RestartSec=5

[Install]
WantedBy=multi-user.target
EOF

systemctl daemon-reload
systemctl enable etcd
systemctl start etcd
```

检测是否成功

```
ETCDCTL_API=3 etcdctl member list \
  --endpoints=https://127.0.0.1:2379 \
  --cacert=/etc/etcd/ca.pem \
  --cert=/etc/etcd/kubernetes.pem \
  --key=/etc/etcd/kubernetes-key.pem
```

输出类似以下内容就表示成功了
```bash
1325837a1719e3af, started, vk001, https://172.26.192.62:2380, https://172.26.192.62:2379
39fbc4cdde58dd99, started, vk003, https://172.26.192.66:2380, https://172.26.192.66:2379
9f4096f003caee88, started, vk002, https://172.26.192.65:2380, https://172.26.192.65:2379
```

下一节[安装Kubernetes控制面板](08-bootstrapping-kubernetes-controllers)

