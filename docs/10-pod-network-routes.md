# 本文引导部署网络

## 此处选用flannel

```bash
kubectl apply -f https://raw.githubusercontent.com/coreos/flannel/master/Documentation/kube-flannel.yml
```

## 查看部署结果

```bash
kubectl get po -n kube-system
```

```log
NAME                          READY   STATUS    RESTARTS   AGE
kube-flannel-ds-amd64-j9wwd   1/1     Running   0          1s
kube-flannel-ds-amd64-x2gr6   1/1     Running   0          1s
```

## 启动ipv4 forward

```bash
echo net.ipv4.ip_forward=1 >> /etc/sysctl.conf
sysctl -w net.ipv4.ip_forward=1
sysctl net.ipv4.ip_forward
```

下一节[部署DNS](11-dns-addon.md)
