# Link
https://www.cloudskillsboost.google/course_templates/655/labs/464678#step6

## Task 1
- set default zone
`gcloud config set compute/zone us-west1-c`
- create a zonal cluster with only two (2) nodes having machine size e2-standard-2.
`gcloud container clusters create onlineboutique-cluster-492 --num-nodes=2 --machine-type=e2-standard-2`
- create namespace
`kubectl create namespace dev && \
kubectl create namespace prod` 
- deploy application to dev
`git clone https://github.com/GoogleCloudPlatform/microservices-demo.git &&
cd microservices-demo && kubectl apply -f ./release/kubernetes-manifests.yaml --namespace dev`

## Task 2
1. login
`gcloud config set compute/zone ${ZONE} && gcloud container clusters get-credentials onlineboutique-cluster-492`
2. Create a new node pool named optimized-pool-3563 with custom-2-3584 as the machine type
   `gcloud container node-pools create optimized-pool-3563 \
   --cluster=onlineboutique-cluster-492 \
   --machine-type=custom-2-3584 \
   --num-nodes=1 \
   --zone=us-west1-c`
3. Set the number of nodes to 2
    `gcloud container clusters resize onlineboutique-cluster-492 --node-pool optimized-pool-3563 --num-nodes 2 --zone us-west1-c`
4. Once the new node pool is set up, migrate your application's deployments to the new nodepool by cordoning off and draining default-pool
    `for node in $(kubectl get nodes -l cloud.google.com/gke-nodepool=default-pool -o=name); do
    kubectl cordon "$node";
    done`
    `for node in $(kubectl get nodes -l cloud.google.com/gke-nodepool=default-pool -o=name); do
    kubectl drain --force --ignore-daemonsets --delete-local-data --grace-period=10 "$node";
    done`
5. Delete the default-pool once the deployments have safely migrated.
    `gcloud container node-pools delete default-pool --cluster onlineboutique-cluster-492 --zone us-west1-c`

## Task 3
Please create pod disruption budget named onlineboutique-frontend-pdb for frontend deployment with min-availability to 1
`kubectl create poddisruptionbudget onlineboutique-frontend-pdb --selector run=frontend --min-available 1`

## Task 4
1. `kubectl autoscale deployment frontend --cpu-percent=50 \
   --min=1 --max=13 --namespace dev`
2. `gcloud beta container clusters update onlineboutique-cluster-492 \
--enable-autoscaling --min-nodes 1 --max-nodes 6 --zone us-west1-c`
3. `kubectl exec $(kubectl get pod --namespace=dev | grep 'loadgenerator' | cut -f1 -d ' ') -it --namespace=dev -- bash -c 'export USERS=8000; locust --host="http://35.230.6.26" --headless -u "8000" 2>&1'`