##**Bootstrapping an etcd cluster on kubernetes cluster on fedora**
This guide assumes that kubernetes cluster with one kubernetes master (fed-master) and atleast 3 kubernetes nodes (fed-node1, fed-node2 and fed-node3) is already set up as per the guide [here](https://github.com/GoogleCloudPlatform/kubernetes/blob/master/docs/getting-started-guides/fedora/fedora_manual_config.md). It also assumes that the etcd server, being used by kubernetes cluster, is running on fed-master, although it is not necessary as the etcd server could be running on a different host too (probably in a container).

* Clone this git repo:

```
#git clone https://github.com/aveshagarwal/etcd-cluster.git
```

* Build etcd image:

```
#cd etcd-cluster/etcd-image-build
#docker build -t etcd-cluster-image .
```

**Note:** It is assumed that etcd-cluster-image created above is accessible to all kubernetes nodes.

* The etcd server on fed-master will be used for bootstrapping this new etcd cluster by creating a discovery url. Run the following command on fed-master (or wherever the etcd server is running):

```
#curl -X PUT http://fed-master:4001/v2/keys/discovery/5c007a14875d53d9bf0ef5a6fc0257c817f0fb79/_config/size -d value=3
```

**Note:** 5c007a14875d53d9bf0ef5a6fc0257c817f0fb79 is just a random 40-byte string.

* Now create 3 etcd nodes by creating 3 replication controllers on the kubernetes cluster by copying json files from etcd-cluster/etcd-rc-jsons directory to the host fed-master.

```
# kubectl create -f etcd-rc-node1.json
# kubectl create -f etcd-rc-node2.json
# kubectl create -f etcd-rc-node3.json
```

**Note:** Please note that if you use different names for kubernetes nodes, or a different etcd image name or a different 40-byte string in the discovery url, you will have to update these json files according before running the above commands.

***

##**Testing the etcd cluster**

* On any kubernetes node (fed-node1, fed-node2, or fed-node3), check the health of the etcd cluster as follows:

```
# docker exec <container-id-of-etcd-node> etcdctl --peers "ip-address-of-etcd-container:4001" cluster-health
cluster is healthy
member 3f0e11cb228231fb is healthy
member c49c7e453f310e53 is healthy
member fa1b503888c0272d is healthy
```

***

##**Recovering the etcd cluster from failures**

* If one of the etcd containers is failed on any kubernetes nodes, the pod will create a new container (on the same node) with the same IP. And in this case, the etcd cluster will keep working without any issues.

* However, if a pod fails, then the replication controller (rc) will create a new pod which will in turn create a new etcd container (on the same node) possibly with a different IP address. In this case, it will require to inform the etcd cluster as follows:

```
# curl http://<ip-address-of-working-etcd-container>:2379/v2/members/<member-id-of-failed-etcd-node> -XPUT -H "Content-Type: application/json" -d '{"peerURLs":["http://<ip-address-of-the-new-container>:2380"]}'
```
