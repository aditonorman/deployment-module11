## Reflection: Hello Minikube

1. **Logs Before/After Exposing**  
   After exposing via `kubectl expose … --type=LoadBalancer`, I refreshed the page multiple times at the URL. Each refresh generated one new “Started HTTP server…” log line in the Pod logs, confirming the Service proxy is correctly routing traffic to my Pod.

2. **Namespaces with `-n`**  
   The `-n kube-system` flag tells `kubectl` to look in the `kube-system` namespace instead of `default`. That’s why my `hello-node` pods (in `default`) didn’t show up there—namespace isolation keeps system resources separate from user workloads.


3. **Recreate strategy**  
   - **Preparation:** I created `deployment-recreate.yaml` including:
     ```yaml
     strategy:
       type: Recreate
     ```
   - **Apply the manifest:**
     ```bash
     kubectl apply -f deployment-recreate.yaml
     ```
   - **Observation (pod deletion):**  
     As soon as the new manifest was applied, Kubernetes terminated **all** existing `spring-petclinic-rest` pods in parallel. I ran:
     ```bash
     kubectl get pods -w
     ```
     and saw each old pod enter the `Terminating` state (respecting the 30 s `terminationGracePeriodSeconds`) before being removed.
   - **Observation (pod creation):**  
     Immediately after the old pods cleared, the Deployment controller spun up four brand-new pods. They all moved through `Pending → ContainerCreating → Running` together, each reaching `1/1 READY` in roughly 8–12 seconds.
   - **Service downtime:**  
     During the interval when no pods were available, hitting the `/petclinic` endpoint returned connection errors (e.g. 502/504). Once the new pods were up, traffic resumed normally.
   - **Key takeaway:**  
     The Recreate strategy guarantees that you never have mixed-version pods running simultaneously, but at the cost of a brief downtime window—unlike RollingUpdate, which would have kept some pods available throughout the upgrade.
