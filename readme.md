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
- Drop in the scheduler through put and increase in latency

Kubernetes Internals

- What happens when I type `kubctl run`? https://github.com/jamiehannaford/what-happens-when-k8s
- Kubernetes networking: https://sookocheff.com/post/kubernetes/understanding-kubernetes-networking-model/
