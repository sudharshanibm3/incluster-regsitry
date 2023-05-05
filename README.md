# incluster-regsitry
![Screenshot 2023-04-27 at 12.15.27 AM.png](https://zenhub.ibm.com/images/63d0c1b66bae02b9f5905ea2/93317411-e360-430f-89f2-9cef5288d300)

Following are the steps to create In-cluster registry and to execute build, push and pull operations in registry-pod

## 1. Setup Registry Pod

-  Login to IBM Cloud and cluster via CLI
-  Clone this [REPO](https://github.com/sudharshanibm3/incluster-regsitry) in local machine 
-  Run `kubectl apply -f registry-pod.yaml` to create registry pod and service
-  Run `kubectl get ep | grep ^re` and copy the ENDPOINTURL
-  Run following command to debug your node
```
kubectl debug node/<NODEIP> -it --image=ubuntu:latest
```
- Export REGISTRYPATH by
```
export REGISTRYPATH="<ENDPOINTURL>"
```
- Make directory with name of registry url
```
mkdir host/etc/containerd/certs.d/${REGISTRYPATH} && cd host/etc/containerd/certs.d/${REGISTRYPATH}
``` 
- Create `hosts.toml` and add regsitry URL by executing the following command
```
cat >hosts.toml<<EOL
server = "http://${REGISTRYPATH}"
[host."http://${REGISTRYPATH}"]
  skip_verify = true
EOL
```
- Exit from node debugger by cmd `exit`

## 2. Setup Builder Pod
- In builder-pod.yaml file update your ENDPOINT URL [in this line](https://github.com/sudharshanibm3/incluster-regsitry/blob/main/builder-pod.yaml#L9)
-  Run `kubectl apply -f builder-pod.yaml`
-  This will build the dockerfile and push to the registry

## 3. Setup Pull pod/ peer pod
- In peer-pod.yaml file update your ENDPOINT URL [in this line](https://github.com/sudharshanibm3/incluster-regsitry/blob/main/peer-pod.yaml#L8)
- Run `kubectl apply -f peer-pod.yaml`
- This will create a pod with image from in-cluster registry
