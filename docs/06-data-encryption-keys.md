# 创建加密配置

kubernetes使用配置的encryption-key加密资源，这里随机获取一个

```bash
ENCRYPTION_KEY=$(head -c 32 /dev/urandom | base64)
#创建配置文件
cat > encryption-config.yaml <<EOF
kind: EncryptionConfig
apiVersion: v1
resources:
  - resources:
      - secrets
    providers:
      - aescbc:
          keys:
            - name: key1
              secret: ${ENCRYPTION_KEY}
      - identity: {}
EOF
```

将该文件分发给所有mater节点

```bash
scp encryption-config.yaml root@vk001:/root
scp encryption-config.yaml root@vk002:/root
scp encryption-config.yaml root@vk003:/root
```