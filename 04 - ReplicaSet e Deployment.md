# Replicaset e Deployment
O Replicaset garante que temos varias replicas do nosso pod.

Para facilitar a criacao de um replicaset, podemos utilizar como base o Deployment

```shell
kubectl create deployment nginx --image nginx -o yaml --dry-run=client > nginx-replicaset.yaml
```

A estrutura do deployment e bem semelhante a do replicaset, sendo apenas alterar o kind de `Deployment` para `ReplicaSet` e remover o parametro `strategy` pois ele nao existe no ReplicaSet.

Arquivo gerado pelo dry-run:
```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  labels:
    app: nginx
  name: nginx
spec:
  replicas: 1
  selector:
    matchLabels:
      app: nginx
  strategy: {}
  template:
    metadata:
      creationTimestamp: null
      labels:
        app: nginx
    spec:
    containers:
    - image: nginx
      name: nginx
      resource: {}
```

Arquivo alterado para virar um replicaset:
```yaml
apiVersion: apps/v1
kind: ReplicaSet
metadata:
  labels:
    app: nginx
  name: nginx
spec:
  replicas: 1
  selector:
    matchLabels:
      app: nginx
  template:
    metadata:
      creationTimestamp: null
      labels:
        app: nginx
    spec:
    containers:
    - image: nginx
      name: nginx
      resource: {}
```

O Deployment e o controle de versionamento e deploy da aplicacao.

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: nginx-deployment
spec:
  replicas: 3
  selector:
    matchLabels:
      app: nginx
  template:
    metadata:
      labels:
        app: nginx
    spec:
      containers:
      - name: nginx
        image: nginx
```

Podemos editar o scalling da aplicacao de duas formas:

editando o deployment:
```shell
kubectl edit deployments.apps nginx
```
Apos a alteracao, a aplicacao ocorre de forma imediata.

A segunda forma e utilizando o `scale`:
```shell
kubectl scale deplyment --replicas <NUMBER_OF_REPLICAS> <DEPLOYMENT_NAME>
```

Para fazer um roolback, primeiro precisamos ver versoes que temos:
```shell
kubeclet roolout history deployment <DEPLOYMENT_NAME>
```

Com o numero do versionamento que queira recuperar, utilize o `undo` passando a revisao:
```shell
kubectl rollout undo --to-revision <NUMBER> deployment <DEPLOYMENT_NUMBER>
```


Cria um arquivo de deployment rapidamente
```shell
kubectl create deploment <NAME> --image <IMAGE_NAME> --port <PORT_NUMBER> --dry-run -o yaml >> <FILE>.yaml
```


