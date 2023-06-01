The instructions in the lab might not be clear. For example, startup script they give you is incorrect resulting in instance groups always in transforming state because of unhealthy VMs

## Load balancer
Before running script update region and zone from the lab value

```
gcloud config set compute/region us-central1
```
```
gcloud config set compute/zone us-central1-b
```
```
gcloud compute health-checks create http http-basic-check \
    --port 80
```
```
gcloud compute firewall-rules create grant-tcp-rule-761 \
    --network=default \
    --action=allow \
    --direction=ingress \
    --source-ranges=130.211.0.0/22,35.191.0.0/16 \
    --target-tags=allow-health-check \
    --rules=tcp:80
```
```
gcloud compute instance-templates create lb-group-template \
   --region=us-central1 \
   --network=default \
   --subnet=default \
   --tags=allow-health-check \
   --image-family=debian-10 \
   --image-project=debian-cloud \
   --metadata=startup-script='#! /bin/bash
   apt-get update
   apt-get install -y nginx
   service nginx start
   sed -i -- "s/nginx/Google Cloud Platform - $HOSTNAME/" /var/www/html/index.nginx-debian.html'
```
```
gcloud compute instance-groups managed create lb-backend-example \
  --template=lb-group-template --size=2 --zone=us-central1-b \
  --health-check=http-basic-check --initial-delay=60
```
```
gcloud compute instance-groups set-named-ports lb-backend-example \
    --named-ports http:80 \
    --zone us-central1-b
```
```
gcloud compute addresses create lb-ipv4-1 \
    --ip-version=IPV4 \
    --network-tier=PREMIUM \
    --global
```
```
gcloud compute backend-services create web-backend-service \
  --load-balancing-scheme=EXTERNAL \
  --protocol=HTTP \
  --port-name=http \
  --health-checks=http-basic-check \
  --global
```
```
gcloud compute backend-services add-backend web-backend-service \
    --instance-group=lb-backend-example \
    --instance-group-zone=us-central1-b \
    --global
```
```
gcloud compute url-maps create web-map-http \
    --default-service web-backend-service
```
```
gcloud compute target-http-proxies create http-lb-proxy \
    --url-map=web-map-http
```
```
gcloud compute forwarding-rules create http-content-rule \
    --load-balancing-scheme=EXTERNAL \
    --address=lb-ipv4-1 \
    --global \
    --target-http-proxy=http-lb-proxy \
    --ports=80
```

  
      
## GKE
Update port before running
```
gcloud container clusters create hello-cluster --zone us-central1-b
```
```
gcloud container clusters get-credentials hello-cluster --zone us-central1-b
```
```
kubectl create deployment hello-server --image=gcr.io/google-samples/hello-app:2.0
```
```
kubectl expose deployment hello-server --type LoadBalancer --port 80 --target-port 8080
```
