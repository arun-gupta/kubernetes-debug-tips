# Kubernetes Debugging Tips

- Debug pods: https://kubernetes.io/docs/tasks/debug-application-cluster/debug-pod-replication-controller/
- Debug Services: https://kubernetes.io/docs/tasks/debug-application-cluster/debug-service/
- Debug cluster: https://kubernetes.io/docs/tasks/debug-application-cluster/debug-cluster/
- Network partitioning: `tcp_retries2` from https://medium.com/hotels-com-technology/improving-kubernetes-responsiveness-under-network-partitioning-df00403a6d97
- SLIs/SLOs: https://github.com/kubernetes/community/blob/master/sig-scalability/slos/slos.md
- Kubectl latency is too high?
  - What is the SLO?
  - What instance type is used for etcd? IOPS-bound?
  - Use an SSD locally attached to the node instead of over-network?
- How do we make sure etcd scales? How big is the typical database? What are the limits around it?
  - For large clusters, configure API server to store events in a dedicated etcd instance
- Building large k8s clusters: https://www.youtube.com/watch?v=kDwQ991NCkg
- Why HPA is not scaling pods?
- Why Cluster Autoscaler is not not scaling cluster?
- Use IPVS (hashtable) instead of IPTable (linear list of routing rules)
- Drop in scheduler through put and increase in latency for large clusters
- etcd
  - 3 etcd?
  - what is the typical database size?
  - how do we size?
- [Updating cluster](http://arun-gupta.github.io/update-eks-cluster/)
  - version skew policy: https://kubernetes.io/docs/setup/release/version-skew-policy/
  	- don't jump two versions at one go in control plane
  	- control plane should always be updated first
  - what if kubectl start sending requests that are not yet honored by data plane
  - what if update fails mid way?
  - what is recommended? migrate to a new node group or update existing one?
  - `eksctl update cluster` is reliable? or should we recommend creating a new cluster, migrating the workloads, and using DNS to switch?


Kubernetes Internals

- What happens when I type `kubctl run`? https://github.com/jamiehannaford/what-happens-when-k8s
  - client-side validation
  - authentication
  - prepare and send REST request
  - API server authenticates and authorizes
  - admission controllers
  - persist resource in etcd
  - run initializers
  - run the controllers
  - schedule to a node
  - kubelet generates pod status
  - kubelet creates cgroups and namespaces
  - kubelet uses CNI to setup networking
  - kubelet create containers using CRI  
- Kubernetes networking: https://sookocheff.com/post/kubernetes/understanding-kubernetes-networking-model/
