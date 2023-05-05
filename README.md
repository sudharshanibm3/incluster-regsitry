# incluster-regsitry
Following are the steps to create In-cluster registry and to execute build, push and pull operations in registry-pod

## 1. Setup Registry Pod

-  Login to IBM Cloud and cluster via CLI
-  Clone this [REPO](https://github.com/sudharshanibm3/incluster-regsitry) in local machine 
-  Run `kubectl apply -f registry-pod.yaml` to create registry pod and service
-  Run `kubectl get ep | grep ^re` and copy the ENDPOINTURL
-  Run `export REGSITRYPATH=<ENDPOINTURL>`
-  Run following command to debug your node
  ```
  kubectl debug node/$(kubectl get nodes -o jsonpath='{ $.items[*].status.addresses[?(@.type=="InternalIP")].address }') -it 
  --image=ubuntu:latest
  ```
- Make directory with name of registry url
```
mkdir host/etc/containerd/certs.d/anothaerone && cd host/etc/containerd/certs.d/${REGSITRYPATH}
``` 
- Create `hosts.toml` and add regsitry URL by executing the following command
```
cat >hosts.toml<<EOL
server = "http:${REGSITRYPATH}"
[host."http:${REGSITRYPATH}"]
  skip_verify=true
EOL
```

## 2. Setup Builder Pod
-  Run `kubectl apply -f builder-pod.yaml`
-  This will build the dockerfile and push to the registry

## 3. Setup Pull pod/ peer pod
- Run `kubectl apply -f peer-pod.yaml`
- This will create a pod with image from in-cluster registry
