# Link
https://www.cloudskillsboost.google/course_templates/655/labs/464671

## Task 2. Choosing the right machine type for the Hello app

### Scale up Hello app
#### Access your cluster's credentials:
`gcloud container clusters get-credentials hello-demo-cluster --zone us-west1-c`

#### Scale up your Hello-Server:
`kubectl scale deployment hello-server --replicas=2`

#### Increase your node pool to handle your new request:
`gcloud container clusters resize hello-demo-cluster --node-pool my-node-pool \
--num-nodes 3 --zone us-west1-c`

### Migrate to optimized node pool
#### Create a new node pool with a larger machine type:
`gcloud container node-pools create larger-pool \
--cluster=hello-demo-cluster \
--machine-type=e2-standard-2 \
--num-nodes=1 \
--zone=us-west1-c`

#### you can migrate pods to the new node pool by following these steps:

1. Cordon the existing node pool: This operation marks the nodes in the existing node pool (node) as unschedulable. Kubernetes stops scheduling new Pods to these nodes once you mark them as unschedulable.
2. Drain the existing node pool: This operation evicts the workloads running on the nodes of the existing node pool (node) gracefully.

##### First, cordon the original node pool:
```
for node in $(kubectl get nodes -l cloud.google.com/gke-nodepool=my-node-pool -o=name); do
kubectl cordon "$node";
done
```
##### Next, drain the pool:
```
for node in $(kubectl get nodes -l cloud.google.com/gke-nodepool=my-node-pool -o=name); do
kubectl drain --force --ignore-daemonsets --delete-local-data --grace-period=10 "$node";
done
```
At this point, you should see that your pods are running on the new, larger-pool, node pool:
```
kubectl get pods -o=wide
```
With the pods migrated, it's safe to delete the old node pool:
```
gcloud container node-pools delete my-node-pool --cluster hello-demo-cluster --zone us-west1-c
```
When asked to continue, type y and enter.

## Task 3. Managing a regional cluster
you'll observe the network traffic of your cluster and move two chatty pods, pods which are generating a lot of traffic to one another, to be in the same zone.

1. In your Cloud Shell tab, create a new regional cluster (this command will take a few minutes to complete):
`gcloud container clusters create regional-demo --region=us-west1 --num-nodes=1`
In order to demonstrate traffic between your pods and nodes, you will create two pods on separate nodes in your regional cluster. We will use ping to generate traffic from one pod to the other to generate traffic which we can then monitor.

2. Run this command to create a manifest for your first pod:
```
cat << EOF > pod-1.yaml
apiVersion: v1
kind: Pod
metadata:
name: pod-1
labels:
security: demo
spec:
containers:
- name: container-1
  image: wbitt/network-multitool
  EOF
```
  
3. Create the first pod in Kubernetes by using this command:
  `kubectl apply -f pod-1.yaml`
4. Next, run this command to create a manifest for your second pod:
```
cat << EOF > pod-2.yaml
apiVersion: v1
kind: Pod
metadata:
name: pod-2
spec:
affinity:
podAntiAffinity:
requiredDuringSchedulingIgnoredDuringExecution:
  - labelSelector:
    matchExpressions:
      - key: security
        operator: In
        values:
          - demo
            topologyKey: "kubernetes.io/hostname"
            containers:
- name: container-2
  image: gcr.io/google-samples/node-hello:1.0
  EOF
```
5. Create the second pod in Kubernetes:
  `kubectl apply -f pod-2.yaml`
6. View the pods you created:
   `kubectl get pod pod-1 pod-2 --output wide`