1- Configmap

```
#config.yml
apiVersion: v1
kind: Namespace
metadata:
  name: kwatch
---
apiVersion: v1
kind: ConfigMap
metadata:
  name: kwatch
  namespace: kwatch
data:
  config.yaml: |
    alert:
      telegram:
        token: TOKEN #====> PUT IT IN DUBLE QOUTE
        chatId: CHAT_ID  #====> PUT IT IN DUBLE QOUTE
```
```
kubectl apply -f config.yml
```
2- Deploy Kwatch 
```
kubectl apply -f https://raw.githubusercontent.com/abahmed/kwatch/v0.10.1/deploy/deploy.yaml

```
