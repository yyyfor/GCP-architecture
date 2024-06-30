# Link
https://www.cloudskillsboost.google/games/5210/labs/34069

## Task 2 Enable the Cloud Run API
Open the Navigation menu (Navigation menu icon) and click APIs & Services > Library. In the search bar, enter "Cloud Run" and select the Cloud Run Admin API from the results list.

## Task 3. Deploy a simple Cloud Run service

#### Command to export cloud run url to env
`
SERVICE_URL=$(gcloud beta run services describe pdf-converter --platform managed --region us-east4 --format="value(status.url)")
`

#### Command to invoke service as authorized user
`
curl -X POST -H "Authorization: Bearer $(gcloud auth print-identity-token)" $SERVICE_URL
`

#### Cloud Storage to send a Pub/Sub notification whenever a new file has finished uploading to the docs bucket
`
gsutil notification create -t new-doc -f json -e OBJECT_FINALIZE gs://$GOOGLE_CLOUD_PROJECT-upload
`

#### create new service account pub/sub to trigger cloud run
`
gcloud iam service-accounts create pubsub-cloud-run-invoker --display-name "PubSub Cloud Run Invoker"
`

#### grant permission to service account
`
gcloud beta run services add-iam-policy-binding pdf-converter --member=serviceAccount:pubsub-cloud-run-invoker@$GOOGLE_CLOUD_PROJECT.iam.gserviceaccount.com --role=roles/run.invoker --platform managed --region us-east4
`

#### 
```
gcloud projects list
Look for the project whose name starts with "qwiklabs-gcp-".
PROJECT_NUMBER=[project number]
```

#### enable project to create pub/sub authentication token 
`
gcloud projects add-iam-policy-binding $GOOGLE_CLOUD_PROJECT --member=serviceAccount:service-$PROJECT_NUMBER@gcp-sa-pubsub.iam.gserviceaccount.com --role=roles/iam.serviceAccountTokenCreator
`

#### create pub/sub subscription
`
gcloud beta pubsub subscriptions create pdf-conv-sub --topic new-doc --push-endpoint=$SERVICE_URL --push-auth-service-account=pubsub-cloud-run-invoker@$GOOGLE_CLOUD_PROJECT.iam.gserviceaccount.com
`

