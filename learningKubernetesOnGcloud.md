# Learning Kubernetes on gcloud

## install gcloud utility
Command Line

```
➜  kubernetes-gcloud  gcloud -h
Usage: gcloud [optional flags] <group | command>
  group may be           auth | components | compute | config | container |
                         deployment-manager | dns | source | sql | topic
  command may be         docker | feedback | help | info | init | version

The *gcloud* CLI manages authentication, local configuration, developer
workflow, and interactions with the Google Cloud Platform APIs.

optional flags:
  --account ACCOUNT      Google Cloud Platform user account to use for
                         invocation.
  --format FORMAT        The format for printing command output resources.
  --help                 Display detailed help.
  --log-http             Log all HTTP server requests and responses to stderr.
  --project PROJECT_ID   Google Cloud Platform project ID to use for this
                         invocation.
  --quiet, -q            Disable all interactive prompts.
  --trace-token TRACE_TOKEN
                         Token used to route traces of service requests for
                         investigation of issues.
  --user-output-enabled  Print user intended output to the console.
  --verbosity VERBOSITY  Override the default verbosity for this command.  This
                         must be a standard logging verbosity level: [debug,
                         info, warning, error, critical, none] (Default:
                         [warning]).
  -h                     Print a summary help and exit.
  -v, --version          Print version information.
  ```

## preparasion 

  * create and cd to project directroy.
  * `gcloud init`
  we need to enable the billing of the account.
  
  * signin account at [console][3] to ensure the project exist
  
  [3]: https://console.developers.google.com/project/trykubernetes-1 
  
## create a cluster

```
➜  kubernetes-gcloud  gcloud container clusters create guestbook
Creating cluster guestbook...done.
Created [https://container.googleapis.com/v1/projects/trykubernetes-1/zones/us-central1-f/clusters/guestbook].
kubeconfig entry generated for guestbook.
NAME       ZONE           MASTER_VERSION  MASTER_IP        MACHINE_TYPE   STATUS
guestbook  us-central1-f  1.0.6           104.197.132.101  n1-standard-1  RUNNING
➜  kubernetes-gcloud
```
get some information about the clusters in this project:

```
➜  kubernetes-gcloud  gcloud container clusters list  # list clusters in the current project
NAME       ZONE           MASTER_VERSION  MASTER_IP        MACHINE_TYPE   STATUS
cluster-1  us-central1-f  1.0.6           104.197.5.7      g1-small       RUNNING
guestbook  us-central1-f  1.0.6           104.197.132.101  n1-standard-1  RUNNING
➜  kubernetes-gcloud  gcloud container clusters delete cluster-1
The following clusters will be deleted.
 - [cluster-1] in [us-central1-f]

Do you want to continue (Y/n)?  y

Deleting cluster cluster-1...done.
Deleted [https://container.googleapis.com/v1/projects/trykubernetes-1/zones/us-central1-f/clusters/cluster-1].
➜  kubernetes-gcloud  gcloud container clusters list  # list clusters in the current project
NAME       ZONE           MASTER_VERSION  MASTER_IP        MACHINE_TYPE   STATUS
guestbook  us-central1-f  1.0.6           104.197.132.101  n1-standard-1  RUNNING
➜  kubernetes-gcloud
```

Info about single cluster:

```
➜  kubernetes-gcloud  gcloud container clusters describe guestbook  # info about single cluster
clusterIpv4Cidr: 10.188.0.0/14
createTime: '2015-10-14T18:13:41+00:00'
currentMasterVersion: 1.0.6
currentNodeVersion: 1.0.6
endpoint: 104.197.132.101
initialClusterVersion: 1.0.6
initialNodeCount: 3
instanceGroupUrls:
- https://www.googleapis.com/replicapool/v1beta2/projects/trykubernetes-1/zones/us-central1-f/instanceGroupManagers/gke-guestbook-7dc33b44-group
loggingService: logging.googleapis.com
masterAuth:
  clientCertificate: LS0tLS1CRUdJTiBDRVJUSUZJQ0FURS0tLS0tCk1JSURDakNDQWZLZ0F3SUJBZ0lRYVNBNHJzZWRWRUxKWHpuN3lFcUxCakFOQmdrcWhraUc5dzBCQVFzRkFEQTYKTVRnd05nWURWUVFEREM5MWN5MWpaVzUwY21Gc01TMW1MVFF4TmpJeE1qUTBNalF5TWkxbmRXVnpkR0p2YjJ0QQpNVFEwTkRnME5qUXlOVEFlRncweE5URXdNVFF4T0RFek5EWmFGdzB5TURFd01USXhPREV6TkRaYU1CRXhEekFOCkJnTlZ
...
```
## 1 - create Redis master controller

#### create 

We will use `redis-master-controller.json` to create replicate controller,


```
➜  kubernetes-gcloud  cat redis-master-controller.json # to see the content
{
   "kind":"ReplicationController",
   "apiVersion":"v1",
   "metadata":{
      "name":"redis-master",
      "labels":{
         "name":"redis-master"
      }
   },
   "spec":{
      "replicas":1,
      "selector":{
         "name":"redis-master"
      },
      "template":{
         "metadata":{
            "labels":{
               "name":"redis-master"
            }
         },
         "spec":{
            "containers":[
               {
                  "name":"master",
                  "image":"redis",
                  "ports":[
                     {
                        "containerPort":6379,
                        "protocol":"TCP"
                     }
                  ]
               }
            ]
         }
      }
   }
}
```

`kubectl create -f redis-master-controller.json`  

#### verify Redis master controller is running

```
➜  kubernetes-gcloud  kubectl get pods -l name=redis-master
NAME                 READY     STATUS    RESTARTS   AGE
redis-master-7oniw   0/1       Running   0          9s
➜  kubernetes-gcloud
```
#### pods info

```
➜  kubernetes-gcloud  kubectl get pods -l name=redis-master -o wide
NAME                 READY     STATUS    RESTARTS   AGE       NODE
redis-master-7oniw   1/1       Running   0          11m       gke-guestbook-7dc33b44-node-9tlv
```
login container

```
➜  kubernetes-gcloud  gcloud compute ssh gke-guestbook-7dc33b44-node-9tlv
WARNING: You do not have an SSH key for Google Compute Engine.
WARNING: [/usr/bin/ssh-keygen] will be executed to generate a key.
Generating public/private rsa key pair.
Enter passphrase (empty for no passphrase):
Enter same passphrase again:
Your identificationtion has been saved in /Users/rkuo/.ssh/google_compute_engine.
Your public key has been saved in /Users/rkuo/.ssh/google_compute_engine.pub.
The key fingerprint is:
...
Updated [https://www.googleapis.com/compute/v1/projects/trykubernetes-1].
Warning: Permanently added '108.59.85.122' (RSA) to the list of known hosts.
Warning: Permanently added '108.59.85.122' (RSA) to the list of known hosts.
Linux gke-guestbook-7dc33b44-node-9tlv 3.16.0-0.bpo.4-amd64 #1 SMP Debian 3.16.7-ckt11-1+deb8u2~bpo70+1 (2015-07-22) x86_64

=== GCE Kubernetes node setup complete ===

rkuo@gke-guestbook-7dc33b44-node-9tlv:~$
```
let's look around about our containers:

```
rkuo@gke-guestbook-7dc33b44-node-9tlv:~$ sudo docker ps
CONTAINER ID        IMAGE                                       COMMAND                CREATED             STATUS              PORTS               NAMES
97414c14630b        redis:latest                                "/entrypoint.sh redi   39 minutes ago      Up 39 minutes                           k8s_master.2e93469c_redis-master-7oniw_default_4efaebfe-72a6-11e5-a6f2-42010af000a8_19fbc616
c2a4977e7f95        gcr.io/google_containers/pause:0.8.0        "/pause"               40 minutes ago      Up 40 minutes                           k8s_POD.49eee8c2_redis-master-7oniw_default_4efaebfe-72a6-11e5-a6f2-42010af000a8_46614c8d
ed94f3bb13a8        gcr.io/google_containers/kube-ui:v1.1       "/kube-ui"             About an hour ago   Up About an hour                        k8s_kube-ui.add839b_kube-ui-v1-z3z9o_kube-system_7ac47ed1-729f-11e5-a6f2-42010af000a8_9bb45b3a
795ff3313359        gcr.io/google_containers/pause:0.8.0        "/pause"               About an hour ago   Up About an hour                        k8s_POD.3b46e8b9_kube-ui-v1-z3z9o_kube-system_7ac47ed1-729f-11e5-a6f2-42010af000a8_df730444
6127c73c419c        gcr.io/google_containers/fluentd-gcp:1.11   "\"/bin/sh -c '/usr/   About an hour ago   Up About an hour                        k8s_fluentd-cloud-logging.44219385_fluentd-cloud-logging-gke-guestbook-7dc33b44-node-9tlv_kube-system_b845047be3634f41e2061ca65fbaa9d2_8891802b
40502c8829d9        gcr.io/google_containers/pause:0.8.0        "/pause"               About an hour ago   Up About an hour                        k8s_POD.e4cc795_fluentd-cloud-logging-gke-guestbook-7dc33b44-node-9tlv_kube-system_b845047be3634f41e2061ca65fbaa9d2_0816f5cb
rkuo@gke-guestbook-7dc33b44-node-9tlv:~$
```
we can see `redis:latest` is there. The container id = 97414c14630b   
The image has fatched too.

```
rkuo@gke-guestbook-7dc33b44-node-9tlv:~$ sudo docker images
REPOSITORY                             TAG                 IMAGE ID            CREATED             VIRTUAL SIZE
redis                                  latest              2f2578ff984f        4 weeks ago         109.2 MB
gcr.io/google_containers/fluentd-gcp   1.11                b6e57ec0903b        10 weeks ago        410.2 MB
gcr.io/google_containers/kube-ui       v1.1                c144d89158dc        3 months ago        5.83 MB
gcr.io/google_containers/pause         0.8.0               2c40b0526b63        6 months ago        241.7 kB
rkuo@gke-guestbook-7dc33b44-node-9tlv:~$
```
We can check the log.  

```
sudo docker logs 97414c14630b
```

```
rkuo@gke-guestbook-7dc33b44-node-9tlv:~$ sudo docker logs 97414c14630b
1:C 14 Oct 19:03:59.463 # Warning: no config file specified, using the default config. In order to specify a config file use redis-server /path/to/redis.conf
                _._
           _.-``__ ''-._
      _.-``    `.  `_.  ''-._           Redis 3.0.3 (00000000/0) 64 bit
  .-`` .-```.  ```\/    _.,_ ''-._
 (    '      ,       .-`  | `,    )     Running in standalone mode
 |`-._`-...-` __...-.``-._|'` _.-'|     Port: 6379
 |    `-._   `._    /     _.-'    |     PID: 1
  `-._    `-._  `-./  _.-'    _.-'
 |`-._`-._    `-.__.-'    _.-'_.-'|
 |    `-._`-._        _.-'_.-'    |           http://redis.io
  `-._    `-._`-.__.-'_.-'    _.-'
 |`-._`-._    `-.__.-'    _.-'_.-'|
 |    `-._`-._        _.-'_.-'    |
  `-._    `-._`-.__.-'_.-'    _.-'
      `-._    `-.__.-'    _.-'
          `-._        _.-'
              `-.__.-'

1:M 14 Oct 19:03:59.464 # Server started, Redis version 3.0.3
1:M 14 Oct 19:03:59.465 # WARNING overcommit_memory is set to 0! Background save may fail under low memory condition. To fix this issue add 'vm.overcommit_memory = 1' to /etc/sysctl.conf and then reboot or run the command 'sysctl vm.overcommit_memory=1' for this to take effect.
1:M 14 Oct 19:03:59.465 # WARNING you have Transparent Huge Pages (THP) support enabled in your kernel. This will create latency and memory usage issues with Redis. To fix this issue run the command 'echo never > /sys/kernel/mm/transparent_hugepage/enabled' as root, and add it to your /etc/rc.local in order to retain the setting after a reboot. Redis must be restarted after THP is disabled.
1:M 14 Oct 19:03:59.465 # WARNING: The TCP backlog setting of 511 cannot be enforced because /proc/sys/net/core/somaxconn is set to the lower value of 128.
1:M 14 Oct 19:03:59.466 * The server is now ready to accept connections on port 6379
rkuo@gke-guestbook-7dc33b44-node-9tlv:~$
```
type `exit` to get out container.

## 2 - Start the Redis master's service

copied:
A service is an abstraction which defines a logical set of pods and a policy by which to access them. It is effectively a named load balancer that proxies traffic to one or more pods.

When you set up a service, you tell it the pods to proxy based on pod labels. Note that the pod that you created in step one has the label name=redis-master.


```
➜  kubernetes-gcloud  cat redis-master-service.json
{
   "kind":"Service",
   "apiVersion":"v1",
   "metadata":{
      "name":"redis-master",
      "labels":{
         "name":"redis-master" ## we will use/select pods with this label to construct our service
      }
   },
   "spec":{
      "ports": [
        {
          "port":6379,
          "targetPort":6379,
          "protocol":"TCP"
        }
      ],
      "selector":{
         "name":"redis-master" ## this container receives message received by service
      }
   }
}
```

copied:
The selector field of the service configuration determines which pods will receive the traffic sent to the service. So, the configuration is specifying that we want this service to point to pods labeled with name=redis-master.

`kubectl create -f redis-master-service.json`  

```
➜  kubernetes-gcloud  kubectl get services
NAME           LABELS                                    SELECTOR            IP(S)            PORT(S)
kubernetes     component=apiserver,provider=kubernetes   <none>              10.191.240.1     443/TCP
redis-master   name=redis-master                         name=redis-master   10.191.249.193   6379/TCP
```

## 3 - Start the replicated Redis worker pods

`kubectl create -f redis-worker-controller.json`  

```
➜  kubernetes-gcloud  kubectl create -f redis-worker-controller.json
replicationcontrollers/redis-slave
➜  kubernetes-gcloud  kubectl get rc
CONTROLLER     CONTAINER(S)   IMAGE(S)                    SELECTOR            REPLICAS
redis-master   master         redis                       name=redis-master   1
redis-slave    slave          kubernetes/redis-slave:v2   name=redis-slave    2
➜  kubernetes-gcloud
```
we have 3 pods in running state.

```
➜  kubernetes-gcloud  kubectl get pods
NAME                 READY     STATUS    RESTARTS   AGE
redis-master-7oniw   1/1       Running   0          5h
redis-slave-aa1s2    1/1       Running   0          1h
redis-slave-dv0x4    1/1       Running   0          1h
```

## 4 - create the Redis worker service

use `redis-worker-service.json:`  to create worker service  
Other than kubernetes itself, we have 2 new services.

```
➜  kubernetes-gcloud  kubectl create -f redis-worker-service.json
services/redis-slave
➜  kubernetes-gcloud  kubectl get services
NAME           LABELS                                    SELECTOR            IP(S)            PORT(S)
kubernetes     component=apiserver,provider=kubernetes   <none>              10.191.240.1     443/TCP
redis-master   name=redis-master                         name=redis-master   10.191.249.193   6379/TCP
redis-slave    name=redis-slave                          name=redis-slave    10.191.249.83    6379/TCP
```

## 5 - create the guestbook web server pods
Now we have backend services running, we will create front end web services.
It is named `frontend`, 3 copies.
This exercise uses a PHP file as frontend services.   

```
➜  kubernetes-gcloud  cat frontend-controller.json
{
   "kind":"ReplicationController",
   "apiVersion":"v1",
   "metadata":{
      "name":"frontend",
      "labels":{
         "name":"frontend"
      }
   },
   "spec":{
      "replicas":3,
      "selector":{
         "name":"frontend"
      },
      "template":{
         "metadata":{
            "labels":{
               "name":"frontend"
            }
         },
         "spec":{
            "containers":[
               {
                  "name":"php-redis",
                  "image":"kubernetes/example-guestbook-php-redis:v2",
                  "ports":[
                     {
                        "containerPort":80,
                        "protocol":"TCP"
                     }
                  ]
               }
            ]
         }
      }
   }
}
➜  kubernetes-gcloud
```
#### create frontend controller 
which will generate frontend pods also.

```
➜  kubernetes-gcloud  kubectl create -f frontend-controller.json
replicationcontrollers/frontend
➜  kubernetes-gcloud  kubectl get rc
CONTROLLER     CONTAINER(S)   IMAGE(S)                                    SELECTOR            REPLICAS
frontend       php-redis      kubernetes/example-guestbook-php-redis:v2   name=frontend       3
redis-master   master         redis                                       name=redis-master   1
redis-slave    slave          kubernetes/redis-slave:v2                   name=redis-slave    2
➜  kubernetes-gcloud
```
let's find out number of pods we have:

```
➜  kubernetes-gcloud  kubectl get pods
NAME                 READY     STATUS    RESTARTS   AGE
frontend-coi93       1/1       Running   0          6m
frontend-jhcrg       1/1       Running   0          6m
frontend-mw7q5       1/1       Running   0          6m
redis-master-7oniw   1/1       Running   0          6h
redis-slave-aa1s2    1/1       Running   0          2h
redis-slave-dv0x4    1/1       Running   0          2h
➜  kubernetes-gcloud
```
We have 3 frontend pods and 3 backend pods (2 workers and 1 master). The container image is specified in replicate controller. In this case, it is php app which is repo at `kubernetes/example-guestbook-php-redis:v2`. We can find it at [dockerhub][1]


[1]:https://hub.docker.com/search/?q=kubernetes%2Fexample-guestbook-php-redis&page=1&isAutomated=0&isOfficial=0&starCount=0&pullCount=0 


```
<?

set_include_path('.:/usr/share/php:/usr/share/pear:/vendor/predis');

error_reporting(E_ALL);
ini_set('display_errors', 1);

require 'predis/autoload.php';

if (isset($_GET['cmd']) === true) {
  header('Content-Type: application/json');
  if ($_GET['cmd'] == 'set') {
    $client = new Predis\Client([
      'scheme' => 'tcp',
      'host'   => 'redis-master',
      'port'   => 6379,
    ]);

    $client->set($_GET['key'], $_GET['value']);
    print('{"message": "Updated"}');
  } else {
    $client = new Predis\Client([
      'scheme' => 'tcp',
      'host'   => 'redis-slave',
      'port'   => 6379,
    ]);

    $value = $client->get($_GET['key']);
    print('{"data": "' . $value . '"}');
  }
} else {
  phpinfo();
} ?>
```

## 6 - create a guestbook web service with an external IP

* connect service to pod (php container) by its label/tag=frontend,
* this web service has role of load balance,
* assign the port 80.
*

```
➜  kubernetes-gcloud  cat frontend-service.json
{
   "kind":"Service",
   "apiVersion":"v1",
   "metadata":{
      "name":"frontend",
      "labels":{
         "name":"frontend"
      }
   },
   "spec":{
      "type": "LoadBalancer",
      "ports": [
        {
          "port":80,
          "targetPort":80,
          "protocol":"TCP"
        }
      ],
      "selector":{
         "name":"frontend"
      }
   }
}
```
We started the frontend web service:  

```
services/frontend
➜  kubernetes-gcloud  kubectl get services
NAME           LABELS                                    SELECTOR            IP(S)             PORT(S)
frontend       name=frontend                             name=frontend       10.191.254.168    80/TCP
                                                                             130.211.174.240
kubernetes     component=apiserver,provider=kubernetes   <none>              10.191.240.1      443/TCP
redis-master   name=redis-master                         name=redis-master   10.191.249.193    6379/TCP
redis-slave    name=redis-slave                          name=redis-slave    10.191.249.83     6379/TCP
➜  kubernetes-gcloud
```

## 7 - use the service

put <public IP> in browser;

If you want to find out the IP address for access the web service later;
`kubectl describe services frontend | grep "LoadBalancer Ingress` which will return 

```
➜  kubernetes-gcloud  kubectl describe services frontend | grep "LoadBalancer Ingress
pipe dquote> "
Name:			frontend
Namespace:		default
Labels:			name=frontend
Selector:		name=frontend
Type:			LoadBalancer
IP:			10.191.254.168
LoadBalancer Ingress:	130.211.174.240
Port:			<unnamed>	80/TCP
NodePort:		<unnamed>	31885/TCP
Endpoints:		10.188.0.6:80,10.188.2.5:80,10.188.2.6:80
Session Affinity:	None
No events.
```

![https://www.evernote.com/l/AS417vKXKkVPt4KZYHz0ajdA2e-g3oYhtcM][2]

[2]:https://www.evernote.com/l/AS417vKXKkVPt4KZYHz0ajdA2e-g3oYhtcM "guestbook in web browser"

#### resizing a replication controller: Changing the number of web servers
We can scale up and down the pods,

```
➜  kubernetes-gcloud  kubectl scale --replicas=5 rc frontend
scaled
➜  kubernetes-gcloud  kubectl get pods
NAME                 READY     STATUS    RESTARTS   AGE
frontend-7hw0k       1/1       Running   0          51s
frontend-coi93       1/1       Running   0          1h
frontend-jhcrg       1/1       Running   0          1h
frontend-mw7q5       1/1       Running   0          1h
frontend-vva9j       1/1       Running   0          51s
redis-master-7oniw   1/1       Running   0          7h
redis-slave-aa1s2    1/1       Running   0          3h
redis-slave-dv0x4    1/1       Running   0          3h
➜  kubernetes-gcloud  kubectl scale --replicas=3 rc frontend
scaled
➜  kubernetes-gcloud  kubectl get pods
NAME                 READY     STATUS    RESTARTS   AGE
frontend-7hw0k       1/1       Running   0          1m
frontend-jhcrg       1/1       Running   0          1h
frontend-vva9j       1/1       Running   0          1m
redis-master-7oniw   1/1       Running   0          7h
redis-slave-aa1s2    1/1       Running   0          3h
redis-slave-dv0x4    1/1       Running   0          3h
```

## 8 - Cleanup

* Delete the frontend service to clean up its external load balancer.

`kubectl delete services frontend`

* When you're done with your cluster, you can shut it down:

`gcloud container clusters delete guestbook`


[1]: https://github.com/kubernetes/kubernetes/blob/master/docs/getting-started-guides/gce.md
[2]: https://cloud.google.com/container-engine/docs/tutorials/guestbook





