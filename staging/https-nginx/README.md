
# Nginx https service

This example creates a basic nginx https service useful in verifying proof of concept, keys, secrets, configmap, and end-to-end https service creation in kubernetes.
It uses an [nginx server block](http://wiki.nginx.org/ServerBlockExample) to serve the index page over both http and https. It will detect changes to nginx's configuration file, default.conf, mounted as a configmap volume and reload nginx automatically.

### Generate certificates

First generate a self signed rsa key and certificate that the server can use for TLS.

```sh
$ openssl req -x509 -nodes -days 365 -newkey rsa:2048 -keyout /d/tmp/nginx.key -out /d/tmp/nginx.crt -subj "/CN=my-nginx/O=my-nginx"
```

### Create a https nginx application running in a kubernetes cluster

You need a [running kubernetes cluster](https://kubernetes.io/docs/setup/pick-right-solution/) for this to work.

Create a secret and a configmap.

```sh
$ kubectl create secret tls nginxsecret --key /tmp/nginx.key --cert /tmp/nginx.crt
secret "nginxsecret" created

$ kubectl create configmap nginxconfigmap --from-file=examples/https-nginx/default.conf
configmap "nginxconfigmap" created
```

Create a service and a replication controller using the configuration in nginx-app.yaml.

```sh
$ kubectl create -f examples/https-nginx/nginx-app.yaml
You have exposed your service on an external port on all nodes in your
cluster.  If you want to expose this service to the external internet, you may
need to set up firewall rules for the service port(s) (tcp:32211,tcp:30028) to serve traffic.
...
service "nginxsvc" created
replicationcontroller "my-nginx" created
```

Then, find the node port that Kubernetes is using for http and https traffic.

```sh
$ kubectl get service nginxsvc 

NAME       TYPE       CLUSTER-IP    EXTERNAL-IP   PORT(S)                      AGE
nginxsvc   NodePort   10.11.251.5   <none>        80:30308/TCP,443:30860/TCP   2h
```

Then in another pod, you can access it with:
```sh
[lita]$ k exec productpage-v1-579987d6b9-qmf6f -c istio-proxy -- curl https://10.11.251.5:443 -k
```

Then you can inject side-car to the app by 
```sh
k delete -f nginx-app.yaml
```
And then
```sh
kubectl apply -f <(bin/istioctl kube-inject --debug -f nginx-app.yaml)
```
For more info, please refer to the doc in original repo


