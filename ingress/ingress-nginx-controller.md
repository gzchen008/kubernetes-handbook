## 全局配置nginx.conf

### SSL配置
使用ingress-nginx-controller 的 ConfigMap 示例配置中设置 SSL 加密套件的示例，可以根据实际需求进行修改：

```
apiVersion: v1
kind: ConfigMap
metadata:
  name: ingress-nginx-controller
  namespace: ingress-nginx
data:
  ssl-ciphers: "ECDHE-ECDSA-AES256-GCM-SHA384:ECDHE-RSA-AES256-GCM-SHA384"
```

在上面的示例中，ssl-ciphers 参数设置为 ECDHE-ECDSA-AES256-GCM-SHA384 和 ECDHE-RSA-AES256-GCM-SHA384，这些是相对安全的加密算法。
