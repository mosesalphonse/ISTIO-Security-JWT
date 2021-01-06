# ISTIO-Security-JWT
This code demonstrates Istio authorization policy to enforce access based on a JSON Web Token (JWT)

## Prerequisite

```
a) Kubernetes Cluster (1.17.14-gke.1600 or above)
b) Istio Instalation (istio-1.8.1 or above)

```

### Agenda

```
a) Secure workload with JWT Request Authentication
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

# Deploy the workloads into the namespce and inject Envoy sidecar alongside workloads

    kubectl apply -f <(istioctl kube-inject -f sashquar_workload.yaml) -n sashvin
    kubectl apply -f <(istioctl kube-inject -f sleep_workload.yaml) -n sashvin

# Test and verify

#Verify that sleep successfully communicates with sashquar workload using below commands(HTTP status code 200):

    kubectl exec "$(kubectl get pod -l app=sleep -n sashvin -o jsonpath={.items..metadata.name})" -c sleep -n sashvin -- curl http://sashquar.sashvin:8000/knative -s -o /dev/null -w "%{http_code}\n"

    kubectl exec "$(kubectl get pod -l app=sleep -n sashvin -o jsonpath={.items..metadata.name})" -c sleep -n sashvin -- curl http://sashquar.sashvin:8000/knative -s
   
```

## Apply JWT Request Authentication

```
# Create authentication policy for the sashquar workload. This policy for sashquar workload which accepts a JWT issued by testing@secure.istio.io:

    kubectl apply -f jwt-authentication-sash.yaml
         
# Test

# Verify that a request with an invalid JWT is denied(http ststuc code:401):
 
   kubectl exec "$(kubectl get pod -l app=sleep -n sashvin -o jsonpath={.items..metadata.name})" -c sleep -n sashvin -- curl "http://sashquar.sashvin:8000/knative" -s -o /dev/null -H "Authorization: Bearer invalidToken" -w "%{http_code}\n"

# Verify that a request without a JWT is allowed because there is no authorization policy (http ststuc code:200):

    kubectl exec "$(kubectl get pod -l app=sleep -n sashvin -o jsonpath={.items..metadata.name})" -c sleep -n sashvin -- curl "http://sashquar.sashvin:8000/knative" -s -o /dev/null -w "%{http_code}\n"

    kubectl exec "$(kubectl get pod -l app=sleep -n sashvin -o jsonpath={.items..metadata.name})" -c sleep -n sashvin -- curl "http://sashquar.sashvin:8000/knative" -s

```

## Apply JWT Authorizations Policy

```
# Apply JWT Authorizations Policy for sashquar workload. The policy requires all requests to the sashquar workload to have a valid JWT with requestPrincipal set to testing@secure.istio.io/testing@secure.istio.io. Istio constructs the requestPrincipal by combining the iss and sub of the JWT token with a / separator as shown:

    kubectl apply -f jwt-authorization-sash.yaml
         
# Test

# Get the token (this can be changed. you may use any other identity provider like keycloak, etc)
 
   TOKEN=$(curl https://raw.githubusercontent.com/istio/istio/release-1.8/security/tools/jwt/samples/demo.jwt -s) && echo "$TOKEN" | cut -d '.' -f2 - | base64 --decode -

# Verify that a request with a valid JWT is allowed(HTTP status code should be 200):

    kubectl exec "$(kubectl get pod -l app=sleep -n sashvin -o jsonpath={.items..metadata.name})" -c sleep -n sashvin -- curl "http://sashquar.sashvin:8000/knative" -s -o /dev/null -H "Authorization: Bearer $TOKEN" -w "%{http_code}\n"
    
    kubectl exec "$(kubectl get pod -l app=sleep -n sashvin -o jsonpath={.items..metadata.name})" -c sleep -n sashvin -- curl "http://sashquar.sashvin:8000/knative"  -H "Authorization: Bearer $TOKEN" -s

# Verify that a request without a JWT is denied ((HTTP status code should be 403):

    kubectl exec "$(kubectl get pod -l app=sleep -n sashvin -o jsonpath={.items..metadata.name})" -c sleep -n sashvin -- curl "http://sashquar.sashvin:8000/knative" -s -o /dev/null -w "%{http_code}\n"
    
```
## Update authorization policy 

```
# Update authorization policy to also require the JWT to have a claim named groups containing the value group1:

   kubectl apply -f jwt-authorization-claim-sash.yaml
         
# Test

# Get the JWT that sets the groups claim to a list of strings: group1 and group2:
   
   TOKEN=$(curl https://raw.githubusercontent.com/istio/istio/release-1.8/security/tools/jwt/samples/demo.jwt -s) && echo "$TOKEN" | cut -d '.' -f2 - | base64 --decode -
    
#Verify that a request with the JWT that includes group1 in the groups claim is allowed (HTTP status code should be 200):

    kubectl exec "$(kubectl get pod -l app=sleep -n sashvin -o jsonpath={.items..metadata.name})" -c sleep -n sashvin -- curl "http://sashquar.sashvin:8000/knative" -s -o /dev/null -H "Authorization: Bearer $TOKEN_GROUP" -w "%{http_code}\n"
    
    kubectl exec "$(kubectl get pod -l app=sleep -n sashvin -o jsonpath={.items..metadata.name})" -c sleep -n sashvin -- curl "http://sashquar.sashvin:8000/knative" -s  -H "Authorization: Bearer $TOKEN_GROUP"
    
    kubectl exec "$(kubectl get pod -l app=sleep -n sashvin -o jsonpath={.items..metadata.name})" -c sleep -n sashvin -- curl "http://sashquar.sashvin:8000/knative/greeting/Moses" -s  -H "Authorization: Bearer $TOKEN_GROUP"

#Verify that a request with a JWT, which doesn’t have the groups claim is rejected (HTTP status code should be 403): 

    kubectl exec "$(kubectl get pod -l app=sleep -n sashvin -o jsonpath={.items..metadata.name})" -c sleep -n sashvin -- curl "http://sashquar.sashvin:8000/knative" -s -o /dev/null -H "Authorization: Bearer $TOKEN" -w "%{http_code}\n"
    
    
```
