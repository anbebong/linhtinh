# MicroK8s & Kubernetes Command Cheatsheet

---

## 1. Quản lý MicroK8s

```bash
microk8s status                      # Xem trạng thái
microk8s status --wait-ready         # Đợi sẵn sàng
microk8s start                       # Khởi động
microk8s stop                        # Dừng
microk8s reset                       # Reset cluster (xóa toàn bộ)
microk8s inspect                     # Chẩn đoán

microk8s enable <addon>              # Bật addon (dns, storage, ingress, registry, dashboard, metrics-server...)
microk8s disable <addon>             # Tắt addon

microk8s add-node                    # Tạo token join (master)
microk8s join <token>                # Join vào cluster (worker)
microk8s remove-node <node-name>     # Xóa node

microk8s dashboard-proxy             # Mở dashboard UI + lấy token
```

**Cấu hình quyền kubectl:**
```bash
sudo usermod -a -G microk8s $USER
newgrp microk8s
sudo snap alias microk8s.kubectl kubectl
microk8s config > ~/.kube/config     # Export kubeconfig
```

---

## 2. Cluster & Node

```bash
microk8s kubectl cluster-info
microk8s kubectl get nodes
microk8s kubectl get nodes -o wide
microk8s kubectl describe node <name>
microk8s kubectl top node            # CPU/memory (cần metrics-server)
```

---

## 3. Namespace

```bash
microk8s kubectl get ns
microk8s kubectl create ns <name>
microk8s kubectl delete ns <name>
microk8s kubectl config set-context --current --namespace=<name>  # Đổi namespace mặc định
```

---

## 4. Pod

```bash
microk8s kubectl get pods
microk8s kubectl get pods -A         # Tất cả namespace
microk8s kubectl get pods -n <ns>
microk8s kubectl get pods -o wide
microk8s kubectl get pods -l app=nginx  # Lọc theo label
microk8s kubectl describe pod <name>
microk8s kubectl logs <name>
microk8s kubectl logs -f <name>      # Follow
microk8s kubectl logs --previous <name>  # Log trước khi restart
microk8s kubectl logs <name> -c <container>  # Log container cụ thể
microk8s kubectl exec -it <name> -- /bin/sh
microk8s kubectl delete pod <name>
microk8s kubectl top pod             # CPU/memory
```

---

## 5. Deployment

```bash
microk8s kubectl get deploy
microk8s kubectl create deployment <name> --image=<image>
microk8s kubectl describe deploy <name>
microk8s kubectl get deploy <name> -o yaml

microk8s kubectl scale deployment <name> --replicas=5
microk8s kubectl set image deployment/<name> <container>=<new-image>
microk8s kubectl rollout status deployment/<name>
microk8s kubectl rollout history deployment/<name>
microk8s kubectl rollout undo deployment/<name>
microk8s kubectl rollout undo deployment/<name> --to-revision=2
microk8s kubectl rollout restart deployment/<name>

microk8s kubectl delete deployment <name>
```

---

## 6. Service

```bash
microk8s kubectl get svc
microk8s kubectl get svc -A
microk8s kubectl describe svc <name>

microk8s kubectl expose deployment <name> --type=ClusterIP --port=80
microk8s kubectl expose deployment <name> --type=NodePort --port=80 --target-port=80 --node-port=30080

microk8s kubectl delete svc <name>
```

**Service types:** ClusterIP (trong cluster), NodePort (mở cổng node 30000-32767), LoadBalancer (cloud/MetalLB)

---

## 7. Ingress

```bash
microk8s kubectl get ingress
microk8s kubectl get ingress -A
microk8s kubectl describe ingress <name>
microk8s kubectl delete ingress <name>
```

---

## 8. ConfigMap & Secret

```bash
# ConfigMap
microk8s kubectl create configmap <name> --from-literal=key=value
microk8s kubectl create configmap <name> --from-file=<path>
microk8s kubectl get cm
microk8s kubectl describe cm <name>
microk8s kubectl get cm <name> -o yaml
microk8s kubectl delete cm <name>

# Secret
microk8s kubectl create secret generic <name> --from-literal=password=xxx
microk8s kubectl create secret generic <name> --from-file=<path>
microk8s kubectl get secret
microk8s kubectl describe secret <name>
microk8s kubectl get secret <name> -o yaml
microk8s kubectl delete secret <name>
```

---

## 9. PVC & PV

```bash
microk8s kubectl get pvc
microk8s kubectl get pvc -A
microk8s kubectl describe pvc <name>
microk8s kubectl get pv
microk8s kubectl describe pv <name>
microk8s kubectl get sc               # StorageClass
microk8s kubectl delete pvc <name>
```

---

## 10. StatefulSet, DaemonSet, Job, CronJob

```bash
microk8s kubectl get sts
microk8s kubectl get ds
microk8s kubectl get jobs
microk8s kubectl get cj
microk8s kubectl describe <resource> <name>
microk8s kubectl delete <resource> <name>
```

---

## 11. HPA

```bash
microk8s kubectl get hpa
microk8s kubectl describe hpa <name>
microk8s kubectl autoscale deployment <name> --cpu-percent=50 --min=2 --max=10
microk8s kubectl delete hpa <name>
```

---

## 12. Manifest (YAML)

```bash
microk8s kubectl apply -f <file.yaml>
microk8s kubectl apply -f <directory>/
microk8s kubectl delete -f <file.yaml>
microk8s kubectl get -f <file.yaml>
microk8s kubectl describe -f <file.yaml>

# Export manifest
microk8s kubectl get deploy <name> -o yaml > deploy.yaml
microk8s kubectl get all --all-namespaces -o yaml > backup.yaml

# Dry-run
microk8s kubectl apply -f <file.yaml> --dry-run=client
```

---

## 13. Port-forward

```bash
microk8s kubectl port-forward svc/<name> 8080:80
microk8s kubectl port-forward pod/<name> 8080:80
microk8s kubectl port-forward -n <ns> svc/<name> 8080:80
```

---

## 14. Debug & Events

```bash
microk8s kubectl describe pod <name>
microk8s kubectl logs <name>
microk8s kubectl logs -f <name>
microk8s kubectl logs --previous <name>
microk8s kubectl exec -it <name> -- /bin/sh
microk8s kubectl get events -A --sort-by='.metadata.creationTimestamp'
microk8s kubectl get events -n <ns> --sort-by='.metadata.creationTimestamp'
microk8s kubectl top pod
microk8s kubectl top node
```

---

## 15. Registry nội bộ

```bash
microk8s enable registry             # Bật registry (localhost:32000)

docker build -t localhost:32000/myapp:latest .
docker push localhost:32000/myapp:latest

# Trong manifest: image: localhost:32000/myapp:latest
```

**Lưu ý:** Nếu Docker chạy ngoài MicroK8s VM, dùng `<NODE_IP>:32000`

---

## 16. Xem tất cả resource

```bash
microk8s kubectl get all -n <ns>
microk8s kubectl get all -A
microk8s kubectl api-resources
```

---

## 17. Context & Kubeconfig

```bash
microk8s kubectl config view
microk8s kubectl config get-contexts
microk8s kubectl config use-context <context>
microk8s kubectl config current-context
microk8s config                      # Export kubeconfig
```

---

## 18. RBAC

```bash
microk8s kubectl get roles -A
microk8s kubectl get rolebindings -A
microk8s kubectl get clusterroles
microk8s kubectl get clusterrolebindings
microk8s kubectl get sa -A
```

---

## 19. Lệnh hữu ích

```bash
microk8s kubectl get nodes,pods,svc,deploy -A
microk8s kubectl delete pods --all -n <ns>
microk8s kubectl delete all --all -n <ns>
microk8s kubectl rollout restart deployment/<name>
microk8s kubectl version
microk8s kubectl get pods -w        # Watch realtime
microk8s kubectl label pod <name> env=prod
microk8s kubectl annotate pod <name> desc="test"
microk8s kubectl edit deployment <name>
```

---

## 20. Alias

Thêm vào `~/.bashrc`:

```bash
alias k='microk8s kubectl'
alias kgp='microk8s kubectl get pods'
alias kgpa='microk8s kubectl get pods -A'
alias kgs='microk8s kubectl get svc'
alias kgd='microk8s kubectl get deploy'
alias kl='microk8s kubectl logs -f'
alias kx='microk8s kubectl exec -it'
alias kdesc='microk8s kubectl describe'
alias kpf='microk8s kubectl port-forward'
alias ka='microk8s kubectl apply -f'
alias kdel='microk8s kubectl delete'
```

Sau đó: `source ~/.bashrc`

---

## 21. Workflow nhanh

```bash
# Deploy nginx
microk8s kubectl create deployment nginx --image=nginx:stable
microk8s kubectl expose deployment nginx --type=NodePort --port=80 --node-port=30080
microk8s kubectl get pods -o wide
microk8s kubectl get svc

# Truy cập
microk8s kubectl port-forward svc/nginx 8080:80  # http://localhost:8080

# Scale
microk8s kubectl scale deployment nginx --replicas=3

# Update
microk8s kubectl set image deployment/nginx nginx=nginx:1.25

# Logs
microk8s kubectl logs -l app=nginx

# Xóa
microk8s kubectl delete svc nginx
microk8s kubectl delete deployment nginx
```

---

## 22. Troubleshooting

**Pod Pending:**
```bash
microk8s kubectl describe pod <name>
microk8s kubectl get events -A --sort-by='.metadata.creationTimestamp'
microk8s kubectl top node
```

**ImagePullBackOff:**
```bash
microk8s kubectl describe pod <name>  # Kiểm tra image name, registry
```

**CrashLoopBackOff:**
```bash
microk8s kubectl logs <name>
microk8s kubectl logs --previous <name>
```

**Node NotReady:**
```bash
microk8s status --wait-ready
microk8s inspect
```

**Service không truy cập được:**
```bash
microk8s kubectl get pods --show-labels
microk8s kubectl describe svc <name>
microk8s kubectl get endpoints <name>
```

---

## 23. Ví dụ Manifest YAML

```yaml
apiVersion: v1
kind: Namespace
metadata:
  name: demo
---
apiVersion: v1
kind: ConfigMap
metadata:
  name: demo-config
  namespace: demo
data:
  APP_MSG: "Hello"
---
apiVersion: v1
kind: Secret
metadata:
  name: demo-secret
  namespace: demo
type: Opaque
stringData:
  SECRET_MSG: "s3cr3t"
---
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: demo-pvc
  namespace: demo
spec:
  accessModes: [ReadWriteOnce]
  resources:
    requests:
      storage: 500Mi
---
apiVersion: apps/v1
kind: Deployment
metadata:
  name: nginx
  namespace: demo
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
        image: nginx:stable
        ports:
        - containerPort: 80
        env:
        - name: APP_MSG
          valueFrom:
            configMapKeyRef:
              name: demo-config
              key: APP_MSG
        volumeMounts:
        - name: data
          mountPath: /data
        readinessProbe:
          httpGet:
            path: /
            port: 80
          initialDelaySeconds: 5
          periodSeconds: 5
        livenessProbe:
          httpGet:
            path: /
            port: 80
          initialDelaySeconds: 15
          periodSeconds: 20
        resources:
          requests:
            cpu: "100m"
            memory: "64Mi"
          limits:
            cpu: "500m"
            memory: "256Mi"
      volumes:
      - name: data
        persistentVolumeClaim:
          claimName: demo-pvc
---
apiVersion: v1
kind: Service
metadata:
  name: nginx
  namespace: demo
spec:
  type: ClusterIP
  selector:
    app: nginx
  ports:
  - port: 80
    targetPort: 80
---
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: nginx-ingress
  namespace: demo
spec:
  rules:
  - host: nginx.local
    http:
      paths:
      - path: /
        pathType: Prefix
        backend:
          service:
            name: nginx
            port:
              number: 80
```

**Áp dụng:**
```bash
microk8s kubectl apply -f manifest.yaml
microk8s kubectl get all -n demo
sudo sh -c 'echo "127.0.0.1 nginx.local" >> /etc/hosts'
curl http://nginx.local
```

---
