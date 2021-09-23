---
title: "IAM & RBAC in EKS"
outputs: ["Reveal"]
reveal_hugo:
  custom_theme: "reveal-hugo/themes/robot-lung.css"
  margin: 0.0
---

# IAM & RBAC in EKS

---

![](/eks_iam_rbac_overview.drawio.svg)

  1. Kubernetes Role-Based Access Control
  1. `aws-auth` ConfigMap
  3. IAM roles for Service Accounts

---

## Kubernetes RBAC

---

### RBAC

  * One authorisation mode in Kubernetes, enabled by default in EKS
  * Controls access _within_ the Kubernetes cluster
  * Deny by default
  * Can only add "allows"

---

### Resources

  * Roles
  * RoleBindings
  * ClusterRoles & ClusterRoleBindings
  * ServiceAccounts
  * Users
  * Groups

---

![](/eks_rbac.drawio.svg)

---

### Roles

```yaml
apiVersion: rbac.authorization.k8s.io/v1
kind: Role
metadata:
  name: kube-prometheus-stack-grafana
  namespace: monitoring
rules:
- apiGroups:
  - extensions
  resourceNames:
  - kube-prometheus-stack-grafana
  resources:
  - podsecuritypolicies
  verbs:
  - use
```

  * Defines access for resources and actions
  * `apiGroups` is the API path
  * `resources` is the resource type (`pods`, `deployment`)
  * `resourceNames` can limit to individual resource
  * Equivalent of an AWS IAM Policy

---

### RoleBindings

```yaml
apiVersion: rbac.authorization.k8s.io/v1
kind: RoleBinding
metadata:
  name: kube-prometheus-stack-grafana
  namespace: monitoring
roleRef:
  apiGroup: rbac.authorization.k8s.io
  kind: Role
  name: kube-prometheus-stack-grafana
subjects:
- kind: ServiceAccount
  name: kube-prometheus-stack-grafana
  namespace: monitoring
```

  * Attaches a `Role` to a "subject"
  * Equiavlent of a `aws.iam.RolePolicyAttachment` resource
  * Subjects are Roles, users, groups and ServiceAccounts

---

### ClusterRoles and ClusterRoleBindings

  * Apply without consideration to namespaces
  * Otherwise same as their non-cluster equiavlents
  * Necessary for non-namespaced resources

---

### ServiceAccounts

```yaml
apiVersion: v1
automountServiceAccountToken: true
kind: ServiceAccount
metadata:
  name: metrics-server
  namespace: monitoring
secrets:
- name: metrics-server-token-q8f6j
```

  * Serviceaccount for authenticating to API server
  * Token is mounted in to pod
  * Can be a subject for a RoleBinding

---

![](/eks_rbac.drawio.svg)

---

### Users and Groups

```
$ kubectl get users
error: the server doesn't have a resource type "users"
$ kubectl get groups
error: the server doesn't have a resource type "groups"
```

---

### `aws-auth` ConfigMap

---

```yaml
apiVersion: v1
kind: ConfigMap
metadata:
  name: aws-auth
  namespace: kube-system
data:
  mapRoles: |
    - rolearn: arn:aws:iam::123456789:role/platform-admin
      username: platform-admin
      groups:
        - system:masters
    - rolearn: arn:aws:iam::123456789:role/engineer
      username: engineer
      groups:
        - engineer
    - rolearn: arn:aws:iam::123456789:role/engineer-team-lead
      username: engineer-team-lead
      groups:
        - engineer-team-lead
    - rolearn: arn:aws:iam::123456789:role/eks-my-cluster-nodes-role
      username: system:node:{{EC2PrivateDNSName}}
      groups:
        - system:bootstrappers
        - system:nodes
```

  * Well-defined ConfigMap read by AWS
  * Maps AWS Roles to users and groups
  * Can break the cluster if you stuff it up :)

---

![](/eks_authmap.drawio.svg)

---

`$ aws eks update-kubeconfig --cluster-name my-cluster --region ap-southeast-2 --profile my-account`

---

`~/.kube/config`:
```yaml
kind: Config
apiVersion: v1
clusters:
- cluster:
    certificate-authority-data: LS0tLS1CRUdJTiBDRVJUSUZJQ0FURS0tLS0tCk1JSUM1ekNDQWMrZ0F3SUJBZ0lCQURBTkJna3Foa2lHOXcwQkFRc0ZBREFWTVJNd0VRWURWUVFERXdwcmRXSmwKY201bGRHVnpNQjRYRFRJeE1EZ3lOREV5TURFeU5Wb1hEVE14TURneU1qRXlNREV5TlZvd0ZURVRNQkVHQTFVRQpBeE1LYTNWaVpYSnVaWFJsY3pDQ0FTSXdEUVlKS29aSWh2Y05BUUVCQlFBRGdnRVBBRENDQVFvQ2dnRUJBTGQzCm9Iak0vR21HdlFGZ3dDc003Sjl1VERLTThteWwyV284Mm9mSFBUL2NYZnhuT0RURUFWWVNQb3I1UHpQcWtiYVEKQ0h2bjlyUzhMN2NPT0w5QkVSTmdoNHFUb3Rkd2k5dE5ITCs3azJHRXFKUzl2c2Q1V3JMK1J5WDVVVWUzQWxJdAp3YlZnbUpTVHZ1ZEhBNy9WRGVkQlp5Y0JvN1NBQmlHenRvWDBReW5SVTZVZVZYbldvVkNjSHhiRSsrS3lZS29QCjZEK1RBb3hUQ1JKM3d0OHRzVVNPWDhqSjZOaHBDZC9Zb21hblpPTXhlRTBoS05vVkxwWiszaktUMDE4RDlLVUkKaUkwWXRRWSt0UmNYZk9qTFNyL2YzcTk2N2tFM0ZHck9JakpoNkgrdk9sQ2pwTG1PUW1PendiNXNPbGQrOGptegpwTFhHZVFqV1p6eUowbG5zS3prQ0F3RUFBYU5DTUVBd0RnWURWUjBQQVFIL0JBUURBZ0trTUE4R0ExVWRFd0VCCi93UUZNQU1CQWY4d0hRWURWUjBPQkJZRUZFMDU1RkxZdFp3V1JSQ09aSTZSd2hDOFlkVGRNQTBHQ1NxR1NJYjMKRFFFQkN3VUFBNElCQVFBNG1RYVlTUnIydFpNaGl3OXg1amxmaDBtTHpXc25XMmVqbGozVzB4cmhLOXRnMWN2UQpXbm5YS2FNd3FDTXEwYlB0NThHV055K2pIRWJFTnVJb0VZTkNmWEtacUc1a1kvdWNEU0tLZVV0OXRlTnpsQ0Y3CmdkQ0NIVklkOEhoSkZjbmF3aDRpSjhaSUMyMzFCSTh2VExlbG5haFZLekFrY3RmSGlmV1F4alZrWXBoYldDY0oKbEc5aDdGRXMwS1J3SXJGSFNtN1o3MGdLMHhhTWJhdHNjYnd0ZFNPTE1rNmx5dTZvVm9QSzBDUzYyK1pvb0lkUQpva2pQZ1Fyck5EcTduUXoxY0doSlM2TStibDBlV1FaNXM2REFhTkZwMC85TXZpa2EwcEtKRjZXMlFrL3kyTTZOCkpoNXNXZjhTU2VheFI3LzJCRnR1VVpHZEFCYmVEQlpkaDQraAotLS0tLUVORCBDRVJUSUZJQ0FURS0tLS0tCg==
    server: https://7DB07D1619AA33CB4B3168D0B703D458.gr7.ap-southeast-2.eks.amazonaws.com
  name: arn:aws:eks:ap-southeast-2:123456789:cluster/my-cluster
- context:
    cluster: arn:aws:eks:ap-southeast-2:123456789:cluster/my-cluster
    user: arn:aws:eks:ap-southeast-2:123456789:cluster/my-cluster
  name: arn:aws:eks:ap-southeast-2:123456789:cluster/my-cluster
- name: arn:aws:eks:ap-southeast-2:123456789:cluster/my-cluster
  user:
    exec:
      apiVersion: client.authentication.k8s.io/v1alpha1
      args:
      - --region
      - ap-southeast-2
      - eks
      - get-token
      - --cluster-name
      - my-cluster
      command: aws
      env:
      - name: AWS_PROFILE
        value: my-account
```

---

`$ aws eks get-token --cluster-name my-cluster --region ap-southeast-2 --profile my-account`
```json
{
  "kind": "ExecCredential",
  "apiVersion": "client.authentication.k8s.io/v1alpha1",
  "spec": {},
  "status": {
    "expirationTimestamp": "2021-09-01T08:03:29Z",
    "token": "k8s-aws-v1.aHR0cHM6Ly9zdHMuYW1hem9uYXdzLmNvbS8_QWN0aW9uPUdldENhbGxlcasdfasdfasdflZlcnNpb249MjAxMS0wNi0xNSZYLUFtei1BbGdvcml0aG09QVdTNC1ITUFDLVNIQTI1NiZYLUFtei1DcmVkZW50aWFsPUFTSUFSWEZWMzRZV1FMWFM3R0gzJTJGMjAyMTA5MDElMkZ1cy1lYXN0LTElMkZzdHMlMkZhd3M0X3JlcXVlc3QmWC1BbXotRGF0ZT0yMDIxMDkwMVQwNzQ5MjlaJlgtQW16LUV4cGlyZXM9NjAmWC1BbXotU2lnbmVkSGVhZGVycz1ob3N0JTNCeC1rOHMtYXdzLWlkJlgtQW16LVNlY3VyaXR5LVRva2VuPUZ3b0daWEl2WVhkekVMdiUyRiUyRiUyRiUyRiUyRiUyRiUyRiUyRiUyRiUyRndFYURDMjA4aCUyRiUyRmJoVjhGZkplaUNLYUFqT0hKODJtQm51RE1QQVBEc05JN0o2SWYlMkZ0R0djYkwwYnJIS2pkRnRySEVHVW5FRlVKSXBXaHQxaUNyZ0RybEk5bjQzbHRjZjRJV1JNOG9WJTJCem1HdE5DYWFsNzRiYVdBMjRPVWF4NjZrdUR1YzFNMEdYd1ElMkJQT1J5WDFQRFZYQ1dWJTJGNTF3aDVkZlkxcWtvMEYlMkJxS3FtMURkTDElMkJSeSUyRkVPSm9zSldLa08wMzdPQVI2Vml3dmRZU0o1eFRJdk1ZZ0czQlNKampCS0Z4ODNYJTJCJTJCVVp4c2lqV2klMkZSVGhNWlVSRXpkMmI4SkhwdyUyQm5paXMlMkZQRjhWZ1VFbzU2RXZFYWFMbnMwZ3QlMkZkNlA4Z243czZqV1JGaFcxJTJCeiUyRkZtOUE0QkhneGF1bTQ4Nlhha202Wlk3TFFCVU5MOEJIVU0lMkJUV25FdERBZXJucDJuWDRJTTR3cU9vbG9OVXk4ajRSNWxYJTJCTU5TQU5hS1J6amg0dXhXdlRIeXViZm10elNpcnFMdUpCaklyVHNESE5ESWUlMkJsSFo3bUlLdFRuZ2hnRVgyZmhjalZreHBFSHFPay1yRlZhJTJCSmpqc2dpdDFmSTNYMjBOZyUzRCUzRCZYLUFasdfasdfasdf1cmU5MDcyZDVjZDY2NWI4NmU4YmQzYzNjZGY1YmRjY2ihYTYxOWU2ZTQxM2E3YjkzMWIyNGE0ZGIwZmE3NTRiOGEzNg"
  }
}
```

---

## IAM Roles for ServiceAccounts

  * Functionally the same as IAM Roles for ECS Tasks
  * Implementation-wise completely different
  * No metadata API
  * Prevents overly permissive roles for EC2 instances

---

### Identity Provider

![](/eks_oidc.png)

---

### Assume role policy document

```json
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Effect": "Allow",
      "Principal": {
        "Federated": "arn:aws:iam::123456789:oidc-provider/oidc.eks.ap-southeast-2.amazonaws.com/id/7DBasdf619AA33C5433168D0B703D458"
      },
      "Action": "sts:AssumeRoleWithWebIdentity",
      "Condition": {
        "StringEquals": {
          "oidc.eks.ap-southeast-2.amazonaws.com/id/7DB07D1619AA33CB4B3168D0B703D458:sub": "system:serviceaccount:monitoring:kube-prometheus-stack-prometheus"
        }
      }
    }
  ]
}
```

  * Condition prevents any ServiceAccount in your cluster from using any IAM Role

---

### Annotation

```yaml
apiVersion: v1
kind: ServiceAccount
metadata:
  annotations:
    eks.amazonaws.com/role-arn: arn:aws:iam::123456789:role/service-role/eks-my-cluster-prometheus
  name: kube-prometheus-stack-prometheus
  namespace: monitoring
secrets:
- name: kube-prometheus-stack-prometheus-token-trbh6
```

---

![](/eks_irsa.png)
https://medium.com/getamis/aws-irsa-for-self-hosted-kubernetes-e045564494af

---

https://github.com/aws/amazon-eks-pod-identity-webhook#readme
```yaml
apiVersion: v1
kind: Pod
metadata:
  name: my-pod
  namespace: default
  annotations:
    # optional: A comma-separated list of initContainers and container names
    #   to skip adding volumes and environment variables
    eks.amazonaws.com/skip-containers: "init-first,sidecar"
    # optional: Defaults to 86400, or value specified in ServiceAccount
    #   annotation as shown in previous step, for expirationSeconds if not set
    eks.amazonaws.com/token-expiration: "86400"
spec:
  serviceAccountName: my-serviceaccount
  initContainers:
  - name: init-first
    image: container-image:version
  containers:
  - name: sidecar
    image: container-image:version
  - name: container-name
    image: container-image:version
### Everything below is added by the webhook ###
    env:
    - name: AWS_DEFAULT_REGION
      value: us-west-2
    - name: AWS_REGION
      value: us-west-2
    - name: AWS_ROLE_ARN
      value: "arn:aws:iam::111122223333:role/s3-reader"
    - name: AWS_WEB_IDENTITY_TOKEN_FILE
      value: "/var/run/secrets/eks.amazonaws.com/serviceaccount/token"
    - name: AWS_STS_REGIONAL_ENDPOINTS
      value: "regional"
    volumeMounts:
    - mountPath: "/var/run/secrets/eks.amazonaws.com/serviceaccount/"
      name: aws-token
  volumes:
  - name: aws-token
    projected:
      sources:
      - serviceAccountToken:
          audience: "sts.amazonaws.com"
          expirationSeconds: 86400
          path: token
```

---

## The End

![](/eks_iam_rbac_overview.drawio.svg)

  1. Kubernetes Role-Based Access Control
  1. `aws-auth` ConfigMap
  3. IAM roles for Service Accounts
