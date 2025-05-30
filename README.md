## Reflection: Hello Minikube

1. **Logs Before/After Exposing**  
   After exposing via `kubectl expose … --type=LoadBalancer`, I refreshed the page multiple times at the URL. Each refresh generated one new “Started HTTP server…” log line in the Pod logs, confirming the Service proxy is correctly routing traffic to my Pod.

2. **Namespaces with `-n`**  
   The `-n kube-system` flag tells `kubectl` to look in the `kube-system` namespace instead of `default`. That’s why my `hello-node` pods (in `default`) didn’t show up there—namespace isolation keeps system resources separate from user workloads.