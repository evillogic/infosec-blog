---
title: Kubernetes Debugging
date: 2023-12-30
draft: false
---

## Print kubectl logs from deployment or scaled-job
```bash
kubectl logs -l scaledjob.keda.sh/name=httpx-scaledjob --follow --max-log-requests 500
```

## Creating an Ubuntu Pod
```bash
cat <<EOF | kubectl apply -f -
apiVersion: v1
kind: Pod
metadata:
  name: ubuntu
  labels:
    app: ubuntu
spec:
  containers:
  - image: ubuntu
    command:
      - "sleep"
      - "604800"
    imagePullPolicy: IfNotPresent
    name: ubuntu
  restartPolicy: Always
EOF
```

`kubectl exec -it ubuntu -- /bin/bash`

# Privileged with service account and host networking

```bash
cat <<EOF | kubectl apply -f -
apiVersion: v1
kind: Pod
metadata:
  name: ubuntu
  labels:
    app: ubuntu
spec:
  hostNetwork: true
  serviceAccountName: httpx-service-account
  containers:
  - image: ubuntu
    command:
      - "sleep"
      - "604800"
    imagePullPolicy: IfNotPresent
    name: ubuntu
    securityContext:  
      privileged: true
  restartPolicy: Always
EOF
```
maybe only try this at home kids
# With volumes mounted

```bash
cat <<EOF | kubectl apply -f -
apiVersion: v1
kind: Pod
metadata:
  name: ubuntu
  labels:
    app: ubuntu
spec:
  hostNetwork: true
  serviceAccountName: httpx-service-account
  containers:
  - image: ubuntu
    command:
      - "sleep"
      - "604800"
    imagePullPolicy: IfNotPresent
    name: ubuntu
    securityContext:  
      privileged: true
    volumeMounts:
      - name: screenshots
        mountPath: /screenshots
      - name: assets
        mountPath: /assets
  volumes:
    - name: screenshots
      hostPath:
        path: /shared/screenshots
    - name: assets
      hostPath:
        path: /shared/assets
  restartPolicy: Always
EOF
```

## Pod with a Secret

```yaml
cat <<EOF | kubectl apply -f -
apiVersion: v1
kind: Secret
metadata:
  name: admin-creds
data:
  username: bXktYXBw
  password: Mzk1MjgkdmRnN0pi
EOF
```

```bash
cat <<EOF | kubectl apply -f -
apiVersion: v1
kind: Pod
metadata:
  name: important-app
  labels:
    app: important-app
spec:
  containers:
  - image: ubuntu
    command:
      - "sleep"
      - "6048000"
    imagePullPolicy: IfNotPresent
    name: ubuntu
    env:
    - name: SECRET_PASSWORD
      valueFrom:
        secretKeyRef:
          name: admin-creds
          key: password
  restartPolicy: Always
EOF
```

## IMDSv2

```
TOKEN=`curl -X PUT "http://169.254.169.254/latest/api/token" -H "X-aws-ec2-metadata-token-ttl-seconds: 21600"` 
curl http://169.254.169.254/latest/meta-data/profile -H "X-aws-ec2-metadata-token: $TOKEN"
```

USER DATA
```
TOKEN=`curl -X PUT "http://169.254.169.254/latest/api/token" -H "X-aws-ec2-metadata-token-ttl-seconds: 21600"` 
curl http://169.254.169.254/latest/user-data/ -H "X-aws-ec2-metadata-token: $TOKEN"
```

## Creating a CSR for a node

```bash
openssl req -nodes -newkey rsa:2048 -keyout k8shack.key -out k8shack.csr -subj "/O=system:nodes/CN=system:node:<node>"
```

```bash
cat <<EOF | ./kubectl --kubeconfig k create -f -
apiVersion: certificates.k8s.io/v1
kind: CertificateSigningRequest
metadata:
  name: node-csr-$(date +%s)
spec:
  signerName: kubernetes.io/kube-apiserver-client-kubelet
  groups:
  - system:nodes
  request: $(cat k8shack.csr | base64 | tr -d '\n')
  usages:
  - digital signature
  - key encipherment
  - client auth
EOF
```

```bash
./kubectl --kubeconfig k get csr node-csr-1543524743 -o jsonpath='{.status.certificate}' | base64 -d > node2.crt
```

Note: I don't think EKS supports this, have never gotten this working right