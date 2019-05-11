# kubernetes-install
Install and Deploy Kubernetes on Ubuntu 18.04 LTS

1. Install Docker on all nodes:

 ```sudo apt install docker.io```

2. Check docker install:

```docker --version```

3. Enable Docker on all nodes:

```sudo systemctl enable docker```

4. Add the Kubernetes signing key on all nodes:

```curl -s https://packages.cloud.google.com/apt/doc/apt-key.gpg | sudo apt-key add```

5. Add Kubernetes Repository on all nodes:

```sudo apt-add-repository "deb http://apt.kubernetes.io/ kubernetes-xenial main"```

6. Install Kubeadm:

```sudo apt install kubeadm```

7. Check Kubeadm installation:

```kubeadm version```

8. Disable swap memory on all nodes:

```sudo swapoff -a```

and eliminate any occurrence of swap in ```/etc/fstab``` using 

```sudo nano /etc/fstab```

9. Initialize Kubernetes on the master node:

```sudo kubeadm init```

10. Run the following:

```mkdir -p $HOME/.kube```

```sudo cp -i /etc/kubernetes/admin.conf $HOME/.kube/config```

```sudo chown $(id -u):$(id -g) $HOME/.kube/config```

11. On the remaining nodes run the command starts with:

```kubeadm join 192.168.100.6:6443 --token ...```

You can find the full command in the results of step 9

12. Create pod network through the master node:

```kubectl apply -f "https://cloud.weave.works/k8s/net?k8s-version=$(kubectl version | base64 | tr -d '\n')"```

13. Check the cluster works correctly:

```kubectl get nodes``` on master node
