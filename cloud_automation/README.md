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

## Install OpenFaaS

First **create a new VM, using the provided information, calling it VM3**. Then, on VM3:

```
git clone https://github.com/openfaas/faasd --depth=1
cd faasd
./hack/install.sh

```

Open port 8080 in the AWS security group for VM3 (remember: **for your laptop only!**) and you should be able to see the OpenFaaS web page pointing your browser to http://_<VM3_public_IP>_:8080.

However, if your network blocks access to port 8080, you may issue this command on VM3:

```
sudo iptables -t nat -A PREROUTING -p tcp --dport 80 -j REDIRECT --to 8080
```

### Log in from the terminal

```
sudo cat /var/lib/faasd/secrets/basic-auth-password | faas-cli login --password-stdin
```

### Log in from the Web interface

The username is `admin`. The password can be found typing the command `sudo cat /var/lib/faasd/secrets/basic-auth-password`. 

## Writing serverless functions

### Install the OpenFaas CLI on VM1

```
curl -sSL https://cli.openfaas.com | sudo -E sh
```

### Hello World

Fetch the latest available list of templates with `faas-cli template pull`; check what they are with `faas-cli new --list`. 

```
mkdir -p ~/cloud_automation/hello
cd ~/cloud_automation/hello
faas-cli new --lang python3 hello-python --prefix="<your_dockerhub_username>"
```

### A DNA base counter

```
mkdir -p ~/cloud_automation/dnabase
cd ~/cloud_automation/dnabase
faas-cli new --lang python3 count-base --prefix="<your_dockerhub_username>"
```

Get the `handler.py` function from this repository and put it in the `count-base` directory. 

Edit `requirements.py` adding a line with the text `requests`, to specify that the `requests` module is needed by our function.

Finally, build and deploy the function:

```
faas-cli build -f ./count-base.yml
faas-cli push -f ./count-base.yml
faas-cli deploy -f ./count-base.yml
```

Or, with a single command, `faas-cli up -f count-base.yml`.

Try it out from the OpenFaas web interface passing for instance a string such as `AAACC` or a url with some real DNA, such as https://github.com/dsalomoni/bdp2-2022/raw/main/cloud_automation/yeast.txt. 

Finally, try to call the `count-base` function from a remote location (for example, your laptop) either via Python or directly via `curl`, targeting the endpoint `http://<VM3_public_ip>:8080/function/count-base`.