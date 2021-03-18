# tektondemo

To create in cluster registry in minikube follow below steps

``` minikube addons enable registry ```

Create aliases ConfigMap

```
apiVersion: v1
data:
  # Add additonal hosts seperated by new-line
  registryAliases: >-
    dev.local
    example.com
  # default registry address in minikube when enabled via minikube addons enable registry
  registrySvc: registry.kube-system.svc.cluster.local
kind: ConfigMap
metadata:
  name: registry-aliases
  namespace: kube-system
```

``` kubectl apply -f registry-aliases-config.yaml ```

Update Minikube /etc/hosts file

```
apiVersion: apps/v1
kind: DaemonSet
metadata:
  name: registry-aliases-hosts-update
  namespace: kube-system
  labels:
    kubernetes.io/minikube-addons: registry-aliases
    addonmanager.kubernetes.io/mode: Reconcile
spec:
  selector:
    matchLabels:
      app: registry-aliases-hosts-update
  template:
    metadata:
      labels:
        app: registry-aliases-hosts-update
    spec:
      initContainers:
        - name: update
          image: registry.fedoraproject.org/fedora
          volumeMounts:
            - name: etchosts
              mountPath: /host-etc/hosts
              readOnly: false
          env:
            - name: REGISTRY_ALIASES
              valueFrom:
                configMapKeyRef:
                  name: registry-aliases
                  key: registryAliases
          command:
            - bash
            - -ce
            - |
              NL=$'\n'
              TAB=$'\t'
              HOSTS="$(cat /host-etc/hosts)"
              [ -z "$REGISTRY_SERVICE_HOST" ] && echo "Failed to get hosts entry for default registry" && exit 1;
              for H in $REGISTRY_ALIASES; do                
                echo "$HOSTS" | grep "$H"  || HOSTS="$HOSTS$NL$REGISTRY_SERVICE_HOST$TAB$H";
              done;
              echo "$HOSTS" | diff -u /host-etc/hosts - || echo "$HOSTS" > /host-etc/hosts
              echo "Done."
      containers:
        - name: pause-for-update
          image: gcr.io/google_containers/pause-amd64:3.1
      terminationGracePeriodSeconds: 30
      volumes:
        - name: etchosts
          hostPath:
            path: /etc/hosts

```

``` kubectl apply -f node-etc-hosts-update.yaml ```

Once the DaemonSet is successfully running, you can check the Minkube VM’s /etc/hosts file, which will now be updated to point to CLUSTER-IP of the registry service.

``` minikube ssh -- cat /etc/hosts ```

In last edit the coredns configmap to add rewrite rules.

``` kubectl edit configmap -n kube-system coredns ```

it should look like this

``` kubectl get -n kube-sytem configmap coredns -o yaml ```

```
apiVersion: v1
data:
  Corefile: |-
    .:53 {
        errors
        health
    rewrite name dev.local  registry.kube-system.svc.cluster.local
    rewrite name example.com  registry.kube-system.svc.cluster.local
        kubernetes cluster.local in-addr.arpa ip6.arpa {
           pods insecure
           upstream
           fallthrough in-addr.arpa ip6.arpa
        }
        prometheus :9153
        forward . /etc/resolv.conf
        cache 30
        loop
        reload
        loadbalance
    }
kind: ConfigMap
metadata:
  name: coredns
  namespace: kube-system
```


1. Run the following command to install Tekton Pipelines and its dependencies:

```
kubectl apply --filename https://storage.googleapis.com/tekton-releases/pipeline/latest/release.yaml
```

2. Apply the created Tasks

```
kc apply -f 1-task.yaml
```

```
kc apply -f 2-task.yaml
```

3. Apply the build-Resources

```
kc apply -f 3-build-resources.yaml
```

4. Apply the 4-pipeline.yaml

```
kc apply -f 4-pipeline.yaml
```

5. Apply the 5-pipelineRun.yaml

```
kc apply -f 5-pipelineRun.yaml
```

6. Deployment and Service files will be applied during pipeline run.

7. For 8-ingress.yaml you can apply it manually or you can modify 2-task.yaml file to make pipeline deploy it for you.
