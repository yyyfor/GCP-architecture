# Link
https://www.cloudskillsboost.google/course_templates/655/labs/464677

## In this lab, you will learn how to:

- Create a container-native load balancer through ingress
- Load test an application
- Configure liveness and readiness probes
- Create a pod disruption budget

## Provision testing environment
1. Set your default zone to "us-east1-c":
   `gcloud config set compute/zone us-east1-c`

3. Create a three node cluster:
`gcloud container clusters create test-cluster --num-nodes=3  --enable-ip-alias`
The --enable-ip-alias flag is included in order to enable the use of alias IPs for pods which will be required for container-native load balancing through an ingress.

For this lab, you'll use a simple HTTP web app that you will first deploy as a single pod.

4. Create a manifest for the gb-frontend pod:
```
cat << EOF > gb_frontend_pod.yaml
apiVersion: v1
kind: Pod
metadata:
  labels:
    app: gb-frontend
  name: gb-frontend
spec:
    containers:
    - name: gb-frontend
      image: gcr.io/google-samples/gb-frontend-amd64:v5
      resources:
        requests:
          cpu: 100m
          memory: 256Mi
      ports:
      - containerPort: 80
EOF
```
5. Apply the newly created manifest to your cluster:
   `kubectl apply -f gb_frontend_pod.yaml`

## Task 1. Container-native load balancing through ingress
1. The following manifest will configure a ClusterIP service that will be used to route traffic to your application pod to allow GKE to create a network endpoint group:
```
cat << EOF > gb_frontend_cluster_ip.yaml
apiVersion: v1
kind: Service
metadata:
  name: gb-frontend-svc
  annotations:
    cloud.google.com/neg: '{"ingress": true}'
spec:
  type: ClusterIP
  selector:
    app: gb-frontend
  ports:
  - port: 80
    protocol: TCP
    targetPort: 80
EOF
```
2. Apply the change to your cluster:
   `kubectl apply -f gb_frontend_cluster_ip.yaml`
3. Next, create an ingress for your application:
```
cat << EOF > gb_frontend_ingress.yaml
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: gb-frontend-ingress
spec:
  defaultBackend:
    service:
      name: gb-frontend-svc
      port:
        number: 80
EOF
```
4. Apply the change to your cluster:
`kubectl apply -f gb_frontend_ingress.yaml`
5. To check the health status of the backend service, first retrieve the name:
`BACKEND_SERVICE=$(gcloud compute backend-services list | grep NAME | cut -d ' ' -f2)`
6. Get the health status for the service:
`gcloud compute backend-services get-health $BACKEND_SERVICE --global`
7. Retrieve it with:
   `kubectl get ingress gb-frontend-ingress`
8. Entering the external IP in a browser window will load the application.

## Task 2. Load testing an application
To load test your pod, you'll use Locust, an open source load-testing framework.

1. Download the Docker image files for Locust in your Cloud Shell:
`gsutil -m cp -r gs://spls/gsp769/locust-image .`
The files in the provided locust-image directory include Locust configuration files.
2. Build the Docker image for Locust and store it in your project's container registry:
`gcloud builds submit \
   --tag gcr.io/${GOOGLE_CLOUD_PROJECT}/locust-tasks:latest locust-image`
3. Verify the Docker image is in your project's container registry:
   `gcloud container images list`
4. Copy and apply the manifest will create a single-pod deployment for the main and a 5-replica deployment for the workers:
`gsutil cp gs://spls/gsp769/locust_deploy_v2.yaml .
   sed 's/${GOOGLE_CLOUD_PROJECT}/'$GOOGLE_CLOUD_PROJECT'/g' locust_deploy_v2.yaml | kubectl apply -f -`
5. To access the Locust UI, retrieve the external IP address of its corresponding LoadBalancer service:
`kubectl get service locust-main`
6. In a new browser window, navigate to [EXTERNAL_IP_ADDRESS]:8089 to open the Locust web page:
7. For this example, to represent a typical load, enter 200 for the number of users to simulate and 20 for the hatch rate.

## Task 3. Readiness and liveness probes
1. To demonstrate a liveness probe, the following will generate a manifest for a pod that has a liveness probe based on the execution of the cat command on a file created on creation:
```
cat << EOF > liveness-demo.yaml
apiVersion: v1
kind: Pod
metadata:
  labels:
    demo: liveness-probe
  name: liveness-demo-pod
spec:
  containers:
  - name: liveness-demo-pod
    image: centos
    args:
    - /bin/sh
    - -c
    - touch /tmp/alive; sleep infinity
    livenessProbe:
      exec:
        command:
        - cat
        - /tmp/alive
      initialDelaySeconds: 5
      periodSeconds: 10
EOF
```
2. Apply the change to your cluster:
   `kubectl apply -f liveness-demo.yaml`
3. You can verify the health of the pod's container by checking the pod's events:
   `kubectl describe pod liveness-demo-pod`
4. Manually delete the file being used by the liveness probe:
   `kubectl exec liveness-demo-pod -- rm /tmp/alive`
5. With the file removed, the cat command being used by the liveness probe should return a non-zero exit code.

6. Once again, check the pod's events:

`kubectl describe pod liveness-demo-pod`

### Setting up a readiness probe
1. To demonstrate this, create a manifest to create a single pod that will serve as a test web server along with a load balancer:
```
cat << EOF > readiness-demo.yaml
apiVersion: v1
kind: Pod
metadata:
  labels:
    demo: readiness-probe
  name: readiness-demo-pod
spec:
  containers:
  - name: readiness-demo-pod
    image: nginx
    ports:
    - containerPort: 80
    readinessProbe:
      exec:
        command:
        - cat
        - /tmp/healthz
      initialDelaySeconds: 5
      periodSeconds: 5
---
apiVersion: v1
kind: Service
metadata:
  name: readiness-demo-svc
  labels:
    demo: readiness-probe
spec:
  type: LoadBalancer
  ports:
    - port: 80
      targetPort: 80
      protocol: TCP
  selector:
    demo: readiness-probe
EOF
```
2. Apply the manifest to your cluster and create a load balancer with it:
   `kubectl apply -f readiness-demo.yaml`
3. Retrieve the external IP address assigned to your load balancer (it may take a minute after the previous command for an address to be assigned):
   `kubectl get service readiness-demo-svc`
5. Check the pod's events:

`kubectl describe pod readiness-demo-pod`
6. Use the following command to generate the file that the readiness probe is checking for:
   `kubectl exec readiness-demo-pod -- touch /tmp/healthz`

## Task 4. Pod disruption budgets
First, you'll have to deploy your application as a deployment.

1. Delete your single pod app:
`kubectl delete pod gb-frontend`
2. And generate a manifest that will create it as a deployment of 5 replicas:
```
cat << EOF > gb_frontend_deployment.yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: gb-frontend
  labels:
    run: gb-frontend
spec:
  replicas: 5
  selector:
    matchLabels:
      run: gb-frontend
  template:
    metadata:
      labels:
        run: gb-frontend
    spec:
      containers:
        - name: gb-frontend
          image: gcr.io/google-samples/gb-frontend-amd64:v5
          resources:
            requests:
              cpu: 100m
              memory: 128Mi
          ports:
            - containerPort: 80
              protocol: TCP
EOF
```
3. Apply this deployment to your cluster:
   `kubectl apply -f gb_frontend_deployment.yaml`
4. Drain the nodes by looping through the output of the default-pool's nodes and running the kubectl drain command on each individual node:
`for node in $(kubectl get nodes -l cloud.google.com/gke-nodepool=default-pool -o=name); do
   kubectl drain --force --ignore-daemonsets --grace-period=10 "$node";
   done`
5. Once your node has been drained, check in on your gb-frontend deployment's replica count:
   `kubectl describe deployment gb-frontend | grep ^Replicas`
6. First bring the drained nodes back by uncordoning them. The command below allows pods to be scheduled on the node again:
`for node in $(kubectl get nodes -l cloud.google.com/gke-nodepool=default-pool -o=name); do
   kubectl uncordon "$node";
   done`
7. Check the status of your deployment:
   `kubectl describe deployment gb-frontend | grep ^Replicas`
8. Create a pod disruption budget that will declare the minimum number of available pods to be 4:
`kubectl create poddisruptionbudget gb-pdb --selector run=gb-frontend --min-available 4`
9. Once again, drain one of your cluster's nodes and observe the output:
`for node in $(kubectl get nodes -l cloud.google.com/gke-nodepool=default-pool -o=name); do
   kubectl drain --timeout=30s --ignore-daemonsets --grace-period=10 "$node";
   done`