# Debugging Kubernetes Pods

This section explains how to debug Kubernetes pods from the command line by getting information about available pods and checking the logs for specific pods. For more information about the `kubectl` CLI, see [Kubectl Reference Docs](https://kubernetes.io/docs/reference/generated/kubectl/kubectl-commands).


<a id="examine-pods"></a>
## Examine Available Pods

Before you begin the debugging process, examine all available pods to find pods with possible issue.

1. To see the status of all pods, use the `kubectl get pods` command. For example:

   ```bash
   kubectl get pods
   ```

   You can combine the `kubectl get pods` with other commands to get more information about a pod:
  
   * List pods for a specific namespace by using the `--namespace` flag followed by the namespace name.
     
   * Count the pods by _piping_ (passing) the output to the `wc` tool. For example:
       
     ```bash
     kubectl get pods | wc -l
     ```
     
   * Extract just the pod names by piping the output to `awk`. For example:
       
     ```bash
     kubectl get pods --no-headers=true | awk '{print $1}'
     ```
     
   * Filter only the pods in the `Running` state by piping the output to `grep`. For example:
       
     ```bash
     kubectl get pods | grep 'Running'
     ```
     
   * For more complex queries, add the `-o json` flag to parse the output to JSON and then pipe it to `jq`. For example:
       
     ```bash
     kubectl get pods -o json | jq .
     ```

2. For any pod that isn't in the `Running` state or has the `CrashLoopBackOff` or `Pending` state, use the `kubectl describe pod` command to get detailed information, such as environment variables, about a pod. For example:

   ```bash
   kubectl describe pod my-pod
   ```

<a id="kubectl-logs"></a>   
## Examine the Logs for a Pod

When you identify a pod with a potential issue, start debugging by examining its logs.

1. To view the logs for an affected pod, use the `kubectl logs` command followed by the pod name. For example:

   ```bash
   kubectl logs my-pod
   ```
  
   To view real-time logs, use the `-f` flag to _tail_ the logs (stream them in real time). For example:
     
   ```bash
   kubectl logs -f my-pod
   ```

2. The logs from a pod might be long and complex. You can take several approaches to examining the logs for a pod:

   * Add timestamps to the log file by using the `--timestamps` flag.
  
   * View the logs for a pod that has multiple containers by using the `--container` flag followed by the container name.
  
   * View the logs for a previously terminated container in the same pod by using the `--previous` flag.
  
     This can be useful for debugging crash loops.

   * Save the log files for offline analysis by redirecting the output to a file. For example:
     
     ```bash
     kubectl logs my-pod > my-log.txt
     ```

   * Search for specific log entries by piping the output to `grep`. For example:
     
     ```bash
     kubectl logs my-pod | grep 'ERROR'
     ```

   * To parse and visualize the logs, use tools such as [Grafana](https://grafana.com/), [Logstash](https://www.elastic.co/logstash), or [Splunk](https://www.splunk.com).

3. Once you identify specific issues or patterns of issues, consider taking steps to prepare your Kubernetes cluster for similar events in the future.
  
   * Collect logs from multiple pods or clusters by using services such as [Amazon CloudWatch](https://aws.amazon.com/cloudwatch/), [Fluentd](https://www.fluentd.org/), or the ELK stack ([Elasticsearch](https://www.elastic.co/elasticsearch/), [Logstash](https://www.elastic.co/logstash), and [Kibana](https://www.elastic.co/kibana).
  
   * Correlate logs with metrics or events from other sources to get a comprehensive view of system behaviour by using observability platforms such as [Datadog](https://www.datadoghq.com/) or [Grafana](https://grafana.com/).
  
   * Monitor logs in real time and receive alerts based on known patterns by using real-time monitoring solutions such as [Datadog](https://www.datadoghq.com/), [Prometheus](https://prometheus.io/), or [Zabbix](https://www.zabbix.com/).


<a id="kubectl-exec"></a>
## Examine the Pod's Container Environment

You can use the `kubectl exec` command to examine the contents of the pod's container environment by opening an interactive shell within the container. For example:

```bash
kubectl exec -it my-pod -- /bin/sh
```

You can replace `/bin/sh` with the shell or command that you want to use.

* Use the `ls` command to explore the file system within the container.
     
* Use the `ps` command to view the processes within the container.
  
* Use the `curl` and `ping` commands to test network connectivity from within the container.


<a id="debug-container"></a>
## Debug a Pod's Container

To isolate the debugging process from a production environment, you can create a debugging session by making a copy of an existing pod or by adding a new container for an existing pod.

>**Note:** When you finish the debugging process, remember to delete the debugging pod or container.

* To create a temporary copy of a pod, use the `kubectl debug` command and the `--copy-to` flag, followed by the name of the temporary copy. For example:

  ```bash
  kubectl debug my-pod --copy-to=my-debug-pod
  ```

  Specify pods in a specific namespace by using the `--namespace` flag followed by the namespace name.

* To begin an interactive debugging session, use the `kubectl debug` command, the `-it` flag, and the `--image` flag followed by the name of the Docker image that contains your debugging tools. For example:

  ```bash
  kubectl debug -it my-pod --image=my-docker-image
  ```

* To add a new container to an existing pod for debugging, use:
  
  * The `kubectl debug` command
  * The `--image` flag followed by the name of the Docker image that contains your debugging tools
  * The `--container` flag followed by the name of the debugging container

  For example:

  ```bash
  kubectl debug my-pod --image=my-docker-image --container=my-container
  ```

  * Override the entry point for the debug container by using `--`. For example: `-- /bin/sh`
    
  * Use the [`kubectl exec`](#kubectl-exec) and [`kubectl logs`](#kubectl-logs) commands within the running container.
