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