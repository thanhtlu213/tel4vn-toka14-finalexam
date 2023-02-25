![](/images/pic1.png)

## 1. Create cluster kubernetes:

- Run command on Master
```
$ kubeadm token create 
$ openssl x509 -pubkey -in /etc/kubernetes/pki/ca.crt | openssl rsa -pubin -outform der 2>/dev/null | openssl dgst -sha256 -hex | sed 's/^.* //'
```

- Run command on Worker 1, Worker 2
```
$ kubeadm join 172.16.1.25:6443 --token yszw81.d7uozabto6hb6uds --discovery-token-ca-cert-hash sha256:d4ac19c84e601a0f4cfd9f86abe8c5accfdc45325c614991ca9ebed97603f5e6
```

- View roles, AGE, VERSION, INTERNAL-IP, EXTERNAL-IP, OS-IMAGE,..:
```
$ kubectl get nodes -owide
NAME                   STATUS   ROLES                  AGE   VERSION   INTERNAL-IP   EXTERNAL-IP   OS-IMAGE             KERNEL-VERSION      CONTAINER-RUNTIME
NAME                 STATUS   ROLES           AGE   VERSION   INTERNAL-IP   EXTERNAL-IP   OS-IMAGE             KERNEL-VERSION      CONTAINER-RUNTIME  
k8s-master-test      Ready    control-plane   12h   v1.25.6   172.16.1.25   <none>        Ubuntu 20.04.4 LTS   5.4.0-139-generic   containerd://1.6.18
k8s-worker-test-01   Ready    <none>          12h   v1.25.6   172.16.1.26   <none>        Ubuntu 20.04.4 LTS   5.4.0-139-generic   containerd://1.6.18
k8s-worker-test-02   Ready    <none>          12h   v1.25.6   172.16.1.27   <none>        Ubuntu 20.04.4 LTS   5.4.0-139-generic   containerd://1.6.18
```

## 2. Create deployment

```
$ cat <<'EOF' > tel4vn-bai1-nginx-deployment.yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: webapp
  labels:
    app: nginx
spec:
  replicas: 2
  selector:
    matchLabels:
      app: nginx
  template:
    metadata:
      labels:                      
        app: nginx
    spec:                                   
      containers:
      - name: nginx
        image: nginx:1.18.0
        resources:
          limits:
            cpu: "0.5"
          requests:
            cpu: "0.2"
        ports:
        - containerPort: 80
EOF

$ kubectl apply -f tel4vn-bai1-nginx-deployment.yaml
```

-  Check status deployment, pod
```
$ kubectl get deploy -o wide
NAME     READY   UP-TO-DATE   AVAILABLE   AGE   CONTAINERS   IMAGES         SELECTOR
webapp   2/2     2            2           45s   nginx        nginx:1.18.0   app=nginx

$ kubectl get pod -o wide
NAME                      READY   STATUS    RESTARTS   AGE   IP           NODE                 NOMINATED NODE   READINESS GATES
webapp-54ddbfd7b6-7fsrm   1/1     Running   0          90s   10.244.1.3   k8s-worker-test-01   <none>           <none>
webapp-54ddbfd7b6-cqzd5   1/1     Running   0          90s   10.244.2.2   k8s-worker-test-02   <none>           <none>
```

## 3. Expose pod using nodeport

```
$ kubectl expose deployment webapp --port=80 --target-port=80 --type NodePort
service/webapp exposed
```

## 4. Check service, access webapp

```
$ kubectl get svc
NAME         TYPE        CLUSTER-IP      EXTERNAL-IP   PORT(S)        AGE
kubernetes   ClusterIP   10.96.0.1       <none>        443/TCP        12h
webapp       NodePort    10.101.173.96   <none>        80:31082/TCP   24s
```

```
$ curl -I http://172.16.1.26:31082
HTTP/1.1 200 OK
Server: nginx/1.18.0
Date: Sat, 25 Feb 2023 07:19:53 GMT
Content-Type: text/html
Content-Length: 612
Last-Modified: Tue, 21 Apr 2020 14:09:01 GMT
Connection: keep-alive
ETag: "5e9efe7d-264"
Accept-Ranges: bytes

$ curl -I http://172.16.1.27:31082
HTTP/1.1 200 OK
Server: nginx/1.18.0
Date: Sat, 25 Feb 2023 07:20:16 GMT
Content-Type: text/html
Content-Length: 612
Last-Modified: Tue, 21 Apr 2020 14:09:01 GMT
Connection: keep-alive
ETag: "5e9efe7d-264"
Accept-Ranges: bytes
```
=============================================================================