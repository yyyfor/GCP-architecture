## Skill boost 
https://www.cloudskillsboost.google/course_templates/648

## Links
- https://cloud.google.com/load-balancing/docs/network/networklb-backend-service#xpn-architecture
- https://cloud.google.com/load-balancing/docs/network/setting-up-network-backend-service

## Script
```
gcloud config set project qwiklabs-gcp-00-dd405677c9b5
gcloud config set compute/region us-central1
gcloud config set compute/zone us-central1-a
gcloud compute instances create nucleus-jumphost-404 --machine-type=e2-micro

cat << EOF > startup.sh
#! /bin/bash
apt-get update
apt-get install -y nginx
service nginx start
sed -i -- 's/nginx/Google Cloud Platform - '"$HOSTNAME"'/' /var/www/html/index.nginx-debian.html
EOF

gcloud compute instance-templates create lb-backend-template --region=us-central1 --network=default --subnet=default --tags=allow-health-check --machine-type=e2-medium --image-family=debian-11 --image-project=debian-cloud --metadata-from-file startup-script=startup.sh

gcloud compute target-pools create nginx-pool

gcloud compute instance-groups managed create lb-backend-group --base-instance-name nginx --template=lb-backend-template --size=2 --target-pool nginx-pool --zone=us-central1-a

gcloud compute firewall-rules create www-firewall --allow tcp:80

gcloud compute firewall-rules create accept-tcp-rule-710 --network=default --action=allow --direction=ingress --source-ranges=130.211.0.0/22,35.191.0.0/16 --target-tags=allow-health-check --rules=tcp:80

gcloud compute addresses create lb-ipv4-1 --ip-version=IPV4 --global

gcloud compute health-checks create http http-basic-check --port 80

gcloud compute forwarding-rules create nginx-lb --region us-central1 --ports=80 --target-pool nginx-pool

gcloud compute http-health-checks create http-basic-check

gcloud compute instance-groups managed set-named-ports lb-backend-group --named-ports http:80

gcloud compute backend-services create web-backend-service --protocol=HTTP --port-name=http --health-checks=http-basic-check --global

gcloud compute backend-services add-backend web-backend-service --instance-group=lb-backend-group --instance-group-zone=us-central1-a --global

gcloud compute url-maps create web-map-http --default-service web-backend-service

gcloud compute target-http-proxies create http-lb-proxy --url-map web-map-http

gcloud compute forwarding-rules create http-content-rule --address=lb-ipv4-1 --global --target-http-proxy=http-lb-proxy --ports=80
```