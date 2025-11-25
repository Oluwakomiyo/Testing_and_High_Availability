This project deploys a simple **NGINX frontend** and **HTTP backend API** on a Kubernetes cluster using Minikube.  
It demonstrates:

- Deployments with 2 replicas  
- Pod anti-affinity  
- Liveness & readiness probes  
- Horizontal Pod Autoscalers (HPA)  
- ClusterIP & NodePort services  
- Service discovery  
- Load testing for autoscaling  
- Metrics server setup

## Deployment Commands

### **Apply Deployments & Services**

``` bash
kubectl apply -f nginx_frontend.yaml
kubectl apply -f http_backend.yaml
kubectl apply -f frontend_svc.yaml
kubectl apply -f backend_svc.yaml
kubectl apply -f frontend_hpa.yaml
kubectl apply -f backend_hpa.yaml
```

### **Check Status**

``` bash
kubectl get pods
kubectl get deployments
kubectl get svc
kubectl get hpa
```

## Load Testing for HPA

using busybox pod for continuous traffic
``` bash
kubectl run curl-client --image=busybox:1.35 --rm -it --restart=Never -- /bin/sh
```
Inside the shell:
``` bash
while true; do
  for i in $(seq 1 50); do
    wget -qO /dev/null http://localhost:5678/ &
  done
  wait
done
```
Monitoring autoscaling:
``` bash
kubectl get hpa -w
kubectl top pods
kubectl top nodes
```

## Install Metrics Server(required for HPA)

``` bash 
kubectl apply -f https://github.com/kubernetes-sigs/metrics-server/releases/latest/download/components.yaml
```
## verify

``` bash
kubectl get deployment metrics-server -n kube-system
kubectl top nodes
```

## Metrics Server Issue & Fix

An **ErrImagePull** occurred while installing the metrics server.

Resolve by patching the deployment:
``` bash
kubectl edit deployment metrics-server -n kube-system
```
Add this to the arguments section of the deployment:
``` bash
--kubelet-insecure-tls
```

## 🔍 Service Discovery Tests

Access via minikube service URL:
``` bash
minikube service frontend --url
```

Verify connectivity using:
``` bash
kubectl exec -it <pod-name> -- curl http://localhost:5678/
```

## Cleanup

``` bash
kubectl delete -f nginx_frontend.yaml
kubectl delete -f http_backend.yaml
kubectl delete -f frontend_svc.yaml
kubectl delete -f backend_svc.yaml
kubectl delete -f frontend_hpa.yaml
kubectl delete -f backend_hpa.yaml
```

## Conclusion

This project demonstrates:

- High availability with multi-replica deployments  
- Pod anti-affinity distribution  
- Health checks and self-healing  
- Autoscaling behavior triggered via load testing  
- Service discovery between frontend and backend  
- Troubleshooting issues