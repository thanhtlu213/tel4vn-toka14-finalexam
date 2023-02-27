![](/images/pic4.png)

## I. Use Taint
## 1. Create taint

```
kubectl taint nodes k8s-master-test cachesystem=redis:NoSchedule
```

## 2. Create pod 

- Describe node master
```
$ kubectl describe node k8s-master-test
...
Labels:             beta.kubernetes.io/arch=amd64
                    beta.kubernetes.io/os=linux
                    kubernetes.io/arch=amd64
                    kubernetes.io/hostname=k8s-master-test
                    kubernetes.io/os=linux
                    node-role.kubernetes.io/control-plane=
Taints:             cachesystem=redis:NoSchedule
...
```

- Describe node worker
```
Labels:             beta.kubernetes.io/arch=amd64
                    beta.kubernetes.io/os=linux  
                    kubernetes.io/arch=amd64
                    kubernetes.io/hostname=k8s-worker-test-01
                    kubernetes.io/os=linux
Taints:             <none>
...     
```

- Create Pod
```
$ cat <<'EOF' > tel4vn-bai4-schedule-pod.yaml
apiVersion: v1
kind: Pod
metadata:
  name: web
spec:
  containers:
  - name: redis
    image: redis:latest
  tolerations:
  - key: cachesystem
    value: redis
    effect: NoSchedule
    operator: Equal
  affinity:
    nodeAffinity:
      requiredDuringSchedulingIgnoredDuringExecution:
        nodeSelectorTerms:
        - matchExpressions:
          - key: node-role.kubernetes.io/control-plane
            operator: Exists
EOF

$ kubectl apply -f tel4vn-bai4-schedule-pod.yaml
```

## 3. Show pod

```
$ kubectl get pod web -o wide
NAME   READY   STATUS    RESTARTS   AGE    IP           NODE              NOMINATED NODE   READINESS GATES
web    1/1     Running   0          2m3s   10.244.0.4   k8s-master-test   <none>           <none>
```

## II. Use Static Pod
- Create Pod
```
# Run this command on the node where kubelet is running

$ cd /etc/kubernetes/manifests
$ cat <<'EOF' > tel4vn-bai4-schedule-pod.yaml
apiVersion: v1
kind: Pod
metadata:
  name: web
spec:
  containers:
  - name: redis
    image: redis:latest
EOF

$ KUBELET_ARGS="--cluster-dns=10.254.0.10 --cluster-domain=kube.local --pod-manifest-path=/etc/kubernetes/manifests/"

$ systemctl restart kubelet
```

- Show Pod
```
kubectl get pod -o wide
NAME                      READY   STATUS    RESTARTS       AGE     IP           NODE                 NOMINATED NODE   READINESS GATES
web-k8s-master-test       1/1     Running   1 (3m6s ago)   2m18s   10.244.0.7   k8s-master-test      <none>           <none>
```

=============================================================================