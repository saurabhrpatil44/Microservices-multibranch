# EKS Setup Guide

## 1. IAM Setup

### Create IAM User

1. Go to **IAM → Users → Create User**
2. User name:

   ```text
   eks-user
   ```
3. Provide console access.
4. Select **IAM user**.
5. Set a password.
6. Set permissions → **Attach policies directly**.

Attach the following policies:

* `EC2FullAccess`
* `EKS_NCI_POLICY`
* `EKSClusterPolicy`
* `EKSWorkerNodePolicy`
* `AWSCloudFormationFullAccess`
* `eksfullaccess` — Custom policy

### Custom `eksfullaccess` Policy

```json
{
    "Version": "2012-10-17",
    "Statement": [
        {
            "Sid": "VisualEditor0",
            "Effect": "Allow",
            "Action": "eks:*",
            "Resource": "*"
        }
    ]
}
```

### Create Access Key

Create an **Access Key** for the `eks-user`.

---

## 2. Create EC2 Instance

Create an EC2 instance with:

* **CPU:** 2 vCPU
* **Memory:** 8 GB

Install the following tools:

* Jenkins
* Docker
* AWS CLI
* eksctl
* kubectl

---

## 3. Configure AWS

Configure AWS CLI using:

```bash
aws configure
```

Provide the required:

* AWS Access Key ID
* AWS Secret Access Key
* Default region
* Output format

---

## 4. Create EKS Cluster

### Create EKS Control Plane

```bash
eksctl create cluster --name=saurabh-eks \
                      --region=ap-south-1 \
                      --zones=ap-south-1a,ap-south-1b \
                      --without-nodegroup
```

### Associate IAM OIDC Provider

```bash
eksctl utils associate-iam-oidc-provider \
    --region ap-south-1 \
    --cluster saurabh-eks \
    --approve
```

### Create Managed Node Group

```bash
eksctl create nodegroup --cluster=saurabh-eks \
                       --region=ap-south-1 \
                       --name=node2 \
                       --node-type=c7i-flex.large \
                       --nodes=3 \
                       --nodes-min=2 \
                       --nodes-max=4 \
                       --node-volume-size=20 \
                       --ssh-access \
                       --ssh-public-key="aws-keypair" \
                       --managed \
                       --asg-access \
                       --external-dns-access \
                       --full-ecr-access \
                       --appmesh-access \
                       --alb-ingress-access
```

---

## 5. Create Service Account

Create the Service Account YAML file:

```bash
vi serviceaccount.yaml
```

Add the following configuration:

```yaml
apiVersion: v1
kind: ServiceAccount
metadata:
  name: jenkins
  namespace: webapps
```

Apply the configuration:

```bash
kubectl apply -f serviceaccount.yaml
```

---

## 6. Create Role

Create a Role YAML file:

```yaml
apiVersion: rbac.authorization.k8s.io/v1
kind: Role
metadata:
  name: app-role
  namespace: webapps
rules:
  - apiGroups:
        - ""
        - apps
        - autoscaling
        - batch
        - extensions
        - policy
        - rbac.authorization.k8s.io
    resources:
      - pods
      - componentstatuses
      - configmaps
      - daemonsets
      - deployments
      - events
      - endpoints
      - horizontalpodautoscalers
      - ingress
      - jobs
      - limitranges
      - namespaces
      - nodes
      - pods
      - persistentvolumes
      - persistentvolumeclaims
      - resourcequotas
      - replicasets
      - replicationcontrollers
      - serviceaccounts
      - services
    verbs:
      - get
      - list
      - watch
      - create
      - update
      - patch
      - delete
```

Apply the Role:

```bash
kubectl apply -f role.yaml
```

---

## 7. Bind the Role to the Service Account

Create a RoleBinding YAML file:

```yaml
apiVersion: rbac.authorization.k8s.io/v1
kind: RoleBinding
metadata:
  name: app-rolebinding
  namespace: webapps
roleRef:
  apiGroup: rbac.authorization.k8s.io
  kind: Role
  name: app-role
subjects:
  - namespace: webapps
    kind: ServiceAccount
    name: jenkins
```

Apply the RoleBinding:

```bash
kubectl apply -f rolebinding.yaml
```

All YAML files have now been executed/applied.

---

## 8. Delete the EKS Cluster

To delete the EKS cluster:

```bash
eksctl delete cluster --name saurabh-eks --region ap-south-1
```
