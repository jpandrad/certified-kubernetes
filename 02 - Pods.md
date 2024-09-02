# Pods

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: primeiro-pod 
spec:
  containers:
  - image: nginx
    name: nginx
```
---

Criando um pod simples utilizando o `dry run`:
```shell
$ kubectl run nginx --image nginx -o yaml --dry-run=client
```

Fazendo replace de um pod:
```shell
$ kubectl replace -f pod.yaml --force --grace-period=0
```

Editando um pod em tempo de execucao:
```shell
$ kubectl edit pod primeiro-pod
```

Editando um pod utilizando o arquivo yaml:
```shell
$ kubectl get pod primeiro-pod -o yaml > pod.yaml
```

Forcando a exclusao de um pod imediatamente:
```shell
$ kubectl delete pod nginx --force --grace-period 0
```

Forcando o replace de um pod imediatamente:
```shell
$ kubectl replace -f pod.yaml --force --grace-period 0
```

Exemplo de multiple container:
```yaml
apiVersion: v1
kind: Pod
metadata:
  name: primeiro-pod 
spec:
  containers:
  - image: nginx
    name: nginx
    ports:
    - containerPort: 80
  - image: redis
    name: redis
    ports:
    - containerPort: 6379    
```