# Command
O command e utilizado para sobescrever o entry point original.

| | Docker | Kubernetes |
|----------|:-------------:|------:|
| comando executado no contêiner | Entrypoint | command |
| argumentos passados para o contêiner | Cmd | args |
| Image Entrypoint | Image Cmd | Container command | Container args | Command run |
|----------|:-------------:|------:|------:|------:|
| [/echo] | [foo] | <not set> | <not set> | [echo foo] |
| [/echo] | [foo] | [/printf] | <not set> | [printf] |
| [/echo] | [foo] | <not set> | [bar] | [echo bar] |
| [/echo] | [foo] | [/printf] | [bar] | [printf bar] |

Uso
```shell
command: ["/bin/sh"]
args: ["-c", "while true; do echo hello; sleep 10;done"]
```

Manifesto
```yaml
apiVersion: v1
kind: Pod
metadata:
  name: command
spec:
  containers:
  - name: command
    image: nginx
    command: ["printf"]
    args: ["bar"]
  restartPolicy: OnFailure
```