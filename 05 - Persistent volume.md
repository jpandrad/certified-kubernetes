
# Persistent Volume
Um `Persistent Volume` (PV) e uma parte do armazenamento no cluster que foi provisionado por um administrador ou provisionamento dinamicamente Storage Classes. Os PVs tem um ciclo de vida independente de qualquer pod associado a ele.

Alguns tipos de PVs sao implementados via plugins. O Kubernetes suporta atualemnte varios plugins.

## Criando um PV

Vamos criar um arquivo chamado `pv.yaml` com a seguinte estrutura:
```yaml
apiVersion: v1
kind: PersistentVolume
metadata:
  name: firstpv
spec:
  accessModes:
  - ReadWriteOnce
  capacity:
    storage: 1Gi
  hostPath:
    path: /root/pvtest
```

Para aplicar o PV, execute o comando `kubectl apply -f`:
```shell
kubectl apply -f pv.yaml 
persistentvolume/firstpv created
```

Para verificar o PV criado:
```shell
kubectl get pv
NAME      CAPACITY   ACCESS MODES   RECLAIM POLICY   STATUS      CLAIM   STORAGECLASS   VOLUMEATTRIBUTESCLASS   REASON   AGE
firstpv   1Gi        RWO            Retain           Available                          <unset>                          27s
```

O PV e um recurso global, nao sendo necessario informar o namespace.

Para obter maiores informacoes sobre o pv, utilize o comando `kubectl describe pv <PV_NAME>`:
```shell
kubectl describe pv firstpv
Name:            firstpv
Labels:          <none>
Annotations:     <none>
Finalizers:      [kubernetes.io/pv-protection]
StorageClass:    
Status:          Available
Claim:           
Reclaim Policy:  Retain
Access Modes:    RWO
VolumeMode:      Filesystem
Capacity:        1Gi
Node Affinity:   <none>
Message:         
Source:
    Type:          HostPath (bare host directory volume)
    Path:          /root/pvtest
    HostPathType:  
Events:            <none>
```

## Pontos de atencao para a prova

### Capacity
```yaml
capacity:
  storage: 1Gi
```


### VolumeMode
```yaml
volumeMode: [Filesystem | Block]
```


### accessMode:
```yaml
AccessModes:
- [ReadWriteOnce | ReadOnlyMany | ReadWriteMany]
```
* ReadWriteOnce: O volume pode ser montado com leitura e escrita (read-write) por um unico node.

* ReadOnlyMany: O volume pode ser monstado como somente leitura (read-only) por varios node.

* ReadWriteMany: O volume pode ser montado como leitura e escrita (read-write) por varios nodes.


### Reclaim Policy
Essa opcao tem como objetivo a acao que o cluster ira fazer apos que o PVC for excluido
```yaml
persistentVolumeReclaimPolicy: [Retain | Recycle | Delete]
```
* Reatin: Precisa de uma acao manual

* Recycle: Exclui os arquivos igual utilizado com o comando `rm -Rf`

* Delete: Utilizado em Cloud Providers como AWS EBS, GCE PD, Azure Disk ou OpenStack Cinder no qual o volume sera excluido


### Class
Determina qual sera o storageClass que o PV ira utilizar:
```yaml
storageClassName: "Slow"
```


### hostPath
Um Volume do tipo `hostPath` monta um arquivo ou diretorio do filesystem do no do host em que esta rodando o pod.
```yaml
hostPath:
  path:
  type: [FileOrCreate | DirectoryOrCreate]
```

* DirectoryOrCreate: Se nao existir o caminho fornecido, um diretorio vazio sera criado com a permissao 0755, tendo o mesmo grupo e prioridade do Kublet.

* FileOrCreate: Se nao existir o caminho fornecido, um arquivo vazio sera criado com a permissao definida como 644, tendo o mesmo grupo e prioridade do Kublet.