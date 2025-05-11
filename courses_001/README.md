https://iamunnip.medium.com/kind-local-kubernetes-cluster-part-5-25844d448926

In this article we will talk about how we can configure Ingress in our cluster

Ingress
An API object that manages external access to the cluster
Provides load balancing, SSL termination and name based virtual hosting
Ingress controller is responsible for fulfilling the ingress
An ingress does not expose arbitrary ports or protocols
ingress-nginx is a popular ingress controller

Configuration
Create a cluster using below configuration file, where we have added extra port mappings and node labels.

extraPortMappings allow the local host to make requests to the Ingress controller over ports 80 and 443.
node-labels only allow the ingress controller to run on a specific node matching the label selector.

$ cat kind.yml 
apiVersion: kind.x-k8s.io/v1alpha4
kind: Cluster
name: dev
nodes:
- role: control-plane
  kubeadmConfigPatches:
  - |
    kind: InitConfiguration
    nodeRegistration:
      kubeletExtraArgs:
        node-labels: "ingress-ready=true"
  extraPortMappings:
  - containerPort: 80
    hostPort: 80
    protocol: TCP
  - containerPort: 443
    hostPort: 443
    protocol: TCP
- role: worker
- role: worker
$ kind create cluster --config kind.yml 
Creating cluster "dev" ...
 ✓ Ensuring node image (kindest/node:v1.26.3) 🖼
 ✓ Preparing nodes 📦 📦 📦  
 ✓ Writing configuration 📜 
 ✓ Starting control-plane 🕹️ 
 ✓ Installing CNI 🔌 
 ✓ Installing StorageClass 💾 
 ✓ Joining worker nodes 🚜 
Set kubectl context to "kind-dev"
You can now use your cluster with:

kubectl cluster-info --context kind-dev
Deploy Nginx ingress to our cluster using the manifest file.
This manifest contains kind specific patches to forward the hostPorts to the ingress controller, set taint tolerations and schedule it to the custom labelled node.

$ kubectl apply -f https://raw.githubusercontent.com/kubernetes/ingress-nginx/main/deploy/static/provider/kind/deploy.yaml
namespace/ingress-nginx created
serviceaccount/ingress-nginx created
serviceaccount/ingress-nginx-admission created
role.rbac.authorization.k8s.io/ingress-nginx created
role.rbac.authorization.k8s.io/ingress-nginx-admission created
clusterrole.rbac.authorization.k8s.io/ingress-nginx created
clusterrole.rbac.authorization.k8s.io/ingress-nginx-admission created
rolebinding.rbac.authorization.k8s.io/ingress-nginx created
rolebinding.rbac.authorization.k8s.io/ingress-nginx-admission created
clusterrolebinding.rbac.authorization.k8s.io/ingress-nginx created
clusterrolebinding.rbac.authorization.k8s.io/ingress-nginx-admission created
configmap/ingress-nginx-controller created
service/ingress-nginx-controller created
service/ingress-nginx-controller-admission created
deployment.apps/ingress-nginx-controller created
job.batch/ingress-nginx-admission-create created
job.batch/ingress-nginx-admission-patch created
ingressclass.networking.k8s.io/nginx created
validatingwebhookconfiguration.admissionregistration.k8s.io/ingress-nginx-admission created
Now verify our ingress controller pod is up and running

$ kubectl -n ingress-nginx get pods --selector=app.kubernetes.io/component=controller
NAME                                        READY   STATUS    RESTARTS   AGE
ingress-nginx-controller-6bdf7bdbdd-gbdpc   1/1     Running   0          79s
Create Nginx pod using the below manifest file and verify it’s status

$ cat nginx.yml 
apiVersion: v1
kind: Pod
metadata:
  labels:
    run: nginx
  name: nginx
spec:
  containers:
  - image: nginx
    name: nginx
    ports:
    - containerPort: 80
$ kubectl apply -f nginx.yml 
pod/nginx created
$ kubectl get pods nginx
NAME    READY   STATUS    RESTARTS   AGE
nginx   1/1     Running   0          20s
Expose our Nginx pod to as ClusterIP service using the below manifest file

$ cat nginx-clusterip.yml 
apiVersion: v1
kind: Service
metadata:
  labels:
    run: nginx
  name: nginx
spec:
  ports:
  - port: 80
    protocol: TCP
    targetPort: 80
  selector:
    run: nginx
  type: ClusterIP
$ kubectl apply -f nginx-clusterip.yml 
service/nginx created
$ kubectl get svc nginx 
NAME    TYPE        CLUSTER-IP     EXTERNAL-IP   PORT(S)   AGE
nginx   ClusterIP   10.96.16.194   <none>        80/TCP    20s
Now create an Ingress resource using the below manifest file

$ cat nginx-ingress.yml 
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: nginx
  annotations:
    nginx.ingress.kubernetes.io/rewrite-target: /$2
spec:
  ingressClassName: nginx
  rules:
  - http:
      paths:
      - pathType: Prefix
        path: /my-app
        backend:
          service:
            name: nginx
            port:
              number: 80
$ kubectl apply -f nginx-ingress.yml 
ingress.networking.k8s.io/nginx created
$ kubectl get ingress nginx
NAME    CLASS   HOSTS   ADDRESS   PORTS   AGE
nginx   nginx   *                 80      21s
Access our application from our local machine and it worked

$ curl http://localhost/my-app
<!DOCTYPE html>
<html>
<head>
<title>Welcome to nginx!</title>
<style>
html { color-scheme: light dark; }
body { width: 35em; margin: 0 auto;
font-family: Tahoma, Verdana, Arial, sans-serif; }
</style>
</head>
<body>
<h1>Welcome to nginx!</h1>
<p>If you see this page, the nginx web server is successfully installed and
working. Further configuration is required.</p>

<p>For online documentation and support please refer to
<a href="http://nginx.org/">nginx.org</a>.<br/>
Commercial support is available at
<a href="http://nginx.com/">nginx.com</a>.</p>

<p><em>Thank you for using nginx.</em></p>
</body>
</html>
Cleanup
Delete the cluster after use

$ kind delete cluster --name dev
Deleting cluster "dev" ...
Deleted nodes: ["dev-worker" "dev-worker2" "dev-control-plane"]
