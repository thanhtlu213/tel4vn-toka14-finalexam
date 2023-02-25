![](/images/pic3.png)

## 1. Create deployment:

```
cat <<'EOF' > tel4vn-bai3-webapp-deployment.yaml
---
apiVersion: v1
kind: Namespace
metadata:
  name: cau3

---
apiVersion: v1
kind: ServiceAccount
metadata: 
  name: webapp-sa
  namespace: cau3  
automountServiceAccountToken: false

---
apiVersion: apps/v1
kind: Deployment
metadata:
  name: webapp
  namespace: cau3
  labels:
    app: webapp
spec:
  replicas: 2
  selector:
    matchLabels:
      app: webapp
  template:
    metadata:
      labels:
        app: webapp
    spec:
      serviceAccountName: webapp-sa
      containers:
      - name: nginx
        image: nginx:latest
        ports:
        - containerPort: 80

---
apiVersion: v1
kind: Service
metadata:
  name: nginx-svc
  namespace: cau3
spec:
  selector:
    app: webapp
  type: ClusterIP
  ports:
  - port: 80
    targetPort: 80
    protocol: TCP

---
apiVersion: rbac.authorization.k8s.io/v1
kind: Role
metadata:
  name: pod-reader
  namespace: cau3
rules:
- apiGroups: [""]
  resources: ["deployments"]
  verbs: ["get", "list", "watch"]

---
apiVersion: rbac.authorization.k8s.io/v1
kind: RoleBinding
metadata:
  name: read-pods
  namespace: cau3
subjects:
  - kind: User
    name: k8s-readonly
    apiGroup: rbac.authorization.k8s.io
roleRef:
  kind: Role
  name: pod-reader
  apiGroup: rbac.authorization.k8s.io

---
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: default-deny-ingress
  namespace: cau3
spec:
  podSelector: {}
  policyTypes:
  - Ingress
EOF

$ kubectl apply -f tel4vn-bai3-webapp-deployment.yaml
```

```
root@k8s-master-test:[~]: kubectl get deploy -n cau3
NAME     READY   UP-TO-DATE   AVAILABLE   AGE
webapp   1/1     1            1           7s
```

```
root@k8s-master-test:[~]: kubectl get sa -n cau3
NAME        SECRETS   AGE
default     0         31s
webapp-sa   0         31s
```

```
root@k8s-master-test:[~]: kubectl get role pod-reader -n cau3
NAME         CREATED AT
pod-reader   2023-02-25T07:47:33Z

root@k8s-master-test:[~]: kubectl describe role pod-reader -n cau3
Name:         pod-reader
Labels:       <none>
Annotations:  <none>
PolicyRule:
  Resources    Non-Resource URLs  Resource Names  Verbs
  ---------    -----------------  --------------  -----
  deployments  []                 []              [get list watch]

```
=============================================================================