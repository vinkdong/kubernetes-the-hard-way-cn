此项目源于[kelseyhightower/kubernetes-the-hard-way](https://github.com/kelseyhightower/kubernetes-the-hard-way)启发，主要做了一些翻译和使用阿里云替代谷歌云.
本文不是写给小白的，如果对linux命令和docker一点都不了解，不建议参考本文。请不要提类似如何安装docker、curl之类的问题。

本文分为以下章节：

- [准备](docs/01-prerequisites.md)
- [安装客户端工具](docs/02-client-tools.md)
- [准备资源](docs/03-compute-resources.md)
- [准备证书](docs/04-certificate-authority.md)
- [生成k8s配置文件](docs/05-kubernetes-configuration-files)
- [创建加密配置](docs/06-data-encryption-keys)
- [安装etcd](docs/07-bootstrapping-etcd)
- [配置Kubernetes控制节点](docs/08-bootstrapping-kubernetes-controllers)
- [配置Work节点](docs/09-bootstrapping-kubernetes-workers.md)
- [部署网络](docs/10-pod-network-routes.md)
- [部署DNS](docs/11-dns-addon.md)