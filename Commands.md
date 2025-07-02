

- kubectl get pods -n open5gs-core - Lists all pods within the open5gs-core namespace.
    
- kubectl get deploy -n open5gs-core - Lists all deployments within the open5gs-core namespace.
    
- kubectl get svc -n open5gs-core - Lists all services within the open5gs-core namespace.
    
- kubectl get cm -n open5gs-core - Lists all ConfigMaps within the open5gs-core namespace.
    
- kubectl get cm core-amf -o yaml -n open5gs-core - Displays the full configuration of the 'core-amf' ConfigMap in YAML format.
	
- kubectl edit cm core-amf -o yaml -n open5gs-core - Opens the 'core-amf' ConfigMap in the default text editor for live modification.
    
- kubectl edit cm core-amf -o yaml -n open5gs-core - Opens the 'core-amf' ConfigMap in the default text editor for live modification. And after editing prints the final configuration.
    
- kubectl logs -f -n open5gs-core core-upf-5578846f46-psrpm - Streams the live logs (-f) from the 'core-upf-5578846f46-psrpm' pod.
    
- kubectl expose svc core-webui --type=NodePort --name=open5gs-webui --port - Creates a new service named 'open5gs-webui' to expose the 'core-webui' service on a port on each node. (Note: This command is incomplete as --port requires a value, e.g., --port=80).
    
- watch kubectl get pods -o wide -n oai-test - Continuously monitors and displays the status of pods in the oai-test namespace, refreshing every 2 seconds.
    
- kubectl get svc -A - Lists all services across all namespaces in the cluster.
    
- kubectl delete -f du-service.yaml -n oai-test - Deletes the Kubernetes resources defined in the du-service.yaml file.
    
- kubectl exec -it -n oai-test oai-nr-ue-75fc5bbf6c-79pg8 -- /bin/bash - Opens an interactive shell session inside the 'oai-nr-ue-75fc5bbf6c-79pg8' pod.
    
- kubectl apply -f oai-cucp.yaml -n oai-test - Creates or updates the Kubernetes resources defined in the oai-cucp.yaml file.
    
- kubectl delete ns oai-test - Deletes the namespace 'oai-test' and all resources contained within it.
    
- kubectl rollout restart deploy core-amf -n open5gs-core - Triggers a rolling restart of the 'core-amf' deployment's pods.
    
- kubectl get pods -n open5gs-core -o wide - Lists pods in the open5gs-core namespace with extra details like IP and node name.
    
- kubectl patch svc core-amf-ngap -n open5gs-core -p '{"spec":{"type":"NodePo - Partially updates the 'core-amf-ngap' service, likely to change its type to NodePort. (Note: This command is truncated).
    
- watch kubectl get pods -A - Continuously monitors and displays the status of all pods across all namespaces.
    
- kubectl describe pod core-amf-57cc44684d-wmn44 -n open5gs-core - Shows detailed information and recent events for the 'core-amf-57cc44684d-wmn44' pod.
    
- kubectl get networkpolicy --all-namespaces - Lists all NetworkPolicy resources across all namespaces.
    
- kubectl taint nodes open5gs-vm node-role.kubernetes.io/control-plane - Applies a taint to the 'open5gs-vm' node, which repels pods that don't tolerate it. (Note: This command is incomplete as it's missing the effect, e.g., NoSchedule).
    
- sudo systemctl restart containerd - Restarts the containerd container runtime service on the Linux node with root permissions.
    
- kubectl -n kube-system get configmap cilium-config -o yaml | grep -i cidr - Finds and displays lines containing "cidr" from the cilium-config ConfigMap.
    
- kubectl logs pod status calico-node-7755r -n calico-system - Displays the logs from the 'calico-node-7755r' pod in the calico-system namespace. (Note: The syntax is slightly incorrect; it should be kubectl logs calico-node-7755r -n calico-system).