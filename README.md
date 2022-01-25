# Filebeat-Sidecar

在 K8s 使用 Filebeat 收集 Log 並以 sidecar 的方式實現

## 使用方式

部署 YAML 檔案
```
kubectl apply -f .
```

port-forward
```
kubectl port-forward [Pod_Name] 9453:80
```

開啟瀏覽器，並且將首頁的「Auto Refresh」打勾，讓服務持續產生 Access Log
```
http://127.0.0.1:9453
```

觀察 filebeat 是否將 Log 傳送到指定的收集器 elasticsearch 或 logstash

## 其他說明

加上環境變數 `TZ=Asia/Taipei` 可以修正 Filebeat 時區
```
apiVersion: apps/v1
kind: Deployment
metadata:
  ...
spec:
  ...
    spec:
      containers:
        - name: nginx
          ...
        - name: filebeat
          image: elastic/filebeat:7.16.2
          env:
            - name: TZ
              value: Asia/Taipei
          ...
      volumes:
        ...
```

因 Filebeat 需要收集 Nginx 的 Access Log，所以需要將產生 Log 的路徑進行共同掛載

> 如果不將 Nginx 路徑 `/var/log/nginx/` 額外掛載出來，Log 則會以 stdout 的方式呈現，掛載出來後則會以檔案寫入的方式

```
apiVersion: apps/v1
kind: Deployment
metadata:
  ...
spec:
  ...
    spec:
      containers:
        - name: nginx
          image: nginxdemos/hello:latest
          ...
          volumeMounts:
            - name: nginx-logs
              mountPath: /var/log/nginx/
        - name: filebeat
          image: elastic/filebeat:7.16.2
          ...
          volumeMounts:
            - name: nginx-logs
              mountPath: /var/log/nginx/
      volumes:
        - name: nginx-logs
          emptyDir: {}
```
