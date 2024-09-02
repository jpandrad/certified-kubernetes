# Introducao

## Pre requisito
### Instalacao do Docker
```shell
curl -fsSL https://get.docker.com | bash
```


### Confgiuracao do kind
```shell
# Kind: https://kind.sigs.k8s.io/docs/user/quick-start/#installing-from-release-binaries
curl -Lo ./kind https://kind.sigs.k8s.io/dl/v0.13.0/kind-linux-amd64
chmod +x ./kind
mv kind /usr/local/bin
kind create cluster
```


### Instalacao do Kubectl
https://kubernetes.io/docs/tasks/tools/install-kubectl-linux/
```shell
curl -fsSLo /usr/share/keyrings/kubernetes-archive-keyring.gpg https://packages.cloud.google.com/apt/doc/apt-key.gpg
echo "deb [signed-by=/usr/share/keyrings/kubernetes-archive-keyring.gpg] https://apt.kubernetes.io/ kubernetes-xenial main" | sudo tee /etc/apt/sources.list.d/kubernetes.list
apt-get update
apt-get install -y kubectl
```
OU
```shell
curl -LO "https://dl.k8s.io/$(curl -L -s https://dl.k8s.io/release/stable.txt)/bin/linux/amd64/kubectl.sha256"
echo "$(cat kubectl.sha256)  kubectl" | sha256sum --check
sudo install -o root -g root -m 0755 kubectl /usr/local/bin/kubectl
```


### Autocomplete
```shell
source <(kubectl completion bash)
echo "source <(kubectl completion bash)" >> ~/.bashrc
```


### Alias
```shell
alias k=kubectl
```