# Ansible Playbook DevOps Infrastructure

This Ansible Playbook deploys a kubernetes cluster

### Prerequisites

To run this project, you must install Ansible on your computer and have at least 2 GNU/Ubuntu servers running.
#### Warning: Kubernetes needs at least 2 cores and 2 GB of RAM to run.

Dev Env :
```
- Proxmox Server (12 Core - 32 Gb RAM)
 - Master
  - Worker_1
  - Worker_2
```

```
apt install ansible
```

Ansible work by SSH, so it's mandatory to have an ssh key.

### Installing

#### On your computer

You need to add this lines to the end of the "/etc/ansible/hosts" file, don't forget to edit the IPs.
```
[Master]
master ansible_host=xxx.xxx.xxx.xxx
[Worker]
worker_1 ansible_host=xxx.xxx.xxx.xxx
worker_2 ansible_host=xxx.xxx.xxx.xxx

[all:vars]
ansible_python_interpreter=/usr/bin/python3
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

## Running the tests

Once the environment is set up, you can perform this test to see if all the servers are seen by your computer :
```
root@computer:~# ansible all -m ping -u root
master | SUCCESS => {
    "changed": false,
    "ping": "pong"
}
worker_1 | SUCCESS => {
    "changed": false,
    "ping": "pong"
}
worker_2 | SUCCESS => {
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

If you want all the outputs of each executed command, add the "-v" option
```
ansible-playbook -v -u root main.yml
```

The infrastructure deployed by this playbook is in fact unstable, all the kubernetes pods don't have internet, so Jenkins does too.
Your (small) infrastructure is now (partially) configured.

#### Here are some examples of my environment to guide you :
```
root@master:~# kubectl get nodes
NAME             STATUS   ROLES    AGE   VERSION
master           Ready    master   3d    v1.19.0
worker-1         Ready    <none>   3d    v1.19.0
worker-2         Ready    <none>   3d    v1.19.0
```
Here we can see all the nodes currently connected.

```
root@master:~# kubectl get pods -A
NAMESPACE     NAME                                   READY   STATUS    RESTARTS   AGE
jenkins       jenkins-deployment-667fb997fb-qznlx    1/1     Running   1          3d
kube-system   coredns-f9fd979d6-g5fvp                1/1     Running   1          3d
kube-system   coredns-f9fd979d6-lf79d                1/1     Running   1          3d
kube-system   etcd-ayomi-master                      1/1     Running   1          3d
kube-system   kube-apiserver-ayomi-master            1/1     Running   1          3d
kube-system   kube-controller-manager-ayomi-master   1/1     Running   1          3d
kube-system   kube-flannel-ds-amd64-2rbdx            1/1     Running   1          3d
kube-system   kube-flannel-ds-amd64-kk7h5            1/1     Running   1          3d
kube-system   kube-flannel-ds-amd64-m4bfp            1/1     Running   1          3d
kube-system   kube-proxy-2fdl2                       1/1     Running   1          3d
kube-system   kube-proxy-7ms27                       1/1     Running   1          3d
kube-system   kube-proxy-kjkrs                       1/1     Running   1          3d
kube-system   kube-scheduler-ayomi-master            1/1     Running   1          3d
```
The "-A" option is for "--all-namespaces", we can currently see jenkins running.

```
root@master:~# kubectl describe pods/jenkins-deployment-667fb997fb-qznlx --namespace jenkins
Name:         jenkins-deployment-667fb997fb-qznlx
Namespace:    jenkins
Priority:     0
Node:         worker-2/192.168.1.131
Start Time:   Fri, 28 Aug 2020 14:34:04 +0000
Labels:       app=jenkins
              pod-template-hash=667fb997fb
Annotations:  <none>
Status:       Running
IP:           172.16.1.3
IPs:
  IP:           172.16.1.3
Controlled By:  ReplicaSet/jenkins-deployment-667fb997fb
[...]
```
Here is a description of the jenkins pod, it currently works on the worker-2 whose ip is 192.168.1.131

```
root@master:~# kubectl get services -A
NAMESPACE     NAME         TYPE        CLUSTER-IP    EXTERNAL-IP   PORT(S)                  AGE
default       kubernetes   ClusterIP   10.96.0.1     <none>        443/TCP                  3d1h
jenkins       jenkins      NodePort    10.96.67.74   <none>        8080:30000/TCP           3d
kube-system   kube-dns     ClusterIP   10.96.0.10    <none>        53/UDP,53/TCP,9153/TCP   3d1h
```
And finally, the list of all services and their open ports.

With all this information, I can deduce that jenkin is accessible with a browser on ip 192.168.1.131:30000.

## Built With

* [Proxmox](https://www.proxmox.com/en/) - Used to create the servers (Level 1 hypervisor)
* [Ansible](https://docs.ansible.com/ansible/latest/index.html) - Used to deploy the servers
* [Docker](https://www.docker.com/) - Used to create containers, needed for kubernetes
* [Kubernetes](https://kubernetes.io/) - Used to deploy every services of the infrastructure
* [Jenkins](https://kubernetes.io/) - Used to run the Dev test

## Authors

* **Thibaut Choppy** - *Initial work* - [Linkedin](https://www.linkedin.com/in/thibaut-choppy/)
