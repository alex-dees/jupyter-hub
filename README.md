Notes for deploying JupyterHub to EKS from an Amazon Workspace using eksctl.

### Install (Windows)
---  

**kubectl** - [docs](https://docs.aws.amazon.com/eks/latest/userguide/install-kubectl.html)  

```
mkdir /c/bin && cd /c/bin

curl -o kubectl.exe https://amazon-eks.s3.us-west-2.amazonaws.com/1.21.2/2021-07-05/bin/windows/amd64/kubectl.exe

add c:\bin to PATH
```

**chocolatey** - [docs](https://chocolatey.org/install)  

```
Set-ExecutionPolicy Bypass -Scope Process -Force; [System.Net.ServicePointManager]::SecurityProtocol = [System.Net.ServicePointManager]::SecurityProtocol -bor 3072; iex ((New-Object System.Net.WebClient).DownloadString('https://community.chocolatey.org/install.ps1'))   
```

**eksctl** - [docs](https://docs.aws.amazon.com/eks/latest/userguide/eksctl.html)

```
choco install -y eksctl
```  

**helm** - [docs](https://docs.aws.amazon.com/eks/latest/userguide/helm.html)  
```
choco install -y kubernetes-helm
```