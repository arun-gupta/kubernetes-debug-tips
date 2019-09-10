# Kubernetes Debugging Tips

- Debug pods: https://kubernetes.io/docs/tasks/debug-application-cluster/debug-pod-replication-controller/
- Debug Services: https://kubernetes.io/docs/tasks/debug-application-cluster/debug-service/
- Debug cluster: https://kubernetes.io/docs/tasks/debug-application-cluster/debug-cluster/
- Network partitioning: `tcp_retries2` from https://medium.com/hotels-com-technology/improving-kubernetes-responsiveness-under-network-partitioning-df00403a6d97
- Troubleshoot k8s deployments: https://docs.bitnami.com/kubernetes/how-to/troubleshoot-kubernetes-deployments/
- Kubernetes recipes: maintenance and troubleshooting: https://www.oreilly.com/ideas/kubernetes-recipes-maintenance-and-troubleshooting
- 10 Most Common Reasons Kubernetes Deployments Fail: https://kukulinski.com/10-most-common-reasons-kubernetes-deployments-fail-part-1/
- SLIs/SLOs: https://github.com/kubernetes/community/blob/master/sig-scalability/slos/slos.md
- Services `curl: (52) Empty reply from server`:
  - Check service:

    ```
    kubectl get svc
    NAME                            TYPE           CLUSTER-IP       EXTERNAL-IP                                                              PORT(S)        AGE
    kubernetes                      ClusterIP      10.100.0.1       <none>                                                                   443/TCP        7h1m
    myapp-greeting                  LoadBalancer   10.100.144.214   ae8edb3e298f211e9b81a02ee809dbcd-201517527.us-west-2.elb.amazonaws.com   80:30947/TCP   63s
    ```

  - Check pods:

    ```
    kubectl get pods
    NAME                                             READY   STATUS         RESTARTS   AGE
    myapp-greeting-84df4cf76-8mn5b                   0/1     ErrImagePull   0          117s
    ```

    Diagnosis: Check the image name and tag, make sure its on the registry, does the registry need credentials?

    OR

    ```
    NAME                                             READY   STATUS             RESTARTS   AGE
    myapp-greeting-84df4cf76-fs5hr                   0/1     CrashLoopBackOff   2          44s
    ```

  - Describe pod:

    ```
    kubectl describe pod/myapp-greeting-84df4cf76-fs5hr
    ...
    Events:
    Type     Reason     Age               From                                                  Message
    ----     ------     ----              ----                                                  -------
    Normal   Scheduled  56s               default-scheduler                                     Successfully assigned default/myapp-greeting-84df4cf76-fs5hr to ip-192-168-2-219.us-west-2.compute.internal
    Normal   Pulling    55s               kubelet, ip-192-168-2-219.us-west-2.compute.internal  pulling image "arungupta/greeting:prom"
    Normal   Pulled     50s               kubelet, ip-192-168-2-219.us-west-2.compute.internal  Successfully pulled image "arungupta/greeting:prom"
    Normal   Created    2s (x4 over 49s)  kubelet, ip-192-168-2-219.us-west-2.compute.internal  Created container
    Normal   Started    2s (x4 over 48s)  kubelet, ip-192-168-2-219.us-west-2.compute.internal  Started container
    Normal   Pulled     2s (x3 over 48s)  kubelet, ip-192-168-2-219.us-west-2.compute.internal  Container image "arungupta/greeting:prom" already present on machine
    Warning  BackOff    2s (x5 over 47s)  kubelet, ip-192-168-2-219.us-west-2.compute.internal  Back-off restarting failed container
    ```

    - Check pod logs:

      ```
      ```

    - Run Docker image:

      ```
      docker container run -p 80:8080 -it arungupta/greeting:prom
      root@f88d9c61ee6c:/# 
      ```

    - Check build output:

      ```
      [INFO] Containerizing application to Docker daemon as arungupta/greeting:prom...
      [INFO] The base image requires auth. Trying again for openjdk:8-jre...
      [INFO] Container program arguments set to [bash] (inherited from base image)
      ```

    Diagnosis: `pom.xml` uses `jib` for building the Docker image and `war` as `<packaging>`. So either add `ServletInitializer` and exclude Tomcat embedded JAR, or change `<packaging>` to `jar`. Alternaitvely, use Dockerfile to create your image.

- Accessing service gives:

  ```
  curl http://$(kubectl get svc/myapp-greeting \
     -o jsonpath='{.status.loadBalancer.ingress[0].hostname}')/hello
  curl: (6) Could not resolve host: a87d6823b994311e9b81a02ee809dbcd-1223624547.us-west-2.elb.amazonaws.com
  ```

  Diagonsis: Takes about 2-3 mins for the ELB to be available.

- kubectl not working, specifically giving error `Unable to connect to the server: net/http: TLS handshake timeout`
  - check Internet connection
  - check KUBECONFIG
  - check context `kubectl config get-contexts`
  - API server down?
    - Get API server address
    - POST `<address>:8080/healthz` or `<address>:8080/healthz/ping` ??
  - check kubectl skew, supporte +/-1 version of API server
- Kubectl latency is too high?
  - What is the SLO?
  - check API server logs from Container Insights
  - What instance type is used for etcd? IOPS-bound?
  - Use an SSD locally attached to the node instead of over-network?
- How do we make sure etcd scales? How big is the typical database? What are the limits around it?
  - For large clusters, configure API server to store events in a dedicated etcd instance
- Building large k8s clusters: https://www.youtube.com/watch?v=kDwQ991NCkg
- Why HPA is not scaling pods?
  - Is metrics-server installed?
- Why is Cluster Autoscaler not scaling the cluster?
  - Is it even installed? 
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

- How does 'kubectl exec' work? https://erkanerol.github.io/post/how-kubectl-exec-works/
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
