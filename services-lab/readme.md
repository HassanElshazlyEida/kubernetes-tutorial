## Create two pods with Nginx running.
```yaml
# pods.yaml
apiVersion: v1
kind: Pod
metadata:
  name: web-app01
  labels:
    app: web-app
spec:
  containers:
  - name: web
    image: nginx
    ports:
    - containerPort: 80
---
apiVersion: v1
kind: Pod
metadata:
  name: web-app02
  labels:
    app: web-app
spec:
  containers:
  - name: web
    image: nginx
    ports:
    - containerPort: 80
```

1. kubectl apply -f pods.yaml

2. kubectl get pods

3. Login to each pod 

    ```bash
    kubectl exec -it web-app01 -- bash
    ```
4. Install `vim` since it is not installed by default in the image.
    ```bash
    apt update
    apt install -y vim
    ```
5. access index.html file using `vim`
    ```bash 
      vim /usr/share/nginx/html/index.html
    ```

6. Change the index.html file to say something like "I am Pod 01" in `<h1>`

7. Do the same thing for the second pod:
    ```bash
    kubectl exec -it web-app02 -- bash
    ```

## Create the following services:

a. ClusterIP:
```yaml
apiVersion: v1
kind: Service
metadata:
  name: my-clusterip-service
spec:
  selector:
    app: web-app
  ports:
    - protocol: TCP
      port: 80
      targetPort: 80
```
Apply the manifest to the cluster.

View the services and get the service IP:

```bash 
kubectl get svc 
```
First, let's try to access the service using a third pod:
```bash
kubectl run ubuntu --image=ubuntu -it --rm -- /bin/bash
```
Inside the pod, let's install the curl command:
```bash
apt update
apt install -y curl
```
Now, let's see if we can access the `ClusterIP` service:
```bash 
curl my-clusterip-service.default.svc.cluster.local 
```
#### several times to show the responses from both pods

We can also run the same command without having to to specify the FQDN:

```bash 
curl my-clusterip-service
```
#### several times to show the responses from both pods
Exit from the current shell and use the laptop to run this command.

Try to access the service using a port-forward command like the following:
```bash
kubectl port-forward service/my-clusterip-service 8080:80
```
Open the browser and try to access the service by going to `localhost:8080`. 

## The NodePort lab
You can also run 
```bash
kubectl get nodes -o wide
```
1. NodePort: Create a new service of type `NodePort` (copy the existing one and make changes)
```yaml
apiVersion: v1
kind: Service
metadata:
  name: my-nodeport-service
spec:
  selector:
    app: web-app
  ports:
    - protocol: TCP
      port: 80
      targetPort: 80
      nodePort: 30007
  type: NodePort
```

Apply the manifest to the cluster.

View the services:
```bash
ahmed@master01:~$ kubectl get svc
NAME                   TYPE        CLUSTER-IP      EXTERNAL-IP   PORT(S)        AGE
kubernetes             ClusterIP   10.96.0.1       <none>        443/TCP        2y22d
my-clusterip-service   ClusterIP   10.98.195.107   <none>        80/TCP         8m56s
my-nodeport-service    NodePort    10.96.241.209   <none>        80:30007/TCP   7s
```
Try accessing the service through any of the nodes in the cluster: 192.168.2.76, 192.168.2.77, or 192.168.2.78 (use your own cluster IPs).

Refreshing the page on each node does NOT provide load balancing.

