![](/images/pic4.png)

## 1. Create taint

```
kubectl taint nodes k8s-master-test cachesystem=redis:NoSchedule
```

## 2. Create pod 

```
cat <<'EOF' > tel4vn-bai4-schedule-pod.yaml
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
EOF

$ kubectl apply -f tel4vn-bai4-schedule-pod.yaml
```

## 3. Show pod

```
$ kubectl get pod web -o wide
NAME   READY   STATUS    RESTARTS   AGE   IP            NODE                 NOMINATED NODE   READINESS GATES
web    1/1     Running   0          5s    10.244.1.15   k8s-worker-test-01   <none>           <none>
```

->> em chưa hiểu sao ko đc ạ. n cứ nhảy vào worker
=============================================================================