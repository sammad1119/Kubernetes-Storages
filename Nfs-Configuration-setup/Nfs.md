## NFS  Configuration and setup.


## Prerequisites

- Kubernetes Cluster:

A running Kubernetes cluster (v1.9 or higher) with at least one master and multiple nodes.
Ensure kubectl is configured to interact with the cluster.

- NFS Server:

A working NFS server with shared directories. The NFS server can be external to the Kubernetes cluster or one of the nodes within the cluster.

- NFS Client Utilities:

NFS client utilities should be installed on all nodes in the Kubernetes cluster to enable mounting of NFS shares. This typically includes the nfs-common package on Linux distributions.

## Requirements
- NFS Server Configuration:

Properly configure the NFS server to export the directories you want to use with Kubernetes.
Ensure the NFS server's firewall allows traffic on the NFS ports (usually TCP/UDP 2049).

- Persistent Volume (PV) Configuration:

Create a PersistentVolume (PV) definition that specifies the NFS server and the shared directory path. 

- Persistent Volume Claim (PVC):

Create a PersistentVolumeClaim (PVC) that requests storage from the PV.

- Pod Specification:

Mount the PVC in a Pod by specifying it in the Pod's volume definitions

## Setup and Configuration 

- Following are the steps and setting to use  NFS for Persistent Volumes on MicroK8s

## Setup an NFS server

1. create a seprate Node for NFS-server

- If you don’t have a suitable NFS server already, you can simply create one on a local machine with the following commands on Ubuntu:

```
sudo apt-get install nfs-kernel-server
```

2. Create a directory to be used for NFS:

```

sudo mkdir -p /srv/nfs
sudo chown nobody:nogroup /srv/nfs
sudo chmod 0777 /srv/nfs

```
3. Edit the /etc/exports file. Make sure that the IP addresses of all your MicroK8s nodes are able to mount this share. Add the following line.

```
/srv/nfs *(rw,sync,no_subtree_check,no_root_squash,no_all_squash,insecure)
```
4. Export using following command.
```
sudo exportfs -av
```
5. Finally, restart the NFS server:
```
sudo systemctl restart nfs-server

sudo systemctl status nfs-server
```
6. Show mount nodes.
```
sudo showmount -e localhost
```

## Install the CSI driver for NFS

All this shold be done on k8s master Node 

- Install helm and Csi dirvers using helm 

```
microk8s enable helm3
microk8s helm3 repo add csi-driver-nfs https://raw.githubusercontent.com/kubernetes-csi/csi-driver-nfs/master/charts
microk8s helm3 repo update

```

- Then, install the Helm chart under the kube-system namespace with:

```
microk8s helm3 install csi-driver-nfs csi-driver-nfs/csi-driver-nfs \
    --namespace kube-system \
    --set kubeletDir=/var/snap/microk8s/common/var/lib/kubelet
 ```

 - After deploying the Helm chart, wait for the CSI controller and node pods to come up using the following kubectl command …

```
microk8s kubectl wait pod --selector app.kubernetes.io/name=csi-driver-nfs --for condition=ready --namespace kube-system
```
- At this point, you should also be able to list the available CSI drivers in your Kubernetes cluster …
```
microk8s kubectl get csidrivers
```

## Create a StorageClass for NFS or Deploy the nfs provionser on kubernetes.

1. Next, we will need to create a Kubernetes Storage Class that uses the nfs.csi.k8s.io CSI driver. Assuming you have configured an NFS share /srv/nfs and the address of your NFS server is <Server-ip>, create the following file:

```

File name :sc-nfs.yaml

apiVersion: storage.k8s.io/v1
kind: StorageClass
metadata:
  name: nfs-csi
provisioner: nfs.csi.k8s.io
parameters:
  server: <Server-ip>
  share: /srv/nfs
reclaimPolicy: Delete
volumeBindingMode: Immediate

```
2. Then apply it on your MicroK8s cluster master node  with:
```
microk8s kubectl apply -f - < sc-nfs.yaml

```

## Deploy the nfs provionser on kubernetes.

<b>Note:</b> If you dont want to do it using storage class 

Follow below steps in k8s cluster access node

1. Add the repository:
```
helm repo add nfs-subdir-external-provisioner https://kubernetes-sigs.github.io/nfs-subdir-external-provisioner
```
2. Install the chart:
```
helm install dev-nfs-provisioner nfs-subdir-external-provisioner/nfs-subdir-external-provisioner --set nfs.server=10.0.0.7 --set nfs.path=/nfs/kubedata --set storageClass.name=nfs-default-storage-class
```
## Create a new PVC

The final step is to create a new PersistentVolumeClaim using the nfs-csi storage class. This is as simple as specifying storageClassName: <nfs-csi in the PVC>.

```
File name : pvc-nfs.yaml

apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: my-pvc
spec:
  storageClassName: nfs-csi
  accessModes: [ReadWriteOnce]
  resources:
    requests:
      storage: 5Gi

```

- Then create the PVC with:

```
microk8s kubectl apply -f - < pvc-nfs.yaml

```
- If everything has been configured correctly, you should be able to check the PVC.

```
microk8s kubectl describe pvc my-pvc

```

 ## Configure the worker node (SSH to the worker nodes and follow execute below commands)
```
 sudo apt update

sudo apt install nfs-common
```
 ## Verify the nfs mount manully.
```

sudo mkdir -p /nfs/data

sudo mount -t nfs 10.0.0.11:/nfs/kubedata /nfs/data

```
