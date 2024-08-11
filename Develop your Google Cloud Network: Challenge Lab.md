# Link
https://www.cloudskillsboost.google/course_templates/625/labs/464390

## Objectives
- Create all resources in the us-east4 region and us-east4-c zone, unless otherwise directed.
- Use the project VPCs.
- Naming is normally team-resource, e.g. an instance could be named kraken-webserver1.
- Allocate cost effective resource sizes. Projects are monitored and excessive resource use will result in the containing project's termination (and possibly yours), so beware. This is the guidance the monitoring team is willing to share: unless directed, use e2-medium.

`
export REGION=us-west1
export ZONE=us-west1-a
`

## Task 1: create development VPC manually
Create a VPC called griffin-dev-vpc with the following subnets only
`gcloud compute networks create griffin-dev-vpc --subnet-mode=custom`

`gcloud compute networks subnets create griffin-dev-wp --network=griffin-dev-vpc --region=$REGION --range=192.168.16.0/20`
`gcloud compute networks subnets create griffin-dev-mgmt --network=griffin-dev-vpc --region=$REGION --range=192.168.32.0/20`

## Task 2: create production
Create a VPC called griffin-prod-vpc with the following subnets only:
`gcloud compute networks create griffin-prod-vpc --subnet-mode=custom`

`gcloud compute networks subnets create griffin-prod-wp --network=griffin-prod-vpc --region=$REGION --range=192.168.48.0/20`
`gcloud compute networks subnets create griffin-prod-mgmt --network=griffin-prod-vpc --region=$REGION --range=192.168.64.0/20`

## Task 3: create bastion host
Create a bastion host with two network interfaces, one connected to griffin-dev-mgmt and the other connected to griffin-prod-mgmt. Make sure you can SSH to the host.
`gcloud compute instances create griffin-bastion --zone=$ZONE --machine-type=e2-medium --network-interface=network-tier=PREMIUM,subnet=griffin-dev-mgmt --network-interface=network-tier=PREMIUM,subnet=griffin-prod-mgmt`

```
gcloud compute firewall-rules create griffin-dev-allow-ssh \
    --network=griffin-dev-vpc \
    --allow=tcp:22 \
    --source-ranges=0.0.0.0/0 \
    --target-tags=bastion \
    --description="Allow SSH access to bastion host"
    
gcloud compute firewall-rules create griffin-prod-allow-ssh \
--network=griffin-prod-vpc \
--allow=tcp:22 \
--source-ranges=0.0.0.0/0 \
--target-tags=bastion \
--description="Allow SSH access to bastion host in production"
```

## Task 4. Create and configure Cloud SQL Instance
```
gcloud sql connect griffin-dev-db --user=root
CREATE DATABASE wordpress;
CREATE USER 'wp_user'@'%' IDENTIFIED BY 'stormwind_rules';
GRANT ALL PRIVILEGES ON wordpress.* TO 'wp_user'@'%';
FLUSH PRIVILEGES;
EOF
```


## Task 5. Create Kubernetes cluster
`gcloud container clusters create griffin-dev \
--machine-type e2-standard-4 \
--num-nodes 2 \
--network=griffin-dev-vpc \
--subnetwork=griffin-dev-wp \
--zone=$ZONE
`

## Task 6. Prepare the Kubernetes cluster
`gsutil -m cp -r gs://cloud-training/gsp321/wp-k8s .`

## Task 7. create a wordpress deployment
`kubectl create -f wp-service.yaml`



