---
title: "Kubernetes Scaling 2: Overprovisioning"
outputs: ["Reveal"]
featuredImage: "/overprovisioning_preview.png"
reveal_hugo:
  custom_theme: "reveal-hugo/themes/robot-lung.css"
  margin: 0.2
---

# Overprovisioning

---

### When does Cluster Autoscaler change the size of a cluster?

Cluster Autoscaler increases the size of the cluster when:

  * **there are pods that failed to schedule** on any of the current nodes due to insufficient resources.
  * adding a node similar to the nodes currently present in the cluster would help.

https://github.com/kubernetes/autoscaler/blob/master/cluster-autoscaler/FAQ.md#when-does-cluster-autoscaler-change-the-size-of-a-cluster

---

## Without overprovisioning

---

  1. `kubectl apply -f my-deployment.yml`
  2. Cannot schedule pods due to insufficient resources, deployment fails :skull:
  3. cluster-autoscaler notices and begins to provision new instance
  4. Wait for instance to be provisioned, boot, join the cluster and become ready :sleepy:
  5. kube-scheduler will notice there is somewhere to put the pods and will schedule them

---

  * Poor developer experience
  * Failed deployments
  * Cluster cannot handle any burst traffic

---

## Overprovisioning

>Allocating more computer data storage space than is strictly necessary
>- https://en.wikipedia.org/wiki/Overprovisioning

---

![Overprovisioning dashboard](/overprovisioning-dashboard.png)

---

  1. `kubectl apply -f my-deployment.yml`
  2. **Placeholder pods** are evicted, deployment is (_almost_) immediately succesful :tada:
  3. Placeholder pods cannot be scheduled due to insufficient resources
  4. Wait for instance to be provisioned, boot, join the cluster and become ready
  5. kube-scheduler will notice there is somewhere to put the placeholder pods and will schedule them

---

## Implementation

---

  * Enable `PodPriority` and `scheduling.k8s.io/v1alpha1`
  * Create a PriorityClass with a value of `-1`
  * Deploy placeholder pods and _proportional_ autoscaler

---

![Enabling PodPriority](/kops_overprovisioning.png)

---

```yaml
apiVersion: scheduling.k8s.io/v1alpha1
kind: PriorityClass
metadata:
  name: overprovisioning
value: -1
globalDefault: false
description: "Priority class used by overprovisioning."
```

---

```
apiVersion: apps/v1
kind: Deployment
metadata:
  name: overprovisioning-autoscaler
  namespace: default
  labels:
    app: overprovisioning-autoscaler
spec:
  selector:
    matchLabels:
      app: overprovisioning-autoscaler
  replicas: 1
  template:
    metadata:
      labels:
        app: overprovisioning-autoscaler
    spec:
      containers:
        - image: k8s.gcr.io/cluster-proportional-autoscaler-amd64:1.1.2
          name: autoscaler
          command:
            - /cluster-proportional-autoscaler
            - --namespace=default
            - --configmap=overprovisioning-autoscaler
            - --default-params={"linear":{"coresPerReplica":1}}  # this is the key part
            - --target=deployment/overprovisioning
            - --logtostderr=true
            - --v=2
      serviceAccountName: cluster-proportional-autoscaler-service-account
```

---

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: overprovisioning
  namespace: default
spec:
  replicas: 1
  selector:
    matchLabels:
      run: overprovisioning
  template:
    metadata:
      labels:
        run: overprovisioning
    spec:
      priorityClassName: overprovisioning
      containers:
      - name: reserve-resources
        image: k8s.gcr.io/pause
        resources:
          requests:
            cpu: "200m"
          limits:
            memory: "256Mi"  # not present by default
```

---

![Resource reservation](/resource-reservation.png)

---

# Thanks
