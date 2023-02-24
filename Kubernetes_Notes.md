## Notes from website [Link](https://blog.adamchalmers.com/kubernetes-problems/)
#### Containers
* A container, like a virtual machine, lets your service act like its the only service on the machine.
* It gets its own filesystem so you don't have to worry about dependency clashes
* VMs each have their own OS but containers share the same underlying OS
* If you containerize all parts of your application  (frontend, backend, load balancers, session management, auth etc.) it may become impoosible for them to coordinate - enter container orchestration
#### Container Orchestration
* Another problem: which containers should be running in the first place?
    * EX: every time you make a PR to master CI creates a new container image with commit and dependencies. Now you got all these containers, how is your server gonna figure out which is the right one to run?
* A container orchastration program selectivly enables some containers and grants them some resources including "communication with other containers"
* Generally serves these roles:
    * Chose which containers should run
    * How many copies of each container should run
    * Which containers are allowed to share resources, like volumes or directories or files or networks
    * Choose which containers can access the same ports and hosts, to network with each other
* Docker Compose is a simple container orchestration tool built into Docker - but only really usefull if your running your containers on one physical machine. If you need to run containers accross multiple servers, enter Kubernetes!
#### Kubernetes
* Key to the desigin of Kubernetes is a cluster. A cluster is a big abstract virtual computer, with its own virtual IP stack, networks, disk, RAM and CPU.
    * It lets you deploy containers as if you were deploying them on one machine
* A cluster runs on *n* servers. A server which is part of a Kubernetes cluster is called a *node*
### Solving Problems with Kubernetes
#### I want to run a containers
* Imagine you have project code named `kangaroo` thats ready for deployment. You build a contianer image to run it: `kangaroo-backend` 
* You need to deploy this container into the companies Kubernetes cluster. To do this you need to define a Kubernetes Pod
    * A Pod is a set of one or moer containers (usually one). Pods are the **minmum unit of deployment in k8s**, its the simplest smallest thing you can deploy
* All resources, regardless of what your deploying have 4 properties
    * apiVersion - Whcih version of the k8s resource definition to user
    * kind - ex. Pod, Service, etc.
    * metadata - Key-Value pairs that you can use to query and organize the resources in your cluster
    * spec - the details of what your deploying. Every kind has a different spec
#### What is my pod doing?
* `kubectl get pods` - see list of pods
* `kubectl describe pod <ID>` - see more infor about the pod
* `kubectl logs <ID>` - see pods logs
* `kubectl exec <ID> -it -- /bin/bash` - ssh into pod
#### I want my container to restart when it crashes
* Since k8s is declaritve, you just need to declare that a pod should be running and if thats ever not true k8s will fix it
#### My container should never have downtime, even when it crashes
* The best way to handle this is with replicas - running the same container multiple times
* The k8s way of doing this is using a deployment (different KIND)
* spec feild is where deployment specific stuff is defined
    * replicas
    * selector - defines which pods will get managed by the deployment. Defined in metadata of Pod/Service (ex externalauth)
    * template - defines a pod which will get managed by thes deployment
        * makes sure labels in metadata matches selector
        * will have spec section to (within spec section of deployment) with difines what containers the pod should run
#### My container can't handle the ammount of traffic its recieving
* the soultion is replicas once again
* You can even set up elastic scalling to enable kubernetes to autormatically choose the number of replicas, called horizontal auto-scaling (see link)
#### I need to deploy a new version of my backend without causing downtime
* You can support this by updating a deployment - either by using kubectl edit CLI or updating the YAML manifest and rerunning kubectly apply
* This will kick off a rolling upgrade (aka replicas upgraded 1 by 1 rather than all at the same time)
#### I need to route traffic to all pods in a deployment
* Each pod gets it own Cluster IP address (address only makes sense within the Kubernetes cluster)
* this can get tricky when you have a deployment with multiple pods, because they'll each have different IP addresses. Enter a Service
* Deploying a service creates a DNS record in your kubernetes cluster
    * EX: `my-svc.my-namespace.svc.cluster-domain.example`
* The Service object will monitor which pods match its selectors. Whenever the service gets traffic on its specified `port`, it forwards that traffic to the `targetPort` on some matching pod
* For example, frontend sends API requests to cluster hostname (`my-svc.my-namespace.svc.cluster-domain.example`), clusters DNS resolver maps that to service, and service forwards request to an available pod
* Note difference between services and deployments. Both select pods with a certain set of labels, but services handle balancing traffic and discovery whereas deployments ensure your pods exists in the right number and the right configuration
* Generally backends/servers in k8s will have 1 service and 1 deployment selecting on the same labels
#### Load balancing accross pods
* If you're using a k8s service to route traffic to your app, then your getting the basics of load balancing
#### I want to accecpt traffic from outside the cluster
* You need deploy two kinds of resource: and Ingress and an IngressController
    * An Ingress defines rules for mapping cluster-external HTTP/s traffic to cluster-internal Services. From there service will forward to a pod
    * An IngressController executes those rules. Kubernetes suports and maintains 3 particular IngressControllers: AWS, GCE, and Nginx
#### I want to limit networking within the cluster
* You can use a Kubernetes Network Policy to limit your projects networking permisions. You can use Network Policies to restrict:
    * Ingress (who your service can recieve traffic from)
    * Egress (who your service can send traffic to)
* The best security practice is to default deny all networking within your namespace, then open up sepecific exceptions for the networking you know your service needs
* Each ingress rule has 2 parts
    * `from` says which kinds of services can send traffic
    * `ports` says which ports they can send traffic to
* Egress rules have 2 parts
    * `to` which destinations your service can send traffic to
    * `ports` which ports on those destinations are allowed
#### I want isolation from other projects using the same cluster
* Use Namespaces - They let you isolate teams or projects from each other sp their names don't clash
* Use `--namespace` in the kubectl CLI to choose which namespace you want to deploy to
 



