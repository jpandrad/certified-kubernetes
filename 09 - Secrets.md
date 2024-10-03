# Secrets
A `Secret` segue a mesma base do `ConfigMap` a diferenca e que os dados sao criptografados.

Quando falamos em secrets, ela e criptografada em base64, abaixo um exemplo simples de criptografar e descriptografar em base64. Lembrando que base64 nao e uma criptografia avancada, porem e muito utilizada entre comunicacao por nao passar texto em branco ou ate mesmo quebrar caracteres durante a transeferencia.

```shell
echo "test" | base64
dGVzdAo=
```

Para descriptografar, basta passar o valor do dado criptografado e adicionar um `-d` apos o base64:
```shell
echo "dGVzdAo=" | base64 -d
test
```

Por padrao, o valor em base64 e criado contendo uma quebra de linha, e se utilizado no Kubernetes, pode ocorrer alguns problemas, para isso, precisamos utilizar um parametro `-n` antes do valor a ser codificado:

codificacao padrao
```shell
echo "text" | base64 
dGV4dAo=
```

codificacao utilizando o parametro `-n`:
```shell
controlplane ~ ➜  echo -n "text" | base64 
dGV4dA==
```

Perceba que o valor codificado e totalmente diferente, se fizermos um decode, podem ver que no primeiro caso temo a quebra de linha e no segundo nao:
```shell
controlplane ~ ➜  echo "dGV4dAo=" | base64 -d
text

controlplane ~ ➜  echo "dGV4dA==" | base64 -d
text
```


## Criando a nossa Secret
Vamos gerar as informacoes em base64, utilize o parametro `-n` para ignorar a quebra de linha:
```shell
echo -n "usuario01" | base64
dXN1YXJpbzAx
```

```shell
echo "senha01" | base64
c2VuaGEwMQ==
```

Com as informacoes ja codificadas em base64, vamos criar um arquivo chamado `secret.yaml`:
```shell
vim secret.yaml
```
Com o seguinte conteudo:
```yaml
apiVersion: v1
kind: Secret
metadata:
  name: my-secret
data:
  user: dXN1YXJpbzAxCg==
  pass: c2VuaGEwMQo=
```

Vamos criar a nossa secret:
```shell
kubectl apply -f secret.yaml 
secret/my-secret created
```

Vamos ver se a nossa secret foi criada corretamente:
```shell
kubectl get secrets
NAME        TYPE     DATA   AGE
my-secret   Opaque   2      35s
```

Para consultar a nossa secret, vamos fazer um `kubectl get secret` saindo em formatacao yaml:
```shell
kubectl get secret my-secret -o yaml
apiVersion: v1
data:
  pass: c2VuaGEwMQo=
  user: dXN1YXJpbzAxCg==
kind: Secret
metadata:
  annotations:
    kubectl.kubernetes.io/last-applied-configuration: |
      {"apiVersion":"v1","data":{"pass":"c2VuaGEwMQo=","user":"dXN1YXJpbzAxCg=="},"kind":"Secret","metadata":{"annotations":{},"name":"my-secret","namespace":"default"}}
  creationTimestamp: "2024-10-03T14:27:13Z"
  name: my-secret
  namespace: default
  resourceVersion: "5076"
  uid: 3bc44ce9-721e-4693-b2f6-20f1454490b6
type: Opaque
```


Podemos criar uma secret utilizando o comando `kubectl create secret`: 

Para a criacao, temos 3 tipos de secrets:
1. docker-registry: Utilizada para o Docker registry.
2. generic: Utilizada para valores gerais, diretorios, arquivos locais e etc.
3. tls: utilizada para comunicacao tls de um ingress.

Vamos criar uma secret do tipo generic passando os valor utilizando o `--from-literal=`:
```shell
kubectl create secret generic sec-secret --from-literal=user=usuario01 --from-literal=pass=senha01
secret/sec-secret created
```

Vamos verificar se a nossa secret foi devidamente criada:
```shell
kubectl get secrets
NAME         TYPE     DATA   AGE
my-secret    Opaque   2      9m15s
sec-secret   Opaque   2      92s
```

Vamos fazer um get na secret:
```shell
kubectl get secret sec-secret -o yaml
apiVersion: v1
data:
  pass: c2VuaGEwMQ==
  user: dXN1YXJpbzAx
kind: Secret
metadata:
  creationTimestamp: "2024-10-03T14:34:56Z"
  name: sec-secret
  namespace: default
  resourceVersion: "5774"
  uid: 665228e2-34a2-490a-b062-7dc55ab78f6b
type: Opaque
```

Os valores sao iguais em ambos os casos (Desconsidere nesta documentacao, pois utilizei informacoes diferentes para gerar os metodos). Na prova sempre de preferencia em criar atraves do comando `kubectl create secret` pois evita erros durante a criacao.


## Editando a secret
Podemos editar uma secret normalmente atraves do manifesto ou tambem via `kubectl edit secret`:

Gerando um novo usuario chamado `adminadmin`:
```shell
echo -n "adminadmin" | base64
YWRtaW5hZG1pbg==
```


```shell
kubectl edit secret sec-secret
```

Content
```yaml
apiVersion: v1
data:
  pass: c2VuaGEwMQ==
  user: YWRtaW5hZG1pbg==
kind: Secret
metadata:
  creationTimestamp: "2024-10-03T14:34:56Z"
  name: sec-secret
  namespace: default
  resourceVersion: "6789"
  uid: 665228e2-34a2-490a-b062-7dc55ab78f6b
type: Opaque
```

Apos alterado, segue a mesma logica do ConfigMap, precisa reiniciar o Pod para aplicar as alteracoes.


## Utilizando a Secret
Vamos criar um pod:
```shell
kubectl run nginx --image nginx --dry-run=client -o yaml > pod-secret.yaml
```

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
    - secretRef:
        name: my-secret
  dnsPolicy: ClusterFirst
  restartPolicy: Always
```

Aplicando o Pod:
```shell
kubectl apply -f pod-secret.yaml 
pod/nginx created
```

Como criamos via variavel, podemos visualizar utilizando o comando exec:
```shell
kubectl exec -it nginx -- env
PATH=/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/sbin:/bin
HOSTNAME=nginx
NGINX_VERSION=1.27.2
NJS_VERSION=0.8.6
NJS_RELEASE=1~bookworm
PKG_RELEASE=1~bookworm
DYNPKG_RELEASE=1~bookworm
user=usuario01
pass=senha01
KUBERNETES_PORT_443_TCP_PORT=443
KUBERNETES_PORT_443_TCP_ADDR=10.96.0.1
KUBERNETES_SERVICE_HOST=10.96.0.1
KUBERNETES_SERVICE_PORT=443
KUBERNETES_SERVICE_PORT_HTTPS=443
KUBERNETES_PORT=tcp://10.96.0.1:443
KUBERNETES_PORT_443_TCP=tcp://10.96.0.1:443
KUBERNETES_PORT_443_TCP_PROTO=tcp
TERM=xterm
HOME=/root
```

Para utilizar apenas uma unica informacao, podemos utilizar 

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
    - name: MY_PASS
      valueFrom:
        secretKeyRef:
          name: sec-secret
          key: pass
  dnsPolicy: ClusterFirst
  restartPolicy: Always
```

Vamos reiniciar o Pod:
```shell
kubectl replace -f pod-secret.yaml --force --grace-period 0
pod/nginx configured
```

pegando os valores:
```shell
kubectl exec -it nginx -- env
PATH=/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/sbin:/bin
HOSTNAME=nginx
NGINX_VERSION=1.27.2
NJS_VERSION=0.8.6
NJS_RELEASE=1~bookworm
PKG_RELEASE=1~bookworm
DYNPKG_RELEASE=1~bookworm
MY_PASS=senha01
KUBERNETES_PORT_443_TCP_PROTO=tcp
KUBERNETES_PORT_443_TCP_PORT=443
KUBERNETES_PORT_443_TCP_ADDR=10.96.0.1
KUBERNETES_SERVICE_HOST=10.96.0.1
KUBERNETES_SERVICE_PORT=443
KUBERNETES_SERVICE_PORT_HTTPS=443
KUBERNETES_PORT=tcp://10.96.0.1:443
KUBERNETES_PORT_443_TCP=tcp://10.96.0.1:443
TERM=xterm
HOME=/root
```

Para criar via volume:
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
    - name: secret
      mountPath: /data
  dnsPolicy: ClusterFirst
  restartPolicy: Always
  volumes:
  - name: secret
    secret:
      secretName: sec-secret
```

Forcando um replace:
```shell
kubectl replace -f pod-secret.yaml --force --grace-period 0
pod "nginx" deleted
pod/nginx replaced
```

Vamos ver no Pod se foi criado:
```shell
kubectl exec -it nginx -- ls /data
pass  user
```

Podemos ver o valor efetuando um cat nos arquivos:
```shell
controlplane ~ ➜  kubectl exec -it nginx -- cat /data/user
adminadmin
controlplane ~ ✖ kubectl exec -it nginx -- cat /data/pass
senha01
```

Para poder pegar somente um valor, pode utilizar o `items`:
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
    - name: secret
      mountPath: /data
  dnsPolicy: ClusterFirst
  restartPolicy: Always
  volumes:
  - name: secret
    secret:
      secretName: sec-secret
      items:
      - key: pass
        path: pass.config
```

Apos a alteracao, vamos forcar o replace no pod:
```shell
kubectl replace -f pod-secret.yaml --force --grace-period 0
pod "nginx" deleted
pod/nginx replaced
```

Vamos verificar se o arquivo foi criado corretamente:
```shell
kubectl exec -it nginx -- cat /data/pass.config
senha01
```


Podemos tambem definir a permissao do arquivo utilizando o `defaultMode`:
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
    - name: secret
      mountPath: /data
  dnsPolicy: ClusterFirst
  restartPolicy: Always
  volumes:
  - name: secret
    secret:
      secretName: sec-secret
      defaultMode: 0400
      items:
      - key: pass
        path: pass.config
```

E tambem podemos fazer isso diretamente na secret:
```shell
kubectl edit secret sec-secret
```

```yaml
apiVersion: v1
data:
  pass: c2VuaGEwMQ==
  user: YWRtaW5hZG1pbg==
immutable: true
kind: Secret
metadata:
  creationTimestamp: "2024-10-03T14:34:56Z"
  name: sec-secret
  namespace: default
  resourceVersion: "9862"
  uid: 665228e2-34a2-490a-b062-7dc55ab78f6b
type: Opaque
```