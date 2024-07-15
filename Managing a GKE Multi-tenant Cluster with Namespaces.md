# Link
https://www.cloudskillsboost.google/course_templates/655/labs/464669

## Task 1. Download required files
```
gsutil -m cp -r gs://spls/gsp766/gke-qwiklab ~
cd ~/gke-qwiklab
```

## Task 2. View and create namespaces
```
export ZONE=us-east1-b
gcloud config set compute/zone ${ZONE} && gcloud container clusters get-credentials multi-tenant-cluster
```

### Default namespaces
- full list of the current cluster's namespaces with
`kubectl get namespace`
- complete list of namespaced resources
`kubectl api-resources --namespaced=true`
- The namespace can also be specified with any kubectl get subcommand to display a namespace's resources
`kubectl get services --namespace=kube-system`

### Creating new namespaces
- Create 2 namespaces for team-a and team-b
`kubectl create namespace team-a && \
  kubectl create namespace team-b`
- run the following to deploy a pod in the team-a namespace and in the team-b namespace using the same name:
`kubectl run app-server --image=centos --namespace=team-a -- sleep infinity && \
  kubectl run app-server --image=centos --namespace=team-b -- sleep infinity`
- kubectl get pods -A to see there are 2 pods named app-server, one for each team namespace
`kubectl get pods -A`
- Use kubectl describe to see additional details for each of the newly created pods by specifying the namespace with the --namespace tag
`kubectl describe pod app-server --namespace=team-a`
- To work exclusively with resources in one namespace, you can set it once in the kubectl context instead of using the --namespace flag for every command
`kubectl config set-context --current --namespace=team-a`
- After this, any subsequent commands will be run against the indicated namespace without specifying the --namespace flag
`kubectl describe pod app-server`

## Task 3. Access Control in namespaces
- Grant the account the Kubernetes Engine Cluster Viewer role by running the following
`gcloud projects add-iam-policy-binding ${GOOGLE_CLOUD_PROJECT} \
  --member=serviceAccount:team-a-dev@${GOOGLE_CLOUD_PROJECT}.iam.gserviceaccount.com  \
  --role=roles/container.clusterViewer`

### Kubernetes RBAC
1. Roles with single rules can be created with kubectl create
`kubectl create role pod-reader \
   --resource=pods --verb=watch --verb=get --verb=list`
2. Inspect the yaml file:
`cat developer-role.yaml`
3. kubectl create -f developer-role.yaml
`kubectl create -f developer-role.yaml`
4. Create a role binding between the team-a-developers serviceaccount and the developer-role:
`kubectl create rolebinding team-a-developers \
   --role=developer --user=team-a-dev@${GOOGLE_CLOUD_PROJECT}.iam.gserviceaccount.com`

### Test the rolebinding
1. Download the service account keys used to impersonate the service account
`gcloud iam service-accounts keys create /tmp/key.json --iam-account team-a-dev@${GOOGLE_CLOUD_PROJECT}.iam.gserviceaccount.com`
2. In Cloud Shell click the + to open a new tab in your terminal.
3. Here, run the following to activate the service account. This will allow you to run the commands as the account:
`gcloud auth activate-service-account  --key-file=/tmp/key.json`
4. Get the credentials for your cluster, as the service account:
`export ZONE=us-east1-b
   gcloud container clusters get-credentials multi-tenant-cluster --zone ${ZONE} --project ${GOOGLE_CLOUD_PROJECT}`
5. you're now able to run kubectl commands as the service account:
`kubectl get pods --namespace=team-a`

## Task 4. Resource quotas
1. the following will set a limit to the number of pods allowed in the namespace team-a to 2, and the number of loadbalancers to 1:
`kubectl create quota test-quota \
   --hard=count/pods=2,count/services.loadbalancers=1 --namespace=team-a`
2. Create a second pod in the namespace team-a:
`kubectl run app-server-2 --image=centos --namespace=team-a -- sleep infinity`
3. Now try to create a third pod:
`kubectl run app-server-3 --image=centos --namespace=team-a -- sleep infinity`
4. You can check details about your resource quota using kubectl describe:
   `kubectl describe quota test-quota --namespace=team-a`
5. Update test-quota to have a limit of 6 pods by running:
   ```
   export KUBE_EDITOR="nano"
   kubectl edit quota test-quota --namespace=team-a
   ```
6. Change the value of count/pods under spec to 6:
 `kubectl describe quota test-quota --namespace=team-a`

#### CPU and memory quotas
1. Apply the file configuration
`kubectl create -f cpu-mem-quota.yaml`
2. Apply the file configuration:
   `kubectl create -f cpu-mem-demo-pod.yaml --namespace=team-a`
3. Once this pod has been created, run the following to see its CPU and memory requests and limits reflected in the quota:
   `kubectl describe quota cpu-mem-quota --namespace=team-a`

## Task 5. Monitoring GKE and GKE usage metering
Run the following to enable GKE usage metering on the cluster and specify the dataset cluster_dataset:
```
export ZONE=us-east1-b
gcloud container clusters \
update multi-tenant-cluster --zone ${ZONE} \
--resource-usage-bigquery-dataset cluster_dataset
```
