# Creating a 3 node kubernetes cluster with ansible

### Requirements
1. A host with Ansible installed. I’ve written previously about how to install Ansible – also, check out the online documentation!
   
   You should also set up an SSH key pair, which will be used to authenticate to the Kubernetes nodes without using a password, allowing Ansible to do it’s thing.

2. Three servers/hosts to which we will use as our targets to deploy Kubernetes. Our servers each have 8GB ram and 4vCPUs.

   This is fine for my lab purposes, which we use to try out new things using Kubernetes. You need to be able to SSH into each of these nodes as root using the SSH key pair we mentioned above.

### Setting up Ansible to Deploy Kubernetes
Before we start to look at the Ansible Playbooks, we need to set up Ansible to communicate with the Kubernetes nodes. 

First of all, on our Ansible host, lets set up a new directory from which we we run our playbooks.

```
mkdir kubernetes
cd kubernetes
```

With that done, we now need to create a hosts file, to tell Ansible how to communicate with the Kubernetes master and worker nodes.

[hosts](https://github.com/ssayibu/k8s-cluster-with-ansible/blob/main/hosts)

Finally, with that done, we can test it’s working by doing a Ansible ping:
```
ansible -i hosts all -m ping
```

Sample output
```
Output

master | SUCCESS => {
    "changed": false,
    "ping": "pong"
}
worker2 | SUCCESS => {
    "changed": false,
    "ping": "pong"
}
worker1 | SUCCESS => {
    "changed": false,
    "ping": "pong"
}
```

Our first task in setting up the Kubernetes cluster is to create a new user on each node.

[user.yaml](https://github.com/ssayibu/k8s-cluster-with-ansible/blob/main/users.yaml)

We’re now ready to run our first playbook. To do so:
```
ansible-playbook -i hosts users.yaml
```
![A9](https://github.com/user-attachments/assets/3399c2a4-2291-4236-b696-083a6d341d10)

Lets dive straight in and have a look at the playbook, which we have named Install-k8s.yaml:

[Install-k8s.yaml](https://github.com/ssayibu/k8s-cluster-with-ansible/blob/main/Install-k8s.yaml)

This playbook will run against all three nodes, and will install the containerd runtime (including some pre-requisite configuration, then go onto install Kubernetes, which includes kubelet, kubeadm and kubectl.
```
ansible-playbook -i hosts Install-k8s.yaml
```
Sample Output
```
PLAY RECAP ****************************************************************************
master                     : ok=13   changed=12   unreachable=0    failed=0
worker1                    : ok=13   changed=12   unreachable=0    failed=0
worker2                    : ok=13   changed=12   unreachable=0    failed=0
```

Now we should have containerd and Kubernetes installed on all our nodes. The next step is to create the cluster on the master node.

[master.yaml](https://github.com/ssayibu/k8s-cluster-with-ansible/blob/main/master.yaml)

execute the masters.yaml playbook:
```
ansible-playbook -i hosts master.yaml
```

Once the playbook has finished, we can check the outcome by connecting to the cluster master node using SSH to check the status of the master node:
```
kubectl get nodes
```

Now we have a Kubernetes cluster initialised, the final step is to join our worker nodes to the cluster. To do so, the final playbook – join-workers.yml – contains the following:

[worker.yaml](https://github.com/ssayibu/k8s-cluster-with-ansible/blob/main/worker.yaml)

This works by copying the file containing the worker join command saved locally earlier to the worker nodes, then it runs the command. Run the playbook with:
```
ansible-playbook -i hosts workers.yaml
```

Once the playbook has complete we can check the status of the cluster nodes by again running the following on the cluster master node:
```
kubectl get nodes
```
