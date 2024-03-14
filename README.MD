
## Fully Automated K3S etcd High Availability Install


Proxmox is an open-source virtualization platform that combines KVM virtualization, container-based virtualization, and software-defined storage and networking, providing a comprehensive solution for managing virtualized environments. It offers a web-based management interface and supports various virtualization technologies, making it suitable for both small-scale deployments and enterprise environments.

Using Ansible, it's possible to automate the creation of a fully functional HA (High Availability) K3s cluster based on previously created VMs with cloud-init Ubuntu Server images. Ansible playbooks can be crafted to provision VMs on Proxmox, ensuring they have the necessary resources and configurations specified for a K3s cluster. Subsequently, Ansible can orchestrate the installation and configuration of K3s on these VMs, setting up etcd HA, master and worker nodes, and ensuring proper networking and security configurations. By leveraging Ansible's automation capabilities, the entire process can be streamlined, allowing for rapid and consistent deployment of HA K3s clusters in Proxmox environments, reducing manual effort and minimizing the potential for configuration errors.
All this could be just bla bla, <br> I was also sceptiacal. Though after following what TIm did I was pleasantly surprised how comfortable and automated whole process could be.

---
![](k3s.jpg)

See  and use yourself <br>
https://technotim.live/posts/k3s-etcd-ansible/   <br>
https://github.com/techno-tim/k3s-ansible

---



__two years ago ie in 2022 I was going more manual way__ :

---

## 1 K8s
> <p>Playing  with k8s is a next step after docker and docker swarm.<br>
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

 
 




## 2.  Collecting and presenting logs in K8s environment( fluentd  installed as DeamonSet, Elasticsearch to visualise).

Once in the cluster there are couple of applications like database,python aplications,  message broker crosschecking all of them especially in real time could a challenge simple kubectl describe pod etc will not be enough. It is even worse if the one of them is not working then logs are essetial for debugging.


Now the question is how do application log this data?<b>
<br/>
There are few options:

* applications right to a file
however as you can imagine it's difficult to analyze
loads of data in raw log files

* another option could be to log directly
into a log database like elastic for example
to then have a visualization of this
data however in this case
each application developer must add a
library for elastic search and configure it to connect to elastic 

* third-party solution [Fluentd](https://github.com/fluent/fluentd) does that reliably meaning
if there is a network outage or data spikes this shouldn't mess up data collection.
It starts collecting logs from all the applications: it can be your own applications
third-party applications. All of it now these logs that fluentd collected will be of different forms  like json format, nginx format some custom format, and so on so fluentd will process them and reformat them into a uniform way. After fluentd processes them, in most cases the goal is to nicely visualize  and analyse them.
---


![](https://camo.githubusercontent.com/9386aa8e7cb67334bf3239e648abb3c034a521ba8cc48a34911645da5fd903d4/68747470733a2f2f7777772e666c75656e74642e6f72672f696d616765732f666c75656e74642d6172636869746563747572652e706e67)

---
 
Fluentd can send these logs to any destination you want elasticsearch, mongodb, s3, kafka.
Maybe  you want your python application logs to go to mongodb storage for data analysis and all other application logs to go to elasticsearch.
You can actually very easily configure that routing in fluentd.

 ===
You could  install fluentd in kubernetes as a DaemonSet(_DaemonSet is a component that runs on each kubernetes node_). 

You can configure fluenty using a fluentd configuration file. It's very powerful in terms of processing and reformatting your data
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
helm repo add fluent https://fluent.github.io/helm-charts
```

 ## 3.  Creating Data science environment based on K8s
 
 * Locally with the use of Hyper-V VM on Windows. Good guide how to do it is under this address
 https://www.nakivo.com/blog/run-linux-hyper-v <br/>
 * installation of [minikube](https://github.com/kubernetes/minikube)  <br/>
 
 Once minikube environment is ready we could start installing required parts: <br/>
 * OLM (Operator Lifecycle Manager)
  Operator Framework, an open source toolkit to manage Kubernetes native applications, called Operators, in an effective, automated, and scalable way
 
  https://github.com/operator-framework/operator-lifecycle-manager/releases/ <br/>
  For the time of writing this info latest release was as follows :<br/>
 
 ```
 curl -L https://github.com/operator-framework/operator-lifecycle-manager/releases/download/v0.21.2/install.sh -o install.sh
chmod +x install.sh
./install.sh v0.21.2
 ```
*  Keycloak  Keycloak, an open source software to provide authentication services.  The goal for 
this component is to provide a common single sign-on feature for all the platform 
components
 Good guide how to make everything work is under the link:
 [Keycloak](https://www.keycloak.org/getting-started/getting-started-kube) <br/>
 ---
 
 To make work more efficient importing realm to keycloak will be used:<br/>
 https://www.keycloak.org/operator/realm-import
 
 Importing directly from admin console realmsettings/Action( right)corner/ Partial import and giving path to json file with settings is good way 
 yo recreate configuration.

```

```
## 3 Setup Kubernetes cluster locally/in the cloud  in minutes using Talos
[Talos](https://www.talos.dev/v1.2/) is a container optimized Linux distro; a reimagining of Linux for distributed systems such as Kubernetes. Designed to be as minimal as possible while still maintaining practicality.

There are plenty of diffrent options installing one locally or in the cloud 
I used one installed locally on Ubuntu 20  

Steps:
1. Downloading and installing `talosctl` binary

```
wget https://github.com/talos-systems/talos/releases/download/v0.8.1/talosctl-linux-amd64

sudo install talosctl-linux-amd64 /usr/local/bin/talosctl 
```
2. We also need [kubectl](https://kubernetes.io/docs/tasks/tools/install-kubectl-linux/) easiest option with curl
 ```
curl -LO "https://dl.k8s.io/release/$(curl -L -s https://dl.k8s.io/release/stable.txt)/bin/linux/amd64/kubectl"
 ```

Install kubectl
 ```
sudo install -o root -g root -m 0755 kubectl /usr/local/bin/kubectl
 ```
 Once in place lets create cluster with three worker nodes and one control plane of course any other [configurations](https://www.talos.dev/v1.2/reference/cli/) with the use of command line is 
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

NAME                     STATUS   ROLES                  AGE   VERSION   INTERNAL-IP   EXTERNAL-IP   OS-IMAGE         KERNEL-VERSION      CONTAINER-RUNTIME
talos-default-master-1   Ready    control-plane,master   17m   v1.20.1   10.5.0.2      <none>        Talos (v0.8.1)   5.13.0-40-generic   containerd://1.4.3
talos-default-worker-1   Ready    <none>                 17m   v1.20.1   10.5.0.3      <none>        Talos (v0.8.1)   5.13.0-40-generic   containerd://1.4.3
talos-default-worker-2   Ready    <none>                 17m   v1.20.1   10.5.0.4      <none>        Talos (v0.8.1)   5.13.0-40-generic   containerd://1.4.3
talos-default-worker-3   Ready    <none>                 17m   v1.20.1   10.5.0.5      <none>        Talos (v0.8.1)   5.13.0-40-generic   containerd://1.4.3
```
 When you could/want more memory per worker node/control plabe then:
 
 
```
 talosctl cluster create --wait --workers 3 --memory 6144
```
There is even text dasboard to have overview of k8s talos cluster 

```
talosctl -n  10.5.0.2,10.5.0.3,10.5.0.4,10.5.0.5 dashboard
kubectl get nodes -o wide
```

Finally, we just need to specify which nodes you want to communicate with using talosctl. Talosctl can operate on one or all the nodes in the cluster – this makes cluster wide commands much easier.
```
talosctl config nodes 10.5.0.2 10.5.0.3 10.5.0.4 10.5.0.5
```
there is buildin text dashboard 
```
talosctl -n  10.5.0.2,10.5.0.3,10.5.0.4,10.5.0.5 dashboard
```

![](dasboard-talos.JPG)

at the end 
```
talosctl cluster destroy
```
### Status
Project is: _in progress_ 
