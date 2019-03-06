# 本节将引导部署DNS

部署DNS

```bash
kubectl apply -f https://storage.googleapis.com/kubernetes-the-hard-way/coredns.yaml
```

- 测试

```bash
kubectl run t-server --image vinkdong/defaultbackend:1.4 
kubectl expose deployment t-server --port 8080
kubectl run t-05 -ti --image vinkdong/net-tools bash
curl t-server:8080
```

应该有返回结果

```bash
default backend - 404
```

完成。