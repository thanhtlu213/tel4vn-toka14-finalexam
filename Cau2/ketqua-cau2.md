![](/images/pic2.png)

## 1. Create multi Multi-Pod with volume "vol"

```
$ cat <<'EOF' > tel4vn-bai2-multipod.yaml
---
apiVersion: v1
kind: Namespace
metadata:
   name: cau2

---
apiVersion: v1
kind: Pod
metadata:
  name: multi-pod
  namespace: cau2
  labels:
    app: multi-pod
spec:
  volumes:
    - name: vol
      emptyDir: {}
  containers:
  - name: pod1
    image: busybox:1.31.1
    command: ["/bin/sh"]
    args: ["-c", "while true; do echo $(hostname) $(date) >> /vol/date.log; sleep 5; done"]
    volumeMounts:
      - name: vol
        mountPath: /vol/date.log
  
  - name: pod2
    image: nginx:latest
    volumeMounts:
      - name: vol
        mountPath: /usr/share/nginx/html
EOF

$ kubectl apply -f tel4vn-bai2-multipod.yaml
```

## 2. Check status deployment, pod

```
$ kubectl get pod -o wide -n cau2
NAME        READY   STATUS    RESTARTS   AGE   IP           NODE                 NOMINATED NODE   READINESS GATES
multi-pod   2/2     Running   0          22s   10.244.2.3   k8s-worker-test-02   <none>           <none>
```

```
$ kubectl describe pod/multi-pod -n cau2
Name:             multi-pod
Namespace:        cau2
Priority:         0
Service Account:  default
Node:             k8s-worker-test-02/172.16.1.27
Start Time:       Sat, 25 Feb 2023 07:32:25 +0000
Labels:           app=multi-pod
Annotations:      <none>
Status:           Running
IP:               10.244.2.3
IPs:
  IP:  10.244.2.3
Containers:
  pod1:
    Container ID:  containerd://9215c96b20626f88d9fa69ef2557e721f4f0b203270799d2cd26d95e8bf64aab
    Image:         busybox:1.31.1
    Image ID:      docker.io/library/busybox@sha256:95cf004f559831017cdf4628aaf1bb30133677be8702a8c5f2994629f637a209
    Port:          <none>
    Host Port:     <none>
    Command:
      /bin/sh
    Args:
      -c
      while true; do echo $(hostname) $(date) >> /vol/date.log; sleep 5; done
    State:          Running
      Started:      Sat, 25 Feb 2023 07:32:31 +0000
    Ready:          True
    Restart Count:  0
    Environment:    <none>
    Mounts:
      /var/run/secrets/kubernetes.io/serviceaccount from kube-api-access-n6chh (ro)
      /vol/date.log from vol (rw)
  pod2:
    Container ID:   containerd://1917189a00a7c33ff91685973091540d1f60758e9cd6812a89bb158a60d17b62
    Image:          nginx:latest
    Image ID:       docker.io/library/nginx@sha256:6650513efd1d27c1f8a5351cbd33edf85cc7e0d9d0fcb4ffb23d8fa89b601ba8 
    Port:           <none>
    Host Port:      <none>
    State:          Running
      Started:      Sat, 25 Feb 2023 07:32:46 +0000
    Ready:          True
    Restart Count:  0
    Environment:    <none>
    Mounts:
      /usr/share/nginx/html from vol (rw)
      /var/run/secrets/kubernetes.io/serviceaccount from kube-api-access-n6chh (ro)
Conditions:
  Type              Status
  Initialized       True
  Ready             True
  ContainersReady   True
  PodScheduled      True
Volumes:
  vol:
    Type:       EmptyDir (a temporary directory that shares a pod's lifetime)
    Medium:
    SizeLimit:  <unset>
  kube-api-access-n6chh:
    Type:                    Projected (a volume that contains injected data from multiple sources)
    TokenExpirationSeconds:  3607
    ConfigMapName:           kube-root-ca.crt
    ConfigMapOptional:       <nil>
    DownwardAPI:             true
QoS Class:                   BestEffort
Node-Selectors:              <none>
Tolerations:                 node.kubernetes.io/not-ready:NoExecute op=Exists for 300s
                             node.kubernetes.io/unreachable:NoExecute op=Exists for 300s
Events:
  Type    Reason     Age   From               Message
  ----    ------     ----  ----               -------
  Normal  Scheduled  53s   default-scheduler  Successfully assigned cau2/multi-pod to k8s-worker-test-02
  Normal  Pulling    53s   kubelet            Pulling image "busybox:1.31.1"
  Normal  Pulled     47s   kubelet            Successfully pulled image "busybox:1.31.1" in 5.551978908s (5.552000152s including waiting)
  Normal  Created    47s   kubelet            Created container pod1
  Normal  Started    47s   kubelet            Started container pod1
  Normal  Pulling    47s   kubelet            Pulling image "nginx:latest"
  Normal  Pulled     32s   kubelet            Successfully pulled image "nginx:latest" in 15.127748435s (15.127854931s including waiting)
  Normal  Created    32s   kubelet            Created container pod2
  Normal  Started    32s   kubelet            Started container pod2

```
=============================================================================
