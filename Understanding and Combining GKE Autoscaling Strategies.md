# Link
https://www.cloudskillsboost.google/course_templates/655/labs/464674

## Objectives
In this lab, you will learn how to:

- Decrease number of replicas for a Deployment with Horizontal Pod Autoscaler
- Decrease CPU request of a Deployment with Vertical Pod Autoscaler
- Decrease number of nodes used in cluster with Cluster Autoscaler
- Automatically create an optimized node pool for workload with Node Auto Provisioning
- Test the autoscaling behavior against a spike in demand
- Overprovision your cluster with Pause Pods

## Provision testing environment
1. Set your default zone to us-east1-d:
`gcloud config set compute/zone us-east1-d`
2. Run the following command to create a three node cluster in the us-east1-d zone:
`gcloud container clusters create scaling-demo --num-nodes=3 --enable-vertical-pod-autoscaling`
3. Create a manifest for the php-apache deployment:
```
cat << EOF > php-apache.yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: php-apache
spec:
  selector:
    matchLabels:
      run: php-apache
  replicas: 3
  template:
    metadata:
      labels:
        run: php-apache
    spec:
      containers:
      - name: php-apache
        image: k8s.gcr.io/hpa-example
        ports:
        - containerPort: 80
        resources:
          limits:
            cpu: 500m
          requests:
            cpu: 200m
---
apiVersion: v1
kind: Service
metadata:
  name: php-apache
  labels:
    run: php-apache
spec:
  ports:
  - port: 80
  selector:
    run: php-apache
EOF
```
4. Apply the newly created manifest to your cluster:
`kubectl apply -f php-apache.yaml`

## Task 1. Scale pods with Horizontal Pod Autoscaling
1. In Cloud Shell, run this command to inspect your cluster's deployments:
kubectl get deployment
2. Apply horizontal autoscaling to the php-apache deployment:
`kubectl autoscale deployment php-apache --cpu-percent=50 --min=1 --max=10`
3. Check the current status of your Horizontal Pod Autoscaler:
`kubectl get hpa`

## Task 2. Scale size of pods with Vertical Pod Autoscaling
1. To verify run:
`gcloud container clusters describe scaling-demo | grep ^verticalPodAutoscaling -A 1` 
2. Apply the hello-server deployment to your cluster:
`kubectl create deployment hello-server --image=gcr.io/google-samples/hello-app:1.0`
3. Ensure the deployment was successfully created:
`kubectl get deployment hello-server`
4. Assign a CPU resource request of 450m to the deployment:
`kubectl set resources deployment hello-server --requests=cpu=450m`
5. Next, run this command to inspect the container specifics of the hello-server pods:
`kubectl describe pod hello-server | sed -n "/Containers:$/,/Conditions:/p"`
6. Now, create a manifest for you Vertical Pod Autoscaler:
```
cat << EOF > hello-vpa.yaml
apiVersion: autoscaling.k8s.io/v1
kind: VerticalPodAutoscaler
metadata:
  name: hello-server-vpa
spec:
  targetRef:
    apiVersion: "apps/v1"
    kind:       Deployment
    name:       hello-server
  updatePolicy:
    updateMode: "Off"
EOF
```
7. Apply the manifest for hello-vpa:
`kubectl apply -f hello-vpa.yaml`
8. Wait a minute, and then view the VerticalPodAutoscaler:
`kubectl describe vpa hello-server-vpa`
9. In order to observe VPA and its effects within this lab, you will change the hello-vpa update policy to Auto and observe the scaling.
10. Update the manifest to set the policy to Auto and apply the configuration:
```
sed -i 's/Off/Auto/g' hello-vpa.yaml
kubectl apply -f hello-vpa.yaml
```
11. Scale hello-server deployment to 2 replicas:
`kubectl scale deployment hello-server --replicas=2`
12. Now, watch your pods:
`kubectl get pods -w`
13. Wait until you see your hello-server-xxx pods in the terminating or pending status (or navigate to Kubernetes Engine > Workloads):

## Task 3. HPA results
1. Run this command to check your HPA:
`kubectl get hpa` 

## Task 4. VPA results
1. Inspect your pods:
`kubectl describe pod hello-server | sed -n "/Containers:$/,/Conditions:/p"`

## Task 5. Cluster autoscaler
1. Enable autoscaling for your cluster:
`gcloud beta container clusters update scaling-demo --enable-autoscaling --min-nodes 1 --max-nodes 5`
2. Switch to the optimize-utilization autoscaling profile so that the full effects of scaling can be observed:
`gcloud beta container clusters update scaling-demo \
--autoscaling-profile optimize-utilization`
3. With autoscaling enabled, observe your cluster in the Cloud Console. Click the three bars at the top left to open the Navigation menu.
4. From the Navigation menu, select Kubernetes Engine > Clusters.
5. On the Clusters page, select the scaling-demo cluster.
6. In the scaling-demo's cluster page, select the Nodes tab.
7. Take a look at the overview of your three nodes' resource utilization.
8. This can be verified by running this command in Cloud Shell:
`kubectl get deployment -n kube-system` 
9. Run these commands to create the Pod Disruption Budgets for each of your kube-system pods:
```
kubectl create poddisruptionbudget kube-dns-pdb --namespace=kube-system --selector k8s-app=kube-dns --max-unavailable 1
kubectl create poddisruptionbudget prometheus-pdb --namespace=kube-system --selector k8s-app=prometheus-to-sd --max-unavailable 1
kubectl create poddisruptionbudget kube-proxy-pdb --namespace=kube-system --selector component=kube-proxy --max-unavailable 1
kubectl create poddisruptionbudget metrics-agent-pdb --namespace=kube-system --selector k8s-app=gke-metrics-agent --max-unavailable 1
kubectl create poddisruptionbudget metrics-server-pdb --namespace=kube-system --selector k8s-app=metrics-server --max-unavailable 1
kubectl create poddisruptionbudget fluentd-pdb --namespace=kube-system --selector k8s-app=fluentd-gke --max-unavailable 1
kubectl create poddisruptionbudget backend-pdb --namespace=kube-system --selector k8s-app=glbc --max-unavailable 1
kubectl create poddisruptionbudget kube-dns-autoscaler-pdb --namespace=kube-system --selector k8s-app=kube-dns-autoscaler --max-unavailable 1
kubectl create poddisruptionbudget stackdriver-pdb --namespace=kube-system --selector app=stackdriver-metadata-agent --max-unavailable 1
kubectl create poddisruptionbudget event-pdb --namespace=kube-system --selector k8s-app=event-exporter --max-unavailable 1
```
10. Rerun this command in Cloud Shell until you see only two nodes total:
`kubectl get nodes`

## Task 6. Node Auto Provisioning
Enable Node Auto Provisioning:
```
gcloud container clusters update scaling-demo \
    --enable-autoprovisioning \
    --min-cpu 1 \
    --min-memory 2 \
    --max-cpu 45 \
```

## Task 7. Test with larger demand
2. In the new tab, run this command to send an infinite loop of queries to the php-apache service:
   `kubectl run -i --tty load-generator --rm --image=busybox --restart=Never -- /bin/sh -c "while sleep 0.01; do wget -q -O- http://php-apache; done"`
4. `kubectl get hpa` 
5. Now, monitor how your cluster handles the increased load by periodically running this command:
`kubectl get deployment php-apache`

## Task 8. Optimize larger loads
1. Create a manifest for a pause pod:
```
cat << EOF > pause-pod.yaml
---
apiVersion: scheduling.k8s.io/v1
kind: PriorityClass
metadata:
  name: overprovisioning
value: -1
globalDefault: false
description: "Priority class used by overprovisioning."
---
apiVersion: apps/v1
kind: Deployment
metadata:
  name: overprovisioning
  namespace: kube-system
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
            cpu: 1
            memory: 4Gi
EOF
```
2. Apply it to your cluster:
`kubectl apply -f pause-pod.yaml`
3. Now, wait a minute and then refresh the nodes tab of your scaling-demo cluster.