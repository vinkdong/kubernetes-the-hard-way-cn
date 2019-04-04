# 生成Kubernetes配置文件

## 为kubelet，proxy，scheduler，controller生成配置文件

先准备一个lb负载到每个master节点的6443端口，以便实现高可用架构，如果没有lb，可以在每个节点部署一个haproxy或者nginx负载到每个master节点上。阿里云可以购买内网的负载均衡就可以目前是免费的。按照提示添加master节点即可，端口全为6443。注意开放安全组。

这里我创建好的lb地址为`172.26.192.67`。

### 同时在两个work节点上执行命令,或者在本机执行分发给对应work节点也是可以的

```bash
export KUBERNETES_PUBLIC_ADDRESS=172.26.192.67
export instance=$(hostname)
kubectl config set-cluster kubernetes-hand-cn \
    --certificate-authority=ca.pem \
    --embed-certs=true \
    --server=https://${KUBERNETES_PUBLIC_ADDRESS}:6443 \
    --kubeconfig=${instance}.kubeconfig

kubectl config set-credentials system:node:${instance} \
    --client-certificate=${instance}.pem \
    --client-key=${instance}-key.pem \
    --embed-certs=true \
    --kubeconfig=${instance}.kubeconfig

kubectl config set-context default \
    --cluster=kubernetes-hand-cn \
    --user=system:node:${instance} \
    --kubeconfig=${instance}.kubeconfig

# 配置默认context
kubectl config use-context default --kubeconfig=${instance}.kubeconfig
```

### 在04节中创建证书的位置执行下述命令，生成配置文件

创建proxy配置文件

```bash
export KUBERNETES_PUBLIC_ADDRESS=172.26.192.67
kubectl config set-cluster kubernetes-hand-cn \
    --certificate-authority=ca.pem \
    --embed-certs=true \
    --server=https://${KUBERNETES_PUBLIC_ADDRESS}:6443 \
    --kubeconfig=kube-proxy.kubeconfig

kubectl config set-credentials system:kube-proxy \
    --client-certificate=kube-proxy.pem \
    --client-key=kube-proxy-key.pem \
    --embed-certs=true \
    --kubeconfig=kube-proxy.kubeconfig

kubectl config set-context default \
    --cluster=kubernetes-hand-cn \
    --user=system:kube-proxy \
    --kubeconfig=kube-proxy.kubeconfig

kubectl config use-context default --kubeconfig=kube-controller-manager.kubeconfig
```

创建controller配置文件

```bash

kubectl config set-cluster kubernetes-hard-cn \
    --certificate-authority=ca.pem \
    --embed-certs=true \
    --server=https://127.0.0.1:6443 \
    --kubeconfig=kube-controller-manager.kubeconfig

kubectl config set-credentials system:kube-controller-manager \
    --client-certificate=kube-controller-manager.pem \
    --client-key=kube-controller-manager-key.pem \
    --embed-certs=true \
    --kubeconfig=kube-controller-manager.kubeconfig

kubectl config set-context default \
    --cluster=kubernetes-hard-cn \
    --user=system:kube-controller-manager \
    --kubeconfig=kube-controller-manager.kubeconfig

kubectl config use-context default --kubeconfig=kube-controller-manager.kubeconfig
```

创建scheduler配置文件

```bash
kubectl config set-cluster kubernetes-hard-cn \
    --certificate-authority=ca.pem \
    --embed-certs=true \
    --server=https://127.0.0.1:6443 \
    --kubeconfig=kube-scheduler.kubeconfig

kubectl config set-credentials system:kube-scheduler \
    --client-certificate=kube-scheduler.pem \
    --client-key=kube-scheduler-key.pem \
    --embed-certs=true \
    --kubeconfig=kube-scheduler.kubeconfig

kubectl config set-context default \
    --cluster=kubernetes-hard-cn \
    --user=system:kube-scheduler \
    --kubeconfig=kube-scheduler.kubeconfig

kubectl config use-context default --kubeconfig=kube-scheduler.kubeconfig
```

创建admin配置文件

```bash
kubectl config set-cluster kubernetes-hard-cn \
    --certificate-authority=ca.pem \
    --embed-certs=true \
    --server=https://127.0.0.1:6443 \
    --kubeconfig=admin.kubeconfig

kubectl config set-credentials admin \
    --client-certificate=admin.pem \
    --client-key=admin-key.pem \
    --embed-certs=true \
    --kubeconfig=admin.kubeconfig

kubectl config set-context default \
    --cluster=kubernetes-hard-cn \
    --user=admin \
    --kubeconfig=admin.kubeconfig

kubectl config use-context default --kubeconfig=admin.kubeconfig
```

##### 将上述分发配置文件分发到所有master节点

```
scp admin.kubeconfig kube-controller-manager.kubeconfig kube-scheduler.kubeconfig root@vk001:/root
scp admin.kubeconfig kube-controller-manager.kubeconfig kube-scheduler.kubeconfig root@vk002:/root
scp admin.kubeconfig kube-controller-manager.kubeconfig kube-scheduler.kubeconfig root@vk003:/root
```
##### 将proxy配置文件分发到所有worker节点

```
scp kube-proxy.kubeconfig root@vk004:/root
scp kube-proxy.kubeconfig root@vk005:/root
```


下一章[创建加密配置](06-data-encryption-keys.md)
