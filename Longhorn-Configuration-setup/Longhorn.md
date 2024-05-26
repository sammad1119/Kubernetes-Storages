## Longhorn Configuration and setup .

## Supported Kubernetes Versions
Please ensure your Kubernetes cluster is at least v1.21 before upgrading to Longhorn v1.6.1 because this is the minimum version Longhorn v1.6.1 supports.


## Installation Requirements

Each node in the Kubernetes cluster where Longhorn is installed must fulfill the following requirements:

- A container runtime compatible with Kubernetes (Docker v1.13+, containerd v1.3.7+, etc.)
- Kubernetes >= v1.21
- open-iscsi is installed, and the iscsid daemon is running on all the nodes. This is necessary, since Longhorn relies on iscsiadm on the host to provide persistent volumes to Kubernetes. For help installing open-iscsi, refer to this section.
- RWX support requires that each node has a NFSv4 client installed.
- For installing a NFSv4 client, refer to this section.
- The host filesystem supports the file extents feature to store the data. Currently we support:
- ext4
- XFS
- bash, curl, findmnt, grep, awk, blkid, lsblk must be installed.

## Using the Environment Check Script

This script can be used to check the Longhorn environment for potential issues.
```
curl -sSfL https://raw.githubusercontent.com/longhorn/longhorn/v1.6.1/scripts/environment_check.sh | bash

```
## Minimum Recommended Hardware
- 3 nodes
- 4 vCPUs per node
- 4 GiB per node
- SSD/NVMe or similar performance block device on the node for storage (recommended)
- HDD/Spinning Disk or similar performance block device on the node for storage (verified)
- 500/250 max IOPS per volume (1 MiB I/O)
- 500/250 max throughput per volume (MiB/s)

## Installing open-iscsi

```
apt-get install open-iscsi

```
## Installing NFSv4 client

- Check NFSv4.1 support is enabled in kernel

```
cat /boot/config-`uname -r`| grep CONFIG_NFS_V4_1

```

-  For Debian and Ubuntu, use this command:

```
apt-get install nfs-common
```
## Installing Longhorn using kubectl 

Install Longhorn on any Kubernetes cluster using this command:
```
kubectl apply -f https://raw.githubusercontent.com/longhorn/longhorn/v1.6.1/deploy/longhorn.yaml
```
One way to monitor the progress of the installation is to watch pods being created in the longhorn-system namespace:

```
kubectl get pods \
--namespace longhorn-system \
--watch
```

## Installing longhorn using  Helm

Installing Longhorn

- Add the Longhorn Helm repository:
```
helm repo add longhorn https://charts.longhorn.io
```
- Fetch the latest charts from the repository:

```
helm repo update
```
- Install Longhorn in the longhorn-system namespace.
```
helm install longhorn longhorn/longhorn --namespace longhorn-system --create-namespace --version 1.6.1
```
- To confirm that the deployment succeeded, run:
```
kubectl -n longhorn-system get pod
```

## Conclusion 

In most cases, of any problem visit longhorn website 

-  website : https://longhorn.io/docs/1.6.2/