# Deployment Prometheus-Grafana on Kubernetes
Kubernetes Cluster Monitoring using Prometheus & Grafana
#
```
helm repo add prometheus-community https://prometheus-community.github.io/helm-charts
helm repo update
helm install my-kube-prometheus-stack prometheus-community/kube-prometheus-stack
kubectl get all
kubectl get pod
kubectl describe pod prometheus-my-kube-prometheus-stack-prometheus-0
kubectl get svc
kubectl expose service my-kube-prometheus-stack-prometheus --type=NodePort --target-port=9090 --name=prometheus-node-port-service
kubectl get svc
kubectl expose service my-kube-prometheus-stack-grafana --type=NodePort --target-port=3000 --name=grafana-node-port-service
kubectl get svc
kubectl get pv
kubectl get pvc
helm list
helm status my-kube-prometheus-stack
kubectl --namespace default get pods -l "release=my-kube-prometheus-stack"
kubectl get storageclasses -n all
kubectl get secrets
kubectl get secret --namespace default my-kube-prometheus-stack-grafana -o jsonpath="{.data.admin-password}" | base64 --decode ; echo
```
#
**Reference:**

[Prometheus-Grafana Integration](https://www.bigbinary.com/blog/prometheus-and-grafana-integration)

[Install Prometheus-Grafana on Kubernetes](https://github.com/tatahnoellimnyuy/install-prometheus-and-grafana-on-kubernetes)

[Deploy Prometheus Grafana on K8s](https://gist.github.com/chadmcrowell/c9b12f32e180a6ad3dc060cdd6d63f05)

[Prometheus-Grafana NFS on K8s](https://www.fosstechnix.com/kubernetes-cluster-monitoring-with-prometheus-and-grafana/)

[]()
[]()
[]()
[]()



