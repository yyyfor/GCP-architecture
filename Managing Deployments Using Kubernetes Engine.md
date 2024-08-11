# Link
https://www.cloudskillsboost.google/course_templates/625/labs/464389

### set zone
`gcloud config set compute/zone us-east4-b`

### Get sample code for this lab
Get the sample code for creating and running containers and deployments:
`gsutil -m cp -r gs://spls/gsp053/orchestrate-with-kubernetes .
cd orchestrate-with-kubernetes/kubernetes`
Create a cluster with 3 nodes (this will take a few minutes to complete):
`gcloud container clusters create bootcamp \
--machine-type e2-small \
--num-nodes 3 \
--scopes "https://www.googleapis.com/auth/projecthosting,storage-rw"`

## Task 2: create deployment
`kubectl create -f deployments/auth.yaml`

Use the kubectl create command to create the auth service:
`kubectl create -f services/auth.yaml`
Now, do the same thing to create and expose the hello deployment:
`kubectl create -f deployments/hello.yaml
kubectl create -f services/hello.yaml`
And one more time to create and expose the frontend deployment:
`kubectl create secret generic tls-certs --from-file tls/
kubectl create configmap nginx-frontend-conf --from-file=nginx/frontend.conf
kubectl create -f deployments/frontend.yaml
kubectl create -f services/frontend.yaml`

Interact with the frontend by grabbing its external IP and then curling to it:
`kubectl get services frontend`
Note: It may take a few seconds before the External-IP field is populated for your service. This is normal. Just re-run the above command every few seconds until the field is populated.
`curl -ks https://<EXTERNAL-IP>`

### Scale a deployment
`kubectl scale deployment hello --replicas=5`

## Task 3. Rolling update
To update your deployment, run the following command:
`kubectl edit deployment hello`

You can also see a new entry in the rollout history:
`kubectl rollout history deployment/hello`

Run the following to pause the rollout:
`kubectl rollout pause deployment/hello`

Continue the rollout using the resume command:
`kubectl rollout resume deployment/hello`

Use the rollout command to roll back to the previous version:
`kubectl rollout undo deployment/hello`
Verify the roll back in the history:
`kubectl rollout history deployment/hello`

### create canary deployment
`kubectl create -f deployments/hello-canary.yaml`

## Task 5. Blue-green deployments
