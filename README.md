# Ansible Playbook DevOps Infrastructure `[v1.1 - Stable]`

This Ansible Playbook deploys a kubernetes cluster

### Prerequisites

To run this project, you must install Ansible on your computer and have at least 2 GNU/Debian 10.4 servers running.
#### Warning: Kubernetes needs at least 2 cores and 2 GB of RAM to run.

Dev Env :
```
- Proxmox Server (12 Core - 32 Gb RAM)
    - Master
        - Node_1
        - Node_2
```

```
apt install ansible
```

Ansible work by SSH, so it's mandatory to have an ssh key.

### Installing

#### On your computer

First, you must edit the `hosts.ini` file with the IPs of your servers.
```
[Master]
master ansible_host=xxx.xxx.xxx.xxx

[Nodes]
node-1 ansible_host=xxx.xxx.xxx.xxx
node-2 ansible_host=xxx.xxx.xxx.xxx
```

Then transfer your user's ssh key to all servers :
```
scp ~/.ssh/id_rsa.pub user@xxx.xxx.xxx.xxx:/home/user
```

#### On all the servers

Add the previously transferred ssh key to the authorized_keys file of the root user :
```
sudo su
cat /home/user/id_rsa.pub >> /root/.ssh/authorized_keys
```
> Create it if it doesn't exist:
> 
> `$> touch /root/.ssh/authorized_keys && chmod 600 /root/.ssh/authorized_keys` 

## Running the tests

Once the environment is set up, you can perform this test to see if all servers are seen by your computer:
```
root@computer:~# ansible all -m ping -u root
master | SUCCESS => {
    "changed": false,
    "ping": "pong"
}
node-1 | SUCCESS => {
    "changed": false,
    "ping": "pong"
}
node-2 | SUCCESS => {
    "changed": false,
    "ping": "pong"
}
```
If the ping failed, check if your ssh keys are in the authorized_keys file of the root user of the unreached server.

## Deployment

To execute the main.yml playbook, run the command :
```
ansible-playbook -u root main.yml
```

> If you want all the outputs of each executed command, add the `-v` option

#### Your (small) infrastructure is now configured !

### Here are some examples of my environment to guide you :
```
root@master:~# kubectl get nodes
NAME             STATUS   ROLES    AGE   VERSION
master           Ready    master   3d    v1.19.2
node-1           Ready    <none>   3d    v1.19.2
node-2           Ready    <none>   3d    v1.19.2
```
Here we can see all the nodes currently connected.

```
root@master:~# kubectl get pods -A
jenkins       jenkins-deployment-667fb997fb-ncdwb   1/1     Running   0          48m
kube-system   coredns-f9fd979d6-swhkp               1/1     Running   0          48m
kube-system   coredns-f9fd979d6-wcnn8               1/1     Running   0          48m
kube-system   etcd-master                           1/1     Running   0          48m
kube-system   kube-apiserver-master                 1/1     Running   0          48m
kube-system   kube-controller-manager-master        1/1     Running   0          48m
kube-system   kube-flannel-ds-lnt9d                 1/1     Running   0          48m
kube-system   kube-flannel-ds-phbkd                 1/1     Running   0          48m
kube-system   kube-flannel-ds-v88hw                 1/1     Running   1          48m
kube-system   kube-proxy-pspws                      1/1     Running   0          48m
kube-system   kube-proxy-rnbkk                      1/1     Running   0          48m
kube-system   kube-proxy-w7jj6                      1/1     Running   0          48m
kube-system   kube-scheduler-master                 1/1     Running   0          48m
```
The `-A` option is for "--all-namespaces", we can currently see jenkins running.

```
root@Master:~# kubectl describe pod/jenkins-deployment-667fb997fb-ncdwb --namespace jenkins
Name:         jenkins-deployment-667fb997fb-ncdwb
Namespace:    jenkins
Priority:     0
Node:         node-1/192.168.1.20
Start Time:   Tue, 22 Sep 2020 01:04:45 +0200
Labels:       app=jenkins
              pod-template-hash=667fb997fb
Annotations:  <none>
Status:       Running
IP:           10.244.1.2
IPs:
  IP:           10.244.1.2
Controlled By:  ReplicaSet/jenkins-deployment-667fb997fb
[...]
```
Here is a description of the jenkins pod, it currently works on the node-1 whose IP is 192.168.1.20

```
root@master:~# kubectl get services -A
NAMESPACE     NAME         TYPE        CLUSTER-IP       EXTERNAL-IP   PORT(S)                  AGE
default       kubernetes   ClusterIP   10.96.0.1        <none>        443/TCP                  48m
jenkins       jenkins      NodePort    10.108.213.130   <none>        8080:30000/TCP           48m
kube-system   kube-dns     ClusterIP   10.96.0.10       <none>        53/UDP,53/TCP,9153/TCP   48m
```
And finally, the list of all services and their open ports.

With all this information, I can deduce that jenkin is accessible with a browser on IP `192.168.1.20:30000`.

## Built With

* [Proxmox](https://www.proxmox.com/en/) - Used to create the servers (Level 1 hypervisor)
* [Ansible](https://docs.ansible.com/ansible/latest/index.html) - Used to deploy the servers
* [Docker](https://www.docker.com/) - Used to create containers, needed for kubernetes
* [Kubernetes](https://kubernetes.io/) - Used to deploy every services of the infrastructure
* [Jenkins](https://kubernetes.io/) - Used to run the Dev test

## Authors

* **Thibaut Choppy** - *Initial work* - [Linkedin](https://www.linkedin.com/in/thibaut-choppy/)
