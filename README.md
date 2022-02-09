Notes for deploying JupyterHub to EKS using eksctl.

## Install (Windows)

[kubectl](https://docs.aws.amazon.com/eks/latest/userguide/install-kubectl.html) 

- add c:\bin to PATH

[chocolatey](https://chocolatey.org/install)  

[eksctl](https://docs.aws.amazon.com/eks/latest/userguide/eksctl.html)

[helm](https://docs.aws.amazon.com/eks/latest/userguide/helm.html)  

## redshift

```
create user test_user password '<PASSWORD>';

grant select on dev.public.names_2010_census to test_user;
```

## eksctl

```
eksctl create cluster \
--name jhub \
--region us-east-1 \
--dry-run > cluster.yml
```

Create a policy to allow [GetClusterCredentials](https://docs.aws.amazon.com/redshift/latest/mgmt/generating-iam-credentials-role-permissions.html)

RedshiftGetClusterCredentials
```
{
    "Version": "2012-10-17",
    "Statement": [
        {
            "Effect": "Allow",
            "Action": "redshift:GetClusterCredentials",
            "Resource": "arn:aws:redshift:<REGION>:<ACCOUNT>:dbuser:<CLUSTER>/test_user"
        }
    ]
}
```

Add [service account](https://eksctl.io/usage/iamserviceaccounts/#usage-with-config-files)

```
iam:
  vpcResourceControllerPolicy: true
  withOIDC: true
  serviceAccounts:
  - metadata:
      name: redshift
      namespace: jhub
      labels: {aws-usage: "application"}
    attachPolicyARNs:
    - "arn:aws:iam::aws:policy/AmazonRedshiftDataFullAccess"
    - "arn:aws:iam::<ACCOUNT ID>:policy/RedshiftGetClusterCredentials"
```

`eksctl create cluster -f cluster.yml`

## dns & tls

## helm

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
