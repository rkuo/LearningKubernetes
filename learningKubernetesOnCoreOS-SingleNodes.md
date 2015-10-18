# Learning Kubernetes on CoreOS - Single Nodes
refer to [Kubernetes Installation with Vagrant & CoreOS][1] for instructions; I copied some notes from the page. 

[1]:https://coreos.com/kubernetes/docs/latest/kubernetes-on-vagrant-single.html

## setup client

* download the binary using a command-line tool such as wget or curl from 
`https://storage.googleapis.com/kubernetes-release/release/v1.0.6/bin/${ARCH}/amd64/kubectl`. Read following instructions first;
* Set the ARCH environment variable to "linux" or "darwin" based on your workstation operating system:

```
ARCH=linux; wget https://storage.googleapis.com/kubernetes-release/release/v1.0.6/bin/$ARCH/amd64/kubectl
```

* After downloading the binary, ensure it is executable and move it into your PATH:

```
$ chmod +x kubectl
$ mv kubectl /usr/local/bin/kubectl
```


## setup kubernetes

#### clone

```
$ git clone https://github.com/coreos/coreos-kubernetes.git
$ cd coreos-kubernetes/single-node/
```

Initial structure;

```
➜  coreos-vagrant git:(master) ✗ cd ../coreos-kubernetes/single-node
➜  single-node git:(master) t
.
├── README.md
├── Vagrantfile
├── init-ssl
├── ssl
│   ├── admin-key.pem
│   ├── admin.csr
│   ├── admin.pem
│   ├── apiserver-key.pem
│   ├── apiserver-req.cnf
│   ├── apiserver.csr
│   ├── apiserver.pem
│   ├── ca-key.pem
│   ├── ca.pem
│   ├── ca.srl
│   └── controller.tar
└── user-data

1 directory, 15 files
➜  single-node git:(master)
```

#### start VM

`vagrant up`

#### config kubectl

* in `coreos-kubernetes/single-node/` directory, configure local Kubernetes client:

```
$ kubectl config set-cluster vagrant --server=https://172.17.4.99:443 --certificate-authority=${PWD}/ssl/ca.pem
$ kubectl config set-credentials vagrant-admin --certificate-authority=${PWD}/ssl/ca.pem --client-key=${PWD}/ssl/admin-key.pem --client-certificate=${PWD}/ssl/admin.pem
$ kubectl config set-context vagrant --cluster=vagrant --user=vagrant-admin
$ kubectl config use-context vagrant
```

* verify client installation by inspecting cluster:

```
$ kubectl get nodes
NAME          LABELS                               STATUS
172.17.4.99   kubernetes.io/hostname=172.17.4.99   Ready
```

* poking around

```
➜  single-node git:(master) pwd
/Users/rkuo/code/coreos-kubernetes/single-node
➜  single-node git:(master) vagrant ssh
Last login: Tue Oct 13 14:01:53 2015 from 10.0.2.2
CoreOS alpha (829.0.0)
Update Strategy: No Reboots
Failed Units: 1
  update-engine.service
core@localhost ~ $ fleetctl -h
NAME:
	fleetctl - fleetctl is a command-line interface to fleet, the cluster-wide CoreOS init system.

USAGE:
	fleetctl [global options] <command> [command options] [arguments...]

VERSION:
	0.11.5

COMMANDS:
	cat		Output the contents of a submitted unit
	destroy		Destroy one or more units in the cluster

<snip>

	--strict-host-key-checking=true			Verify host keys presented by remote machines before initiating SSH connections.
	--tunnel=					Establish an SSH tunnel through the provided address for communication with fleet and etcd.
	--version=false					Print the version and exit

Global options can also be configured via upper-case environment variables prefixed with "FLEETCTL_"
For example, "some-flag" => "FLEETCTL_SOME_FLAG"

Run "fleetctl help <command>" for more details on a specific command.
```

try systemctl, works too.

```
Run "fleetctl help <command>" for more details on a specific command.
core@localhost ~ $ systemctl -h
systemctl [OPTIONS...] {COMMAND} ...

Query or send control commands to the systemd manager.

  -h --help           Show this help
     --version        Show package version
     --system         Connect to system manager
     --user           Connect to user service manager
  -H --host=[USER@]HOST
                      Operate on remote host
<snip>

```

Check containers;

```
core@localhost ~ $ docker ps
CONTAINER ID        IMAGE                                            COMMAND                  CREATED             STATUS              PORTS               NAMES
0348feab7cd5        gcr.io/google_containers/skydns:2015-03-11-001   "/skydns -machines=ht"   21 hours ago        Up 21 hours                             k8s_skydns.9e669ab2_kube-dns-v9-umcux_kube-system_f4d4fd72-7112-11e5-867b-080027ebc558_2715d48c
1524c2b974a8        gcr.io/google_containers/exechealthz:1.0         "/exechealthz '-cmd=n"   21 hours ago        Up 21 hours                             k8s_healthz.9b2da2a1_kube-dns-v9-umcux_kube-system_f4d4fd72-7112-11e5-867b-080027ebc558_e72e86a7
bac218c3845e        gcr.io/google_containers/kube2sky:1.11           "/kube2sky -domain=cl"   21 hours ago        Up 21 hours                             k8s_kube2sky.71be6a8b_kube-dns-v9-umcux_kube-system_f4d4fd72-7112-11e5-867b-080027ebc558_2dcc31ab
0f409b98b45b        gcr.io/google_containers/etcd:2.0.9              "/usr/local/bin/etcd "   21 hours ago        Up 21 hours                             k8s_etcd.8508bf43_kube-dns-v9-umcux_kube-system_f4d4fd72-7112-11e5-867b-080027ebc558_90a567f1
115810d32dcd        gcr.io/google_containers/pause:0.8.0             "/pause"                 21 hours ago        Up 21 hours                             k8s_POD.2688308a_kube-dns-v9-umcux_kube-system_f4d4fd72-7112-11e5-867b-080027ebc558_7a7f7c65
6ba914ebf57b        gcr.io/google_containers/hyperkube:v1.0.3        "/hyperkube controlle"   21 hours ago        Up 21 hours                             k8s_kube-controller-manager.fdd2caeb_kube-controller-manager-172.17.4.99_kube-system_674d84509c6a72ed41954a52f707cd35_89efa71f
70bf826acac0        gcr.io/google_containers/hyperkube:v1.0.3        "/hyperkube proxy --m"   21 hours ago        Up 21 hours                             k8s_kube-proxy.d2ac3080_kube-proxy-172.17.4.99_kube-system_efdb0ab67e30a0c62af95683d95280cd_dd389aaf
afb26262c0ec        gcr.io/google_containers/hyperkube:v1.0.3        "/hyperkube apiserver"   21 hours ago        Up 21 hours                             k8s_kube-apiserver.9da81f55_kube-apiserver-172.17.4.99_kube-system_18b0d3030f9284170b64191baf861f49_08aa7a58
1a0de24fcdeb        gcr.io/google_containers/hyperkube:v1.0.3        "/hyperkube scheduler"   21 hours ago        Up 21 hours                             k8s_kube-scheduler.ff1358dd_kube-scheduler-172.17.4.99_kube-system_e247c9f3cab6552495bb59e2a2143b4e_6ba342ac
63055152ca84        gcr.io/google_containers/pause:0.8.0             "/pause"                 21 hours ago        Up 21 hours                             k8s_POD.e4cc795_kube-scheduler-172.17.4.99_kube-system_e247c9f3cab6552495bb59e2a2143b4e_77dcafe9
<snip>
```
There are some containers are running: skydns, exechealthz, kube2sky, etcd, hyperkube controller, hyperkube proxxy, hyperkub apiserver, hyperkube scheduler, pause, ..

This matches kubernetes client side.

Let's check the server side; we found kubelet.

```
core@localhost /usr/bin $ ls -l k*
-rwxr-xr-x 1 root root   129504 Oct  8 07:18 kbxutil
-rwxr-xr-x 1 root root     3170 Oct  8 07:52 kernel-install
-rwxr-xr-x 1 root root    34768 Oct  8 07:35 keyctl
-rwxr-xr-x 1 root root    22480 Oct  8 07:34 kill
-rwxr-xr-x 1 root root   161848 Oct  8 07:13 kmod
-rwxr-xr-x 1 root root     2590 Oct  8 07:10 ksba-config
-rwxr-xr-x 1 root root 23826200 Oct  8 07:41 kubelet
core@localhost /usr/bin $
```

Refer to [Kelsey's tutorial][3], 
[3]:https://github.com/kelseyhightower/coreos-ops-tutorial

# load Guestbook Example

Refer to 
[2]:http://kubernetes.io/v1.0/examples/guestbook-go/README.html

```
➜  guestbook git:(master) ✗ t
.
├── README.md
├── frontend-controller.yaml
├── frontend-service.yaml
├── php-redis
│   ├── Dockerfile
│   ├── controllers.js
│   ├── guestbook.php
│   └── index.html
├── redis-master-controller.yaml
├── redis-master-service.yaml
├── redis-slave
│   ├── Dockerfile
│   └── run.sh
├── redis-slave-controller.yaml
└── redis-slave-service.yaml

2 directories, 13 files
```

## 0 - kubectl command

```
kubectl api-versions - Print available API versions.
kubectl cluster-info - Display cluster info
kubectl config - config modifies kubeconfig files
kubectl create - Create a resource by filename or stdin
kubectl delete - Delete a resource by filename, stdin, resource and name, or by resources and label selector.
kubectl describe - Show details of a specific resource or group of resources
kubectl exec - Execute a command in a container.
kubectl expose - Take a replicated application and expose it as Kubernetes Service
kubectl get - Display one or many resources
kubectl label - Update the labels on a resource
kubectl logs - Print the logs for a container in a pod.
kubectl namespace - SUPERCEDED: Set and view the current Kubernetes namespace
kubectl patch - Update field(s) of a resource by stdin.
kubectl port-forward - Forward one or more local ports to a pod.
kubectl proxy - Run a proxy to the Kubernetes API server
kubectl replace - Replace a resource by filename or stdin.
kubectl rolling-update - Perform a rolling update of the given ReplicationController.
kubectl run - Run a particular image on the cluster.
kubectl scale - Set a new size for a Replication Controller.
kubectl stop - Gracefully shut down a resource by name or filename.
kubectl version - Print the client and server version information.
```

## 1 - create the Redis master pod 

It is easier to create a pod with replicate controller; so rather than creating pod directly, we will setup replicate controller first.

#### create

```
kubectl create -f examples/guestbook-go/redis-master-controller.json
```   

This is what we want to create.

```
➜  guestbook git:(master) ✗ cat redis-master-controller.yaml
apiVersion: v1
kind: ReplicationController
metadata:
  name: redis-master
  labels:
    name: redis-master
spec:
  replicas: 1
  selector:
    name: redis-master
  template:
    metadata:
      labels:
        name: redis-master
    spec:
      containers:
      - name: master
        image: redis
        ports:
        - containerPort: 6379
```

#### verify master-controller 

Verify the appX-master-controller is up, ok, it works. appX=redis

```
➜  single-node git:(master) ✗ kubectl get rc
CONTROLLER     CONTAINER(S)   IMAGE(S)   SELECTOR                REPLICAS
redis-master   redis-master   redis      app=redis,role=master   1
```
#### verify master-pod

Verify the pod is up, it is good, we have a pod.

```
➜  guestbook git:(master) ✗ kubectl get pods
NAME                 READY     STATUS    RESTARTS   AGE
redis-master-2i4pi   1/1       Running   0          1h
➜  guestbook git:(master) ✗
```

#### verify container is running.  

```
core@localhost /usr/bin $ docker ps
CONTAINER ID        IMAGE                                            COMMAND                  CREATED             STATUS              PORTS               NAMES
d5179d5ca112        redis                                            "/entrypoint.sh redis"   3 hours ago         Up 3 hours                              k8s_redis-master.d5991e3c_redis-mast
```

## 2 - create the Redis master service

* A Kubernetes 'service' is a named load balancer that proxies traffic to one or more containers. 
* The services in a Kubernetes cluster are discoverable inside other containers via environment variables or DNS.
* Services find the containers to load balance based on pod labels. The pod that you created in Step One has the label app=redis and role=master. The selector field of the service determines which pods will receive the traffic sent to the service.

#### create master-service 

* Use the redis-master-service.json file to create the service in your Kubernetes cluster by running the `kubectl create -f <filename>` command:  

```
$ kubectl create -f examples/guestbook-go/redis-master-service.json
services/redis-master
```

Service definition:

```
➜  guestbook git:(master) ✗ cat redis-master-service.yaml
apiVersion: v1
kind: Service
metadata:
  name: redis-master
  labels:
    name: redis-master
spec:
  ports:
    # the port that this service should serve on
  - port: 6379
    targetPort: 6379
  selector:
    name: redis-master
```

#### verify master-service

```
➜  guestbook git:(master) ✗ kubectl get services
NAME           LABELS                                    SELECTOR                IP(S)        PORT(S)
kubernetes     component=apiserver,provider=kubernetes   <none>                  10.3.0.1     443/TCP
redis-master   app=redis,role=master                     app=redis,role=master   10.3.0.204   6379/TCP
```

Result: All new pods will see the `redis-master` service running on the host (`$REDIS_MASTER_SERVICE_HOST` environment variable) at port 6379, or running on `redis-master:6379`. After the service is created, the service proxy on each node is configured to set up a proxy on the specified port (in our example, that's port 6379). 

## 3 - create the Redis slave pods

In Kubernetes, a replication controller is responsible for managing the multiple instances of a replicated pod.

#### create slave-pod

Use the file `redis-slave-controller.json` to create the replication controller by running the `kubectl create -f <filename>` command:  

```
kubectl create -f examples/guestbook-go/redis-slave-controller.json
    replicationcontrollers/redis-slave
```  

#### verify slave-pod

To verify that the guestbook replication controller is running, run the `kubectl get rc`:

```
➜  examples git:(master) ✗ kubectl get rc
CONTROLLER     CONTAINER(S)   IMAGE(S)                    SELECTOR                REPLICAS
redis-master   redis-master   redis                       app=redis,role=master   1
redis-slave    redis-slave    kubernetes/redis-slave:v2   app=redis,role=slave    2
➜  examples git:(master) ✗
```

Result: The replication controller creates and configures the Redis slave pods through the redis-master service (name:port pair, in our example that's redis-master:6379).

```
➜  examples git:(master) ✗ cat guestbook-go/redis-slave-controller.json
{
   "kind":"ReplicationController",
   "apiVersion":"v1",
   "id":"redis-slave",
   "metadata":{
      "name":"redis-slave",
      "labels":{
         "app":"redis",
         "role":"slave"
      }
   },
   "spec":{
      "replicas":2,
      "selector":{
         "app":"redis",
         "role":"slave"
      },
      "template":{
         "metadata":{
            "labels":{
               "app":"redis",
               "role":"slave"
            }
         },
         "spec":{
            "containers":[
               {
                  "name":"redis-slave",
                  "image":"kubernetes/redis-slave:v2",
                  "ports":[
                     {
                        "name":"redis-server",
                        "containerPort":6379
                     }
                  ]
               }
            ]
         }
      }
   }
}
➜  examples git:(master) ✗
```

To start replication by running master:

```
redis-server --slaveof redis-master 6379
```
```
➜  examples git:(master) ✗ kubectl get pods
NAME                 READY     STATUS    RESTARTS   AGE
redis-master-2i4pi   1/1       Running   0          6h
redis-slave-255hl    1/1       Running   0          53m
redis-slave-ycm6g    1/1       Running   0          53m
➜  examples git:(master) ✗
```

##  4 - create the Redis slave service

We want to have a service to proxy connections to the read slaves. In this case, in addition to discovery, the Redis slave service provides transparent load balancing to clients.

#### Create slave service

Use the `redis-slave-service.json` file to create the Redis slave service by running the `kubectl create -f filename` command:

```
➜  examples git:(master) ✗ kubectl get pods
NAME                 READY     STATUS    RESTARTS   AGE
redis-master-2i4pi   1/1       Running   0          6h
redis-slave-255hl    1/1       Running   0          53m
redis-slave-ycm6g    1/1       Running   0          53m
➜  examples git:(master) ✗ kubectl create -f guestbook-go/redis-slave-service.json
services/redis-slave
```
#### verify slave service

```
➜  examples git:(master) ✗ kubectl get services
NAME           LABELS                                    SELECTOR                IP(S)        PORT(S)
kubernetes     component=apiserver,provider=kubernetes   <none>                  10.3.0.1     443/TCP
redis-master   app=redis,role=master                     app=redis,role=master   10.3.0.204   6379/TCP
redis-slave    app=redis,role=slave                      app=redis,role=slave    10.3.0.70    6379/TCP
➜  examples git:(master) ✗
```
Service has IP Address.

#### 5 - create the guestbook controller
This is the actual app (aggregated).

```
➜  examples git:(master) ✗ kubectl create -f guestbook-go/guestbook-controller.json
replicationcontrollers/guestbook
➜  examples git:(master) ✗
```

App spec:

```
➜  examples git:(master) ✗ cat guestbook-go/guestbook-controller.json
{
   "kind":"ReplicationController",
   "apiVersion":"v1",
   "metadata":{
      "name":"guestbook",
      "labels":{
         "app":"guestbook"
      }
   },
   "spec":{
      "replicas":3,
      "selector":{
         "app":"guestbook"
      },
      "template":{
         "metadata":{
            "labels":{
               "app":"guestbook"
            }
         },
         "spec":{
            "containers":[
               {
                  "name":"guestbook",
                  "image":"kubernetes/guestbook:v2",
                  "ports":[
                     {
                        "name":"http-server",
                        "containerPort":3000
                     }
                  ]
               }
            ]
         }
      }
   }
}
➜  examples git:(master) ✗
```
#### verify guestbook controller is running

```
➜  examples git:(master) ✗ kubectl get rc
CONTROLLER     CONTAINER(S)   IMAGE(S)                    SELECTOR                REPLICAS
guestbook      guestbook      kubernetes/guestbook:v2     app=guestbook           3
redis-master   redis-master   redis                       app=redis,role=master   1
redis-slave    redis-slave    kubernetes/redis-slave:v2   app=redis,role=slave    2
➜  examples git:(master) ✗
```

#### verify guestbook pod is running

```
➜  examples git:(master) ✗ kubectl get pods
NAME                 READY     STATUS         RESTARTS   AGE
guestbook-6g8lj      0/1       GeneralError   0          14m
guestbook-9stx7      0/1       GeneralError   0          14m
guestbook-yala0      0/1       GeneralError   0          14m
redis-master-2i4pi   1/1       Running        0          8h
redis-slave-255hl    1/1       Running        0          2h
redis-slave-ycm6g    1/1       Running        0          2h
➜  examples git:(master) ✗
```

**There is an error here.** remove them and rebuild.

```
➜  examples git:(master) ✗ kubectl delete guestbook-6g8lj
error: you must provide one or more resources by argument or filename
➜  examples git:(master) ✗ kubectl delete pods guestbook-6g8lj
pods/guestbook-6g8lj
➜  examples git:(master) ✗ kubectl delete pods guestbook-9stx7
pods/guestbook-9stx7
➜  examples git:(master) ✗ kubectl delete pods guestbook-yala0
pods/guestbook-yala0
➜  examples git:(master) ✗ kubectl get rc
CONTROLLER     CONTAINER(S)   IMAGE(S)                    SELECTOR                REPLICAS
guestbook      guestbook      kubernetes/guestbook:v2     app=guestbook           3
redis-master   redis-master   redis                       app=redis,role=master   1
redis-slave    redis-slave    kubernetes/redis-slave:v2   app=redis,role=slave    2
➜  examples git:(master) ✗ kubectl get pods
NAME                 READY     STATUS    RESTARTS   AGE
guestbook-1v7gl      1/1       Running   0          44s
guestbook-5i4hj      1/1       Running   0          21s
guestbook-h4d09      1/1       Running   0          1m
redis-master-2i4pi   1/1       Running   0          8h
redis-slave-255hl    1/1       Running   0          3h
redis-slave-ycm6g    1/1       Running   0          3h
➜  examples git:(master) ✗
```

#### 6 - create the guestbook service

Just like the others, we create a service to group the guestbook pods but this time, to make the guestbook front-end externally visible, we specify "type": "LoadBalancer".

* Use the `guestbook-service.json` file to create the guestbook service by running the `kubectl create -f filename` command:

```
kubectl create -f guestbook-go/guestbook-service.json
```
Service:

```
➜  guestbook-go git:(master) ✗ cat guestbook-service.json
{
   "kind":"Service",
   "apiVersion":"v1",
   "metadata":{
      "name":"guestbook",
      "labels":{
         "app":"guestbook"
      }
   },
   "spec":{
      "ports": [
         {
           "port":3000,
           "targetPort":"http-server"
         }
      ],
      "selector":{
         "app":"guestbook"
      },
      "type": "LoadBalancer"
   }
}
➜  guestbook-go git:(master) ✗
```

#### verify the guestbook service is up

* list all the services in the cluster with the kubectl get services command:

```
➜  examples git:(master) ✗ kubectl get services
NAME           LABELS                                    SELECTOR                IP(S)        PORT(S)
guestbook      app=guestbook                             app=guestbook           10.3.0.251   3000/TCP
kubernetes     component=apiserver,provider=kubernetes   <none>                  10.3.0.1     443/TCP
redis-master   app=redis,role=master                     app=redis,role=master   10.3.0.204   6379/TCP
redis-slave    app=redis,role=slave                      app=redis,role=slave    10.3.0.70    6379/TCP
➜  examples git:(master) ✗ kubectl describe services guestbook
Name:			guestbook
Namespace:		default
Labels:			app=guestbook
Selector:		app=guestbook
Type:			LoadBalancer
IP:			10.3.0.251
Port:			<unnamed>	3000/TCP
NodePort:		<unnamed>	31801/TCP
Endpoints:		10.2.52.10:3000,10.2.52.11:3000,10.2.52.9:3000
Session Affinity:	None
No events.

➜  examples git:(master) ✗
```

## 7 - view the guestbook

* For local: http://localhost:3000 in browser.
* For remote: http://<host-ip>:3000 in browser.

use `kubectl get services` to get service ip.

## 8 - clean up

`kubectl delete -f guestbook-go`







