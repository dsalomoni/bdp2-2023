# bdp2-2023 - Cloud Automation
This repository contains files used in the course <b>Infrastructures for Big Data Processing</b> (BDP2) at the University of Bologna, Academic Year 2022-2023, taught by prof. Davide Salomoni.

For details, see the course slides.

For more information on the course, see <a href="https://www.unibo.it/it/didattica/insegnamenti/insegnamento/2022/435337">here</a>.

Note: unless specified otherwise, all the commands below should be performed on VM1.

## Create a directory for this module and go there

```
mkdir -p ~/cloud_automation
cd ~/cloud_automation

```

## Install kubectl

`kubectl` is a command line tool used to control Kubernetes clusters. Install it with the following commands:

```
curl -LO "https://dl.k8s.io/release/$(curl -L -s https://dl.k8s.io/release/stable.txt)/bin/linux/amd64/kubectl"
chmod +x ./kubectl
sudo mv ./kubectl /usr/local/bin/

```

## Install minikube

```
curl -LO https://storage.googleapis.com/minikube/releases/latest/minikube-linux-amd64
chmod +x ./minikube-linux-amd64
sudo mv ./minikube-linux-amd64 /usr/local/bin/minikube

```

Before moving on, check that both kubectl and minikube work with the commands

```
kubectl version --output=json
minikube version
```

## Creation of a one-node Kubernetes cluster

```
minikube create cluster
```

Check what happened with the following commands:

```
docker ps
kubectl cluster-info
kubectl get nodes
kubectl get nodes -o wide
docker exec minikube ip address show eth0
kubectl get pods
```

Can you make out what type of output each command returns, and how they relate to each other?

## Destroy the Kubernetes cluster

```
minikube delete
```

Check that the cluster is gone with 

```
docker ps
kubectl cluster-info
```

## Creation of a Kubernetes cluster with one control node and two workers

```
minikube start --nodes 3
```

Check again what happened with the following commands:

```
docker ps
docker ps --format "table {{.ID}} | {{.Names}}"
kubectl cluster-info
kubectl get nodes -o wide
kubectl get pods
```

You should now have your Kubernetes cluster running. Notice that you still have no running pods in the cluster.

## Creation of the nginx pod(s)

### Create a single pod

```
kubectl apply -f nginx-pod.yaml
```

Check the status of the cluster before and after the command above with `kubectl get all -o wide`

You can delete the pod with `kubectl delete pod nginx`.

### Create a Deployment

Open a second terminal window on VM1 and issue the command `watch kubectl get all -o wide`. This will periodically check the status of our Kubernetes cluster. 

Create a "Kubernetes Deployment" now with

```
kubectl apply -f nginx-deployment.yaml
```

and observe what happens to the cluster. 

Now delete one of the pods with `kubectl delete pod nginx-deployment-xxxxx` (replace xxxxx with the appropriate string for one of your pods). What happens? Play with the number of replicas in the YAML file.

You can delete the Deployment with `kubectl delete deployment nginx-deploy`. 

## Scaling up and down

Try these two methods:
- edit the YAML file and then run `kubectl apply -f nginx-deployment.yaml`.
- Just type `kubectl scale deployment/nginx-deployment --replicas=5` (or whatever number).

## Exposing IP addresses

Focus on the difference between what was described in class:
- using __port forwarding__: `kubectl port-forward nginx 8888:80`.
  - Connect with a browser to http://_<VM1_public_IP>_:8888 (make sure you open incoming port 8888 in the AWS security group).
- __exposing the deployment__ called _nginx-deployment_: `kubectl expose deployment nginx-deployment --type=LoadBalancer --port=80`.
  - check with `kubectl get all`. Notice we do not have an _external IP_ yet.
  - run `minikube tunnel` and in another terminal check what happened with `kubectl get all`.
  - check with `curl <LoadBalancer_address>:80` that you can access one of the nginx pods of the deployment.

## Docker Swarm

### Initialize the cluster

On the `manager`:

```
docker swarm init --advertise-addr <MANAGER-PRIVATE-IP>
```

### Add a node to the cluster

Issue the command `docker swarm join-token worker` on the `manager`. Copy the output command and issue that command on the node that you add to the cluster (that is, issue that command on `worker1` and `worker2`).

### Create a Swarm service

On the `manager`:

```
docker service create --replicas 3 -p 8082:80 --name web_swarm nginx
```

### Check the Swarm status

Check the status of the `web_swarm` service:

```
docker service ls
docker service ps web_swarm
```

Verify that on `worker1` and `worker2` you have one of more containers running, using `docker ps`.

### Scaling, draining

Scaling a service: `docker service scale web_swarm=5`. Check with `docker service ps web_swarm`. 

Draining a node: `docker node update --availability drain <worker2>`
Making a drained node available: `docker node update --availability drain <worker2>`

### Load balancing the web servers

Create a directory on the `manager` and change to that directory:

```
mkdir -p ~/balancer
chdir ~/balancer
```

Copy `nginx.conf` from this GitHub repo and then edit it as explained in the course slides; copy also the `Dockerfile`. Build the load balancer image and run it with

```
docker build -t load_balancer .
docker run --rm -d -p 80:80 load_balancer
```

At this point, if you open port 80 on the proper security group, and open `http://<manager_public_ip>:80` in a browser, you should get a web page displayed.

### Additional commmands

Remove a Swarm service: `docker service rm <service_name>`
Secrets:
- generate a random password with `openssl rand -base64 10`
- create a secret: `printf "a strong password" | docker secret create my_password -`
- create a Swarm service with access to a certain secret: `docker service create --name nginx_secure --secret my_password -p 80:80 nginx`
- verify that a secret is visible to a Swarm service under `/run/secrets`: `docker exec <container_id> cat /run/secrets/my_password`
- revoke access to a secret for a Swarm service: `docker service update --secret-rm my_password nginx_secure`
- remove a secret: `docker secret rm my_password`
