# PersistentVolumeClaim
Um `PersistentVolumeClaim` (PVC) e uma solicitacao de armazenamento por um usuario. E semelhante a um pod. Os pods consomem recursos de no e os PVC's consomem recursos do PV.

Vamos criar um `namespace` chamado `mynamespace`:
```shell
kubectl create ns mynamespace
namespace/mynamespace created
```

Uma diferenca entre e PV e o PVC e que o PV e um recurso global e um PVC precisa de uma namespace.

Vamos criar um arquivo chamado pvc.yaml com a seguinte estrutura:
```yaml
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: firstpvc
  namespace: mynamespace
spec:
  accessModes:
  - ReadWriteOnce
  resources:
    requests:
      storage: 1Gi
```

Vamos criar o nosso PVC:
```yaml
kubectl apply -f pvc.yaml 
persistentvolumeclaim/firstpvc created
```

Com o nosso PVC criado, vamos verificar as informacoes dele, lembrando que precisamos passar o parametro `-n <NAMESPACE>` pois ele se encontra em uma namespace.
```shell
kubectl get pvc -n mynamespace
```
OUTPUT
```shell
NAME       STATUS   VOLUME    CAPACITY   ACCESS MODES   STORAGECLASS   VOLUMEATTRIBUTESCLASS   AGE
firstpvc   Bound    firstpv   1Gi        RWO                           <unset>                 83s
```

Podemos realizar um `describe` e verificar maiores informacoes sobre o PVC:
```shell
kubectl describe pvc firstpvc -n mynamespace
```
OUTPUT
```shell
Name:          firstpvc
Namespace:     mynamespace
StorageClass:  
Status:        Bound
Volume:        firstpv
Labels:        <none>
Annotations:   pv.kubernetes.io/bind-completed: yes
               pv.kubernetes.io/bound-by-controller: yes
Finalizers:    [kubernetes.io/pvc-protection]
Capacity:      1Gi
Access Modes:  RWO
VolumeMode:    Filesystem
Used By:       <none>
Events:        <none>
```

Vamos criar um pod para chamar o nosso PVC:
```shell
kubectl run nginx --image nginx --dry-run=client -o yaml > pod-pvc.yaml
```

Vamos adicionar as seguintes informacoes para a utilizacao do nosso volume:
```yaml
...
  namespace: mynamespace
...
    volumeMounts:
    - name: usingpvc
      mountPath: /data
...
  volumes:
  - name: usingpvc
    persistentVolumeClaim:
      claimName: firstpvc
```



Arquivo final pod-pvc.yaml:
```yaml
apiVersion: v1
kind: Pod
metadata:
  labels:
    run: nginx
  name: nginx
  namespace: mynamespace
spec:
  containers:
  - image: nginx
    name: nginx
    resources: {}
    volumeMounts:
    - name: usingpvc
      mountPath: /data
  dnsPolicy: ClusterFirst
  restartPolicy: Always
  volumes:
  - name: usingpvc
    persistentVolumeClaim:
      claimName: firstpvc
```

Vamos criar o Pod:
```shell
kubectl apply -f pod-pvc.yaml
pod/nginx created
```

Podemos ver informacoes do PV, PVC e POD utilizando o comando `kubectl get pv,pvc,pod -n <NAMESPACE>`:
```shell
kubectl get pv,pvc,pod -n mynamespace
NAME                       CAPACITY   ACCESS MODES   RECLAIM POLICY   STATUS   CLAIM                  STORAGECLASS   VOLUMEATTRIBUTESCLASS   REASON   AGE
persistentvolume/firstpv   1Gi        RWO            Retain           Bound    mynamespace/firstpvc                  <unset>                          71m

NAME                             STATUS   VOLUME    CAPACITY   ACCESS MODES   STORAGECLASS   VOLUMEATTRIBUTESCLASS   AGE
persistentvolumeclaim/firstpvc   Bound    firstpv   1Gi        RWO                           <unset>                 16m

NAME        READY   STATUS    RESTARTS   AGE
pod/nginx   1/1     Running   0          83s
```

Vamos olhar como esta no Pod:
```shell
kubectl describe pod nginx -n mynamespace
```
OUTPUT
```shell
Name:             nginx
Namespace:        mynamespace
Priority:         0
Service Account:  default
Node:             node01/192.17.80.11
Start Time:       Sat, 14 Sep 2024 14:40:18 +0000
Labels:           run=nginx
Annotations:      <none>
Status:           Running
IP:               10.244.1.2
IPs:
  IP:  10.244.1.2
Containers:
  nginx:
    Container ID:   containerd://192007ee89b01ca76a571ce5e0b9d33f740b1154e09348ece35367925c6d7ce8
    Image:          nginx
    Image ID:       docker.io/library/nginx@sha256:04ba374043ccd2fc5c593885c0eacddebabd5ca375f9323666f28dfd5a9710e3
    Port:           <none>
    Host Port:      <none>
    State:          Running
      Started:      Sat, 14 Sep 2024 14:40:23 +0000
    Ready:          True
    Restart Count:  0
    Environment:    <none>
    Mounts:
      /data from usingpvc (rw)
      /var/run/secrets/kubernetes.io/serviceaccount from kube-api-access-2zj2z (ro)
Conditions:
  Type                        Status
  PodReadyToStartContainers   True 
  Initialized                 True 
  Ready                       True 
  ContainersReady             True 
  PodScheduled                True 
Volumes:
  usingpvc:
    Type:       PersistentVolumeClaim (a reference to a PersistentVolumeClaim in the same namespace)
    ClaimName:  firstpvc
    ReadOnly:   false
  kube-api-access-2zj2z:
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
  Type    Reason     Age    From               Message
  ----    ------     ----   ----               -------
  Normal  Scheduled  3m23s  default-scheduler  Successfully assigned mynamespace/nginx to node01
  Normal  Pulling    3m22s  kubelet            Pulling image "nginx"
  Normal  Pulled     3m18s  kubelet            Successfully pulled image "nginx" in 4.148s (4.148s including waiting). Image size: 71027698 bytes.
  Normal  Created    3m18s  kubelet            Created container nginx
  Normal  Started    3m18s  kubelet            Started container nginx
```

Repare que temos a configuracao `Mounts` e `Volumes`:
```shell
...
    Mounts:
      /data from usingpvc (rw)
      /var/run/secrets/kubernetes.io/serviceaccount from kube-api-access-2zj2z (ro)
...
Volumes:
  usingpvc:
    Type:       PersistentVolumeClaim (a reference to a PersistentVolumeClaim in the same namespace)
    ClaimName:  firstpvc
    ReadOnly:   false
...
```