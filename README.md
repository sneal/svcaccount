# Kubernetes API Access Sample
Example of using the Golang k8s client bindings to list all cluster nodes from a running pod via a service account context.

The below sample commands assume you have the following installed and running:
- Podman (instead of Docker)
- Microk8s with (insecure) registry enabled

## Create Service Account and RBAC
```sh
# Create a service account
microk8s kubectl create serviceaccount svcaccount-serviceaccount

# Create a new cluster role that can read nodes
microk8s kubectl apply -f node-reader-role.yml

# Create cluster role binding to service account
microk8s kubectl create clusterrolebinding svcaccount-readonly \
  --clusterrole=node-reader \
  --serviceaccount=default:svcaccount-serviceaccount
```

## Build Image & Run
```bash
# Build image (localhost is relative to the Node running app)
podman build . -t localhost:32000/svcaccount:latest

# Push image to your insecure local registry (IP is the Node relative to Mac)
podman push 7ce9e0aec42a 192.168.64.2:32000/svcaccount:latest

# Run the pod override the service account
microk8s kubectl run svcaccount \
    -it \
    --image=localhost:32000/svcaccount:latest \
    --overrides='{ "spec": { "serviceAccountName": "svcaccount-serviceaccount" } }' \
    --quiet \
    --image-pull-policy=IfNotPresent \
    --restart=Never \
    --rm \
    --override-type=strategic
```
