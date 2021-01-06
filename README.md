# ISTIO-Security-JWT
This code demonstrates Istio authorization policy to enforce access based on a JSON Web Token (JWT)

## Prerequisite

```
a) Kubernetes Cluster (1.17.14-gke.1600 or above)
b) Istio Instalation (istio-1.8.1 or above)

```

### Agenda

```
a) Secure workload with JWT Authentication
b) Secure workload with JWT Authorization Policy
c) Secure workload with JWT Authorization Policy with claims
d) Test and verify

```
## SetUp Workload

```
# Download the source code
  
    git clone https://github.com/mosesalphonse/ISTIO-Security-JWT.git
    
    cd ISTIO-Security-JWT
    
# Create namespace
 
   kubectl create ns sashvin

# Deploy the workloads into the namespce

    kubectl apply -f <(istioctl kube-inject -f sashquar_workload.yaml) -n sashvin
    kubectl apply -f <(istioctl kube-inject -f sleep_workload.yaml) -n sashvin

# Test and verify

#Verify that sleep successfully communicates with sashquar workload using below commands(HTTP status code 200):

    kubectl exec "$(kubectl get pod -l app=sleep -n sashvin -o jsonpath={.items..metadata.name})" -c sleep -n sashvin -- curl http://sashquar.sashvin:8000/knative -s -o /dev/null -w "%{http_code}\n"

    kubectl exec "$(kubectl get pod -l app=sleep -n sashvin -o jsonpath={.items..metadata.name})" -c sleep -n sashvin -- curl http://sashquar.sashvin:8000/knative -s
   
```

