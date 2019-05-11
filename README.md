# kubernetes-install
Install and Deploy Kubernetes on Ubuntu 18.04 LTS

1. Install Docker on all nodes:

 ```bash
 sudo apt install docker.io
 ```

2. Check docker install:

```bash
docker --version
```

3. Enable Docker on all nodes:

```bash
sudo systemctl enable docker
```

4. Add the Kubernetes signing key on all nodes:

```bash
curl -s https://packages.cloud.google.com/apt/doc/apt-key.gpg | sudo apt-key add
```

5. Add Kubernetes Repository on all nodes:

```bash
sudo apt-add-repository "deb http://apt.kubernetes.io/ kubernetes-xenial main"
```

6. Install Kubeadm:

```bash
sudo apt install kubeadm
```

7. Check Kubeadm installation:

```bash
kubeadm version
```

8. Disable swap memory on all nodes:

```bash
sudo swapoff -a
```

and eliminate any occurrence of swap in ```/etc/fstab``` using 

```bash
sudo nano /etc/fstab
```

9. Initialize Kubernetes on the master node:

```bash
sudo kubeadm init
```

10. Run the following:

```bash
mkdir -p $HOME/.kube
```

```bash
sudo cp -i /etc/kubernetes/admin.conf $HOME/.kube/config
```

```bash
sudo chown $(id -u):$(id -g) $HOME/.kube/config
```

11. On the remaining nodes run the command starts with:

```bash
kubeadm join 192.168.100.6:6443 --token ...
```

You can find the full command in the results of step 9

12. Create pod network through the master node:

```bash
kubectl apply -f "https://cloud.weave.works/k8s/net?k8s-version=$(kubectl version | base64 | tr -d '\n')"
```

13. Check the cluster works correctly:

```bash
kubectl get nodes
``` 
on master node


# Configure local kubectl to access newly created cluster

Generate admin.key and admin.csr using openssl

```bash
openssl genrsa -out admin.key 2048
openssl req -new -key admin.key -out admin.csr -subj "/O=system:masters/CN=kubernetes-admin"
```

Now create CSR in kubernetes using above openssl admin.csr

```bash
cat <<EOF | kubectl create -f -
apiVersion: certificates.k8s.io/v1beta1
kind: CertificateSigningRequest
metadata:
  name: admin_csr
spec:
  groups:
  - system:authenticated
  request: $(cat admin.csr | base64 | tr -d '\n')
  usages:
  - digital signature
  - key encipherment
  - client auth
EOF
```

Now approve the CSR generated using

```bash
kubectl certificate approve admin_csr
```

Now extract the admin.crt from approved CSR

```bash
kubectl get csr admin_csr -o jsonpath='{.status.certificate}' | base64 -d > admin.crt
```

Now change the current user and context to use the new admin key and certificates

```bash
kubectl config set-credentials kubernetes-admin --client-certificate=/home/centos/certs/admin.crt  --client-key=/home/centos/certs/admin.key
kubectl config set-context kubernetes-admin@kubernetes --cluster=kubernetes --user=kubernetes-admin
```


