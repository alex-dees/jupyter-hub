Notes for manually deploying JupyterHub to EKS from an Amazon Workspace.

## Install (Windows)

[kubectl](https://docs.aws.amazon.com/eks/latest/userguide/install-kubectl.html) 

- add c:\bin to PATH

[chocolatey](https://chocolatey.org/install)  

[eksctl](https://docs.aws.amazon.com/eks/latest/userguide/eksctl.html)

[helm](https://docs.aws.amazon.com/eks/latest/userguide/helm.html)  

## IAM

Create a **RedshiftGetClusterCredentials** policy to allow **test_user** to [GetClusterCredentials](https://docs.aws.amazon.com/redshift/latest/mgmt/generating-iam-credentials-role-permissions.html).  The Redshift cluster and user are created later.


```
{
    "Version": "2012-10-17",
    "Statement": [
        {
            "Effect": "Allow",
            "Action": "redshift:GetClusterCredentials",
            "Resource": "arn:aws:redshift:<REGION>:<ACCOUNT>:dbuser:jhub-cluster/test_user"
        }
    ]
}
```

## eksctl

Cluster [dry run](https://eksctl.io/usage/dry-run/)  

```
eksctl create cluster \
--name jhub \
--region us-east-1 \
--dry-run > cluster.yml
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
Create cluster [creates a dedicated VPC](https://eksctl.io/usage/vpc-networking/)

`eksctl create cluster -f cluster.yml`

## helm

Create config.yml

```
singleuser:
  serviceAccountName: redshift
hub:
  config:
    Authenticator:
      admin_users:
        - <admin>
      allowed_users:
        - <user 1>
        - <user 2>
    DummyAuthenticator:
      password: <password>
    JupyterHub:
      authenticator_class: dummy
```

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
To update the config: `helm upgrade -f config.yml -n jhub jhub jupyterhub/jupyterhub`

## DNS 
Create R53 domain/zone e.g. deeslabs.com  
Add CNAME record (jhub.deeslabs.com) for [proxy-public](https://zero-to-jupyterhub.readthedocs.io/en/latest/administrator/security.html#set-up-your-domain) ALB

## TLS

Create ACM cert for domain (jhub.deeslabs.com)  
[Update the Helm config.yml](https://zero-to-jupyterhub.readthedocs.io/en/latest/administrator/security.html#off-loading-ssl-to-a-load-balancer)  
```
proxy:
  https:
    enabled: true
    type: offload
  service:
    annotations:
      service.beta.kubernetes.io/aws-load-balancer-ssl-ports: "https"
      service.beta.kubernetes.io/aws-load-balancer-backend-protocol: "tcp"
      service.beta.kubernetes.io/aws-load-balancer-connection-idle-timeout: "3600"
      service.beta.kubernetes.io/aws-load-balancer-ssl-cert: "<certificate arn>"
```
Run helm upgrade

## Redshift
Create cluster subnet group (jhub-cluster-subnet-group)
- VPC:  The EKS VPC
- Add all the subnets for this VPC

Create security group (eks-vpc-sg)
- Inbound rule: All traffic, 192.168.0.0/16

Create cluster (jhub-cluster)
- Node:  1 dc2.large
- Admin:  jhubadmin
- Role: Manage IAM roles->Create IAM role (any s3)
- Network
  - VPC: The EKS VPC
  - Security group: eks-vpc-sg
  - Cluster subnet group: jhub-cluster-subnet-group

Load [2010 Census Surname](https://www.census.gov/topics/population/genealogy/data/2010_surnames.html) data

Create a test user
```
create user test_user password '<PASSWORD>';

grant select on dev.public.names_2010_census to test_user;
```
