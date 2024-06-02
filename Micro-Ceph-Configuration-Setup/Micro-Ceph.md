## MicroCeph Configuration and setup with microk8s cluster.

MicroCeph is focused on the small scale. It simplifies deploying and operating a Ceph cluster, making it easier to consume. The solution fits small but growing clusters, testing environments, home labs, and development instances. It is offering minimal overhead, quick and predictable deployments alongside straightforward operations for bootstrapping, adding OSDs, enabling, and placing services.

## Prerequisites

- MicroK8s: Ensure you have a MicroK8s cluster set up and running.
- MicroCeph: Have at least one MicroCeph node ready for integration.
- kubectl: The Kubernetes command-line tool should be installed and configured to interact with your MicroK8s cluster.

## Setup and Configuration 

## Single Node

MicroCeph can scale down to a single node with 3 disks and 4GB of memory.

- Note MicroCeph will use complete disk drives, it doesn't work with disk partitions.

## Installation and bootstrapping a single node cluster 

- Install the software

- Install the most recent stable release of MicroCeph:

```
sudo snap install microceph

```
-Next, prevent the software from being auto-updated:
```
sudo snap refresh --hold microceph
```


## Initialise the cluster

- Begin by initialising the cluster with the cluster bootstrap command:

```
sudo microceph cluster bootstrap

```
- Then look at the status of the cluster with the status command:
```
sudo microceph status
```
- It should look similar to the following:
```
MicroCeph deployment summary:
- node-mees (10.246.114.49)
    Services: mds, mgr, mon
      Disks: 0
```

- Here, the machine’s hostname of ‘node-mees’ is given along with its IP address of ‘10.246.114.49’. The MDS, MGR, and MON services are running but there is not yet any storage available.


## Add storage

- Three OSDs will be required to form a minimal Ceph cluster. In a production system, typically we would assign a physical block device to an OSD. However for this tutorial, we will make use of file backed OSDs for simplicity.

- Add the three file-backed OSDs to the cluster by using the disk add command. In the example, three 4GiB files are being created:

```
sudo microceph disk add loop,4G,3
```

- Recheck status:
```
sudo microceph status

```
- The output should now show three disks and the additional presence of the OSD service:
```
MicroCeph deployment summary:
- node-mees (10.246.114.49)
    Services: mds, mgr, mon, osd
      Disks: 3
```
## Manage the cluster

- Your Ceph cluster is now deployed and can be managed by following the resources found in the How-to section.

- The cluster can also be managed using native Ceph tooling if snap-level commands are not yet available for a desired task:

```
sudo ceph status
```
- The cluster built during this tutorial gives the following output:

```
cluster:
  id:     4c2190cd-9a31-4949-a3e6-8d8f60408278
  health: HEALTH_OK

services:
  mon: 1 daemons, quorum node-mees (age 7d)
  mgr: node-mees(active, since 7d)
  osd: 3 osds: 3 up (since 7d), 3 in (since 7d)

data:
  pools:   1 pools, 1 pgs
  objects: 2 objects, 577 KiB
  usage:   96 MiB used, 2.7 TiB / 2.7 TiB avail
  pgs:     1 active+clean

```

## Connect MicroCeph to MicroK8s Cluster 

- The rook-ceph addon first appeared with the 1.28 release, so we should select a MicroK8s deployment channel greater or equal to 1.28:
```

sudo snap install microk8s --channel=1.28/stable

sudo microk8s status --wait-ready

```
- Note: Before enabling the rook-ceph addon on a strictly confined MicroK8s, make sure the rbd kernel module is loaded with sudo modprobe rbd.

- The output message of enabling the addon, sudo microk8s enable rook-ceph, describes what the next steps should be to import a Ceph cluster:

```
Infer repository core for addon rook-ceph                                                                                                                                                                                                   
Add Rook Helm repository https://charts.rook.io/release  rook-release" has been added to your repositories               

```
- Rook Ceph operator v1.11.9 is now deployed in your MicroK8s cluster and
will shortly be available for use.

- As a next step, you can either deploy Ceph on MicroK8s, or connect MicroK8s with an
existing Ceph cluster.

- To connect MicroK8s with an existing Ceph cluster, you can use the helper command 'microk8s connect-external-ceph'. If you are running MicroCeph on the same node, then
you can use the following command:

```
    sudo microk8s connect-external-ceph

```
- Alternatively, you can connect MicroK8s with any external Ceph cluster using:

```
    sudo microk8s connect-external-ceph \
        --ceph-conf /path/to/cluster/ceph.conf \
        --keyring /path/to/cluster/ceph.keyring \
        --rbd-pool microk8s-rbd
```

If the above command gives  the following error.
```
  File "/var/snap/microk8s/common/plugins/connect-external-ceph", line 184, in <module>
    main()
  File "/snap/microk8s/6809/usr/lib/python3/dist-packages/click/core.py", line 764, in __call__
    return self.main(*args, **kwargs)
  File "/snap/microk8s/6809/usr/lib/python3/dist-packages/click/core.py", line 717, in main
    rv = self.invoke(ctx)
  File "/snap/microk8s/6809/usr/lib/python3/dist-packages/click/core.py", line 956, in invoke
    return ctx.invoke(self.callback, **ctx.params)
  File "/snap/microk8s/6809/usr/lib/python3/dist-packages/click/core.py", line 555, in invoke
    return callback(*args, **kwargs)
  File "/var/snap/microk8s/common/plugins/connect-external-ceph", line 166, in main
    ceph_initialize_rbd_pool(ceph_conf, keyring, rbd_pool)
  File "/var/snap/microk8s/common/plugins/connect-external-ceph", line 67, in ceph_initialize_rbd_pool
    client = rados.Rados(conffile=ceph_conf, conf={"keyring": keyring})
  File "rados.pyx", line 709, in rados.Rados.__init__
  File "rados.pyx", line 595, in rados.requires.wrapper.validate_func
  File "rados.pyx", line 766, in rados.Rados.__setup
  File "rados.pyx", line 595, in rados.requires.wrapper.validate_func
  File "rados.pyx", line 849, in rados.Rados.conf_read_file
rados.ObjectNotFound: [errno 2] RADOS object not found (error calling conf_read_file)
```
- Above error solution.
```
Copy the files ceph.config and ceph.keyring files to the master node and the run the  command with the master node path 

sudo microk8s connect-external-ceph \
        --ceph-conf /path/to/cluster/ceph.conf \
        --keyring /path/to/cluster/ceph.keyring \
        --rbd-pool microk8s-rbd
```


- For a list of all supported options, use helper command.
```
microk8s connect-external-ceph --help
```
- To deploy Ceph on the MicroK8s cluster using storage from your Kubernetes nodes, refer to https://rook.io/docs/rook/latest-release/CRDs/Cluster/ceph-cluster-crd/

- As we have already setup MicroCeph having it managed by rook is done with just:
```
sudo microk8s connect-external-ceph
```
- At the end of this process you should have a storage class ready to use:
```
NAME       PROVISIONER                  RECLAIMPOLICY   VOLUMEBINDINGMODE   ALLOWVOLUMEEXPANSION   AGE
ceph-rbd   rook-ceph.rbd.csi.ceph.com   Delete          Immediate           true                   3h38m

```

## For more detail visit the following links 
1. https://canonical-microceph.readthedocs-hosted.com/en/reef-stable/tutorial/single-node/.
2. https://microk8s.io/docs/how-to-ceph.
3. https://sabaini.at/peterlog/posts/2023/Aug/10/introducing-microceph/.
