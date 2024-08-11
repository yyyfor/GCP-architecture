# Link
https://www.cloudskillsboost.google/course_templates/625/labs/464387

### Task 1. Create custom mode VPC networks with firewall rules

#### create management
```
gcloud compute networks create managementnet --project=qwiklabs-gcp-02-145b778a7eab --subnet-mode=custom --mtu=1460 --bgp-routing-mode=regional && gcloud compute networks subnets create managementsubnet-us --project=qwiklabs-gcp-02-145b778a7eab --range=10.130.0.0/20 --stack-type=IPV4_ONLY --network=managementnet --region=us-central1
```

#### create the privatenet network
Create the privatenet network using the Cloud Shell command line.

Run the following command to create the privatenet network:
`gcloud compute networks create privatenet --subnet-mode=custom`
Run the following command to create the privatesubnet-us subnet:
`gcloud compute networks subnets create privatesubnet-us --network=privatenet --region=us-central1 --range=172.16.0.0/24`
Run the following command to create the privatesubnet-eu subnet:
`gcloud compute networks subnets create privatesubnet-eu --network=privatenet --region=europe-west4 --range=172.20.0.0/20`
Run the following command to list the available VPC networks:
`gcloud compute networks list`
Run the following command to list the available VPC subnets (sorted by VPC network):
`gcloud compute networks subnets list --sort-by=NETWORK`

#### create firewall rules for management
`gcloud compute --project=qwiklabs-gcp-02-145b778a7eab firewall-rules create managementnet-allow-icmp-ssh-rdp --direction=INGRESS --priority=1000 --network=managementnet --action=ALLOW --rules=tcp:22,tcp:3389,icmp --source-ranges=0.0.0.0/0`

#### create firewall rules for privatenet
`gcloud compute firewall-rules create privatenet-allow-icmp-ssh-rdp --direction=INGRESS --priority=1000 --network=privatenet --action=ALLOW --rules=icmp,tcp:22,tcp:3389 --source-ranges=0.0.0.0/0`

### Task 2: create vm
```
gcloud compute instances create managementnet-us-vm --project=qwiklabs-gcp-02-145b778a7eab --zone=us-central1-c --machine-type=e2-micro --network-interface=network-tier=PREMIUM,stack-type=IPV4_ONLY,subnet=managementsubnet-us --metadata=enable-oslogin=true --maintenance-policy=MIGRATE --provisioning-model=STANDARD --service-account=795710117399-compute@developer.gserviceaccount.com --scopes=https://www.googleapis.com/auth/devstorage.read_only,https://www.googleapis.com/auth/logging.write,https://www.googleapis.com/auth/monitoring.write,https://www.googleapis.com/auth/service.management.readonly,https://www.googleapis.com/auth/servicecontrol,https://www.googleapis.com/auth/trace.append --create-disk=auto-delete=yes,boot=yes,device-name=managementnet-us-vm,image=projects/debian-cloud/global/images/debian-12-bookworm-v20240709,mode=rw,size=10,type=pd-balanced --no-shielded-secure-boot --shielded-vtpm --shielded-integrity-monitoring --labels=goog-ec-src=vm_add-gcloud --reservation-affinity=any
```

#### create the privatenet vm
In Cloud Shell, run the following command to create the privatenet-us-vm instance:
`gcloud compute instances create privatenet-us-vm --zone=us-central1-c --machine-type=e2-micro --subnet=privatesubnet-us`
