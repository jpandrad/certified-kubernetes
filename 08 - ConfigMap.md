# ConfigMap e Secrets
O ConfigMap e Secrets sao as formas de armazenar informacoes sensiveis e nao sensiveis. A diferenca e que o ConfigMap nao e criptografado em quanto o Secrets e criptografado.

## Criando um ConfigMap de forma simples
Vamos criar um arquivo chamado configmap.yaml:
```shell
vim configmap.yaml
```
Conteudo do arquivo:
```yaml
apiVersion: v1
kind: ConfigMap
metadata:
  name: primeiro-cmap
data:
  ip: "192.168.0.1"
  server: "web"
```

Vamos fazer o apply do arquivo que foi criado:
```shell
kubectl apply -f configmap.yaml 
configmap/primeiro-cmap created
```

Vamos consultar se o configMap foi criado:
```shell
kubectl get cm
NAME               DATA   AGE
kube-root-ca.crt   1      6h1m
primeiro-cmap      2      23s
```

Podemos ver o conteudo do arquivo:
```shell
kubectl get cm primeiro-cmap -o yaml
```
OUTPUT
```yaml
apiVersion: v1
data:
  ip: 192.168.0.1
  server: web
kind: ConfigMap
metadata:
  annotations:
    kubectl.kubernetes.io/last-applied-configuration: |
      {"apiVersion":"v1","data":{"ip":"192.168.0.1","server":"web"},"kind":"ConfigMap","metadata":{"annotations":{},"name":"primeiro-cmap","namespace":"default"}}
  creationTimestamp: "2024-09-17T22:46:59Z"
  name: primeiro-cmap
  namespace: default
  resourceVersion: "33190"
  uid: a900e107-c1b5-42b6-8f11-3e25b4129b48
```

Tambem podemos fazer um describe
```shell
kubectl describe cm primeiro-cmap
```
OUTPUT
```shell
Name:         primeiro-cmap
Namespace:    default
Labels:       <none>
Annotations:  <none>

Data
====
ip:
----
192.168.0.1

server:
----
web


BinaryData
====

Events:  <none>
```

Vamos subir um pod para consultar o nosso configMap. Criando um 
```shell
kubectl run nginx --image nginx --dry-run=client -o yaml > pod.yaml
```

No arquivo pod.yaml, vamos colocar o seguinte bloco para podermos chamar o configmap como variavel:
```yaml
    envFrom:
    - configMapRef:
        name: primeiro-cmap
```

O Arquivo ficara desta forma:
```yaml
apiVersion: v1
kind: Pod
metadata:
  creationTimestamp: null
  labels:
    run: nginx
  name: nginx
spec:
  containers:
  - image: nginx
    name: nginx
    resources: {}
    envFrom:
    - configMapRef:
        name: primeiro-cmap
  dnsPolicy: ClusterFirst
  restartPolicy: Always
```

Fazendo uma consulta nas variaveis, podemos ver que temos a variavel ip e server:
```shell
kubectl exec -it nginx -c nginx -- env
```
OUTPUT
```shell
PATH=/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/sbin:/bin
HOSTNAME=nginx
NGINX_VERSION=1.27.1
NJS_VERSION=0.8.5
NJS_RELEASE=1~bookworm
PKG_RELEASE=1~bookworm
DYNPKG_RELEASE=2~bookworm
ip=192.168.0.1
server=web
KUBERNETES_PORT_443_TCP=tcp://10.96.0.1:443
KUBERNETES_PORT_443_TCP_PROTO=tcp
KUBERNETES_PORT_443_TCP_PORT=443
KUBERNETES_PORT_443_TCP_ADDR=10.96.0.1
KUBERNETES_SERVICE_HOST=10.96.0.1
KUBERNETES_SERVICE_PORT=443
KUBERNETES_SERVICE_PORT_HTTPS=443
KUBERNETES_PORT=tcp://10.96.0.1:443
TERM=xterm
HOME=/root
```


Para que possamos passar item a item precisamos utilizar o `valueFrom`:
```yaml
apiVersion: v1
kind: Pod
metadata:
  creationTimestamp: null
  labels:
    run: nginx
  name: nginx
spec:
  containers:
  - image: nginx
    name: nginx
    resources: {}
    env:
    - name : MYIP
      valueFrom:
       configMapKeyRef: primeiro-cmap
       key: ip
#    envFrom:
#    - configMapRef:
#        name: primeiro-cmap
  dnsPolicy: ClusterFirst
  restartPolicy: Always
```


Outra forma e montar o config map como `volume`:
```yaml
apiVersion: v1
kind: Pod
metadata:
  creationTimestamp: null
  labels:
    run: nginx
  name: nginx
spec:
  containers:
  - image: nginx
    name: nginx
    resources: {}
    volumeMounts:
    - name: configmap-volume
      mountPath: /data
  dnsPolicy: ClusterFirst
  restartPolicy: Always
  volumes:
  - name: configmap-volume
    configmap:
      name: primeiro-cmap
```


Adicionando o `items` podemos especificar apenas uma informacao do nosso configmap como tambem colocar ele em um arquivo:
```yaml
apiVersion: v1
kind: Pod
metadata:
  creationTimestamp: null
  labels:
    run: nginx
  name: nginx
spec:
  containers:
  - image: nginx
    name: nginx
    resources: {}
    volumeMounts:
    - name: configmap-volume
      mountPath: /data
  dnsPolicy: ClusterFirst
  restartPolicy: Always
  volumes:
  - name: configmap-volume
    configmap:
      name: primeiro-cmap
      items:
      - key: ip
        path: mmyip.conf
```


## Criando um ConfigMap de forma rapida

```shell
kubectl create configmap second-cmap --from-literal=ip=10.0.0.2 --dry-run=client -o yaml
```
```yaml
apiVersion: v1
data:
  ip: 10.0.0.2
kind: ConfigMap
metadata:
  creationTimestamp: null
  name: second-cmap
```

```shell
kubectl create configmap second-cmap --from-literal=ip=10.0.0.2 --from-literal=tier=web --dry-run=client -o yaml
```
```yaml
apiVersion: v1
data:
  ip: 10.0.0.2
  tier: web
kind: ConfigMap
metadata:
  creationTimestamp: null
  name: second-cmap
```

Podemos editar ele utilizando o comando `kubectl edit cm`:
```shell
kubectl edit cm second-cmap
configmap/second-cmap edited
```

Depois de alterado o `ConfigMap` devera ser feito um restart no Pod para aplicar as alteracoes.


## ConfigMap Imutavel

Para deixar o ConfigMap sem alteracao, adicione o `immutable` para `true`:
```yaml
apiVersion: v1
kind: ConfigMap
metadata:
  name: second-cmap
data:
  ip: 10.0.0.2
  tier: web
immutable: true
```


## PROVA
1. Como criar de forma rapida
2. As formas para Exportar para o ConfigMap