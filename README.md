Notes for deploying JupyterHub to EKS using eksctl.

## Install (Windows)
---

[kubectl](https://docs.aws.amazon.com/eks/latest/userguide/install-kubectl.html) 

- add c:\bin to PATH

[chocolatey](https://chocolatey.org/install)  

[eksctl](https://docs.aws.amazon.com/eks/latest/userguide/eksctl.html)

[helm](https://docs.aws.amazon.com/eks/latest/userguide/helm.html)  


## eksctl
---
```
eksctl create cluster \
--name jhub \
--region us-east-1 \
--dry-run > cluster.yml
```

Add [service account](https://eksctl.io/usage/iamserviceaccounts/#usage-with-config-files)

```
  serviceAccounts:
  - metadata:
      name: redshift
      namespace: jhub
      labels: {aws-usage: "application"}
    attachPolicyARNs:
    - "arn:aws:iam::aws:policy/AmazonRedshiftDataFullAccess"
```

`eksctl create cluster -f cluster.yml`

## helm
---
[Install JupyterHub](https://zero-to-jupyterhub.readthedocs.io/en/latest/jupyterhub/installation.html)
```
helm repo add jupyterhub https://jupyterhub.github.io/helm-chart/
helm repo update

helm upgrade --cleanup-on-fail \
  --install jhub jupyterhub/jupyterhub \
  --namespace jhub \
  --create-namespace \
  --version=1.2.0 \
  --values config.yml
```
**Update config**  
`helm upgrade -f config.yml -n jhub jhub jupyterhub/jupyterhub`