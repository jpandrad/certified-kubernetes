# EmptyDir
O EmptyDir e um volume efemero, sendo que quando finalizamos o nosso pod os dados sao perdidos.

Criando o nosso pod:
```shell
kubectl run nginx --image nginx --dry-run=client -o yaml > pod.yaml
```

Com o arquivo de YAML criado, precisamos adicionar o bloco de volumes e na secao spec, adicionar o bloco de volumeMounts:
```yaml
...
spec:
...
    volumeMounts:
    - mountPath: /data
      name: empty
...
  volumes:
  - name: empty
    emptyDir: {}
```

O arquivo ficara desta forma:
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
    - mountPath: /data
      name: empty
  dnsPolicy: ClusterFirst
  restartPolicy: Always
  volumes:
  - name: empty
    emptyDir: {}
```

Vamos fazer o deploy do nosso arquivo:
```shell
kubectl apply -f pod.yaml 
pod/nginx created
```

Realizando um describe no nosso pod, podemos ver que foi criado um EmptyDir:
```shell
kubectl describe pod nginx
```
OUTPUT
```shell
Name:             nginx
Namespace:        default
Priority:         0
Service Account:  default
Node:             node01/192.43.109.3
Start Time:       Tue, 17 Sep 2024 22:27:58 +0000
Labels:           run=nginx
Annotations:      <none>
Status:           Running
IP:               10.244.1.2
IPs:
  IP:  10.244.1.2
Containers:
  nginx:
    Container ID:   containerd://19897e0a6e0bc8b80f8ddc77c289ac44491ea45a75b4141d9be4ae5c6f1eedf0
    Image:          nginx
    Image ID:       docker.io/library/nginx@sha256:04ba374043ccd2fc5c593885c0eacddebabd5ca375f9323666f28dfd5a9710e3
    Port:           <none>
    Host Port:      <none>
    State:          Running
      Started:      Tue, 17 Sep 2024 22:28:04 +0000
    Ready:          True
    Restart Count:  0
    Environment:    <none>
    Mounts:
      /data from empty (rw)
      /var/run/secrets/kubernetes.io/serviceaccount from kube-api-access-psmgh (ro)
Conditions:
  Type                        Status
  PodReadyToStartContainers   True 
  Initialized                 True 
  Ready                       True 
  ContainersReady             True 
  PodScheduled                True 
Volumes:
  empty:
    Type:       EmptyDir (a temporary directory that shares a pod's lifetime)
    Medium:     
    SizeLimit:  <unset>
  kube-api-access-psmgh:
    Type:                    Projected (a volume that contains injected data from multiple sources)
    TokenExpirationSeconds:  3607
    ConfigMapName:           kube-root-ca.crt
    ConfigMapOptional:       <nil>
    DownwardAPI:             true
QoS Class:                   BestEffort
Node-Selectors:              <none>
Tolerations:                 node.kubernetes.io/not-ready:NoExecute op=Exists for 300s
                             node.kubernetes.io/unreachable:NoExecute op=Exists for 300s
Events:
  Type    Reason     Age   From               Message
  ----    ------     ----  ----               -------
  Normal  Scheduled  52s   default-scheduler  Successfully assigned default/nginx to node01
  Normal  Pulling    51s   kubelet            Pulling image "nginx"
  Normal  Pulled     46s   kubelet            Successfully pulled image "nginx" in 5.08s (5.08s including waiting). Image size: 71027698 bytes.
  Normal  Created    46s   kubelet            Created container nginx
  Normal  Started    46s   kubelet            Started container nginx
```


Vamos criar mais um pod e compartilhar o mesmo volumeMounts, desta vez iremos fazer com que o pod novo adicione uma data no arquivo index.html e depois o pod do nginx leia o arquivo:
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
    - mountPath: /usr/share/nginx/html
      name: empty
  - image: busybox
    name: input-date
    volumeMounts:
    - mountPath: /html
      name: empty
    command: ["/bin/sh", "-c"]
    args:
    - while true; do
        date >> /html/index.html;
        sleep 1;
      done
  dnsPolicy: ClusterFirst
  restartPolicy: Always
  volumes:
  - name: empty
    emptyDir: {}
```

Vamos executar o replace:
```shell
kubectl replace -f pod.yaml
```

Vamos fazer um curl localhost dentro do container do nginx:
```shell
kubectl exec -it nginx -c nginx -- curl localhost
```
OUTPUT
```shell
Tue Sep 17 22:35:38 UTC 2024
Tue Sep 17 22:35:39 UTC 2024
Tue Sep 17 22:35:40 UTC 2024
Tue Sep 17 22:35:41 UTC 2024
Tue Sep 17 22:35:42 UTC 2024
Tue Sep 17 22:35:43 UTC 2024
Tue Sep 17 22:35:44 UTC 2024
```