## 1 K8s
> <p>Palying  with k8s is a next step after docker and docker swarm.<br>
There are plenty of resources starting from local minikube or KIND  , or in the cloud like AWS GCP
K9s is a terminal based UI to interact with your Kubernetes clusters.

Too make it easier to navigate, observe and manage your deployed applications in the wild I started from K9s.
K9s continually watches Kubernetes for changes and offers subsequent commands to interact with your observed resources.

[K9s](https://github.com/derailed/k9s)
 _quickly installing  with_ 

 ```
 curl -sS https://webinstall.dev/k9s | bash
 ```

![](k9s.JPG)

 ---

 
 




## 2.  Collecting an d presenting logs in K8s environment.

Once in the cluster there are couple of applications like database,python aplications,  message broker crosschecking all of them especially in real time could a challenge simple kubectl describe pod etc will not be enough. It is even worse if the one of them is not working then logs are essetial for debugging.

Now the question is how do application log this data?
There are few options:

*applications right to a file
however as you can imagine it's difficult to analyze
loads of data in raw log files

*another option could be to log directly
into a log database like elastic for example
to then have a visualization of this
data however in this case
each application developer must add a
library for elastic search and configure it to connect to elastic 

third-party solution Fluentd does that reliably meaning
if there is a network outage or data spikes this shouldn't mess up data collection.
It starts collecting logs from all the applications: it can be your own applications
third-party applications. All of it now these logs that fluentd collected will be of different forms  like json format, nginx format some custom format, and so on so fluentd will process them and reformat them into a uniform way. After fluentd processes them, in most cases the goal is to nicely visualize  and analyse them.


Fluentd can send these logs to any destination you want elasticsearch, mongodb, s3, kafka
maybe  you want your python application logs to go to mongodb storage for data analysis and all other application logs to go to elasticsearch
You can actually very easily configure that routing in fluentd.

You could  install fluentd in kubernetes as a DaemonSet
DaemonSet is a component that runs on each kubernetes node. 

You can configure fluenty using a fluentd configuration file.
it's very powerful in terms of processing and reformatting your data
fluentd has tons of plugins for different use cases
 you can define the data
sources these are all the applications
 you configure how
these data entries will be processed
line by line  you parse each log as an individual
key value pair
after that you can enrich the data using
record transformers
finally you have
the output where should the logs go
and for each such output target
there is a plugin like elasticsearch, mongodb


```

```
## Setup Kubernetes cluster locally/in the cloud  in minutes using Talos
[Talos](https://www.talos.dev/v1.2/) is a container optimized Linux distro; a reimagining of Linux for distributed systems such as Kubernetes. Designed to be as minimal as possible while still maintaining practicality.

There are plenty of diffrent options installing one localluy or in the cloud 
I used one installed locally on Ubuntu 20  

Steps:
1. Downloading and installing `talosctl` binary

```
wget https://github.com/talos-systems/talos/releases/download/v0.8.1/talosctl-linux-amd64
```
sudo install talosctl-linux-amd64 /usr/local/bin/talosctl 
2. We also need [kubectl](https://kubernetes.io/docs/tasks/tools/install-kubectl-linux/) easiest option with curl
 ```
curl -LO "https://dl.k8s.io/release/$(curl -L -s https://dl.k8s.io/release/stable.txt)/bin/linux/amd64/kubectl"
 ```

Install kubectl
 ```
sudo install -o root -g root -m 0755 kubectl /usr/local/bin/kubectl
 ```
 Once in place lets create cluster with three worker nodes and one control plane of course any other configuration with the use of command line is 
 possible 
```
talosctl cluster create --wait --workers 3
```
result:
```
NAME              talos-default
NETWORK NAME      talos-default
NETWORK CIDR      10.5.0.0/24
NETWORK GATEWAY   10.5.0.1
NETWORK MTU       1500

NODES:

NAME                      TYPE           IP         CPU    RAM      DISK
/talos-default-master-1   controlplane   10.5.0.2   2.00   2.1 GB   -
/talos-default-worker-1   join           10.5.0.3   2.00   2.1 GB   -
/talos-default-worker-2   join           10.5.0.4   2.00   2.1 GB   -
/talos-default-worker-3   join           10.5.0.5   2.00   2.1 GB   -
```
There is even text dasboard to have overview of k8s talos cluster 

```
talosctl -n  10.5.0.2,10.5.0.3,10.5.0.4,10.5.0.5 dashboard
kubectl get nodes -o wide
```
at the end 
```
talosctl cluster destroy
```
### Status
Project is: _in progress_ 
