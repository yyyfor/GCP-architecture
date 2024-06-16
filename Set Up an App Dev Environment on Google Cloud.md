# Link
https://www.cloudskillsboost.google/course_templates/637

## Cloud Storage: Qwik Start - CLI/SDK
https://www.cloudskillsboost.google/course_templates/637/labs/464351

### Command 

#### Set the region and zone
```
gcloud config set compute/zone "ZONE"
export ZONE=$(gcloud config get compute/zone)

gcloud config set compute/region "REGION"
export REGION=$(gcloud config get compute/region)
```

#### create cloud storage bucket
`
gsutil mb gs://<siming-test>
`

#### upload to storage
`
gsutil cp ada.jpg gs://siming-test
`

#### download
`
gsutil cp -r gs://siming-test/ada.jpg .
`

#### copy to a folder
`
gsutil cp gs://siming-test/ada.jpg gs://siming-test/image-folder/
`

#### list
`
gsutil ls gs://siming-test
`

#### grant permission
`
gsutil acl ch -u AllUsers:R gs://siming-test/ada.jpg
`

#### revoke permission
`
gsutil acl ch -d AllUsers gs://siming-test/ada.jpg
`

#### delete objects
`
gsutil rm gs://siming-test/ada.jpg
`

#### vim to replace string
`
:%s/old/new/g
`

## Cloud IAM: Qwik Start
https://www.cloudskillsboost.google/course_templates/637/labs/464352

## Cloud Functions: Qwik Start - Command Line
https://www.cloudskillsboost.google/course_templates/637/labs/464355

#### create cloud storage bucket
`
gsutil mb -p qwiklabs-gcp-01-7ee460651a71 gs://qwiklabs-gcp-01-7ee460651a71
`

#### deploy cloud function
`
gcloud projects add-iam-policy-binding qwiklabs-gcp-01-7ee460651a71 \
--member="serviceAccount:qwiklabs-gcp-01-7ee460651a71@appspot.gserviceaccount.com" \
--role="roles/artifactregistry.reader"
`

#### deploy function to pub sub
`
gcloud functions deploy helloWorld \
--stage-bucket qwiklabs-gcp-01-7ee460651a71 \
--trigger-topic hello_world \
--runtime nodejs20
`
#### test function
`
DATA=$(printf 'Hello World!'|base64) && gcloud functions call helloWorld --data '{"data":"'$DATA'"}'
`

#### view logs
`
gcloud functions logs read helloWorld
`

## Pub/Sub: Qwik Start - Python
https://www.cloudskillsboost.google/course_templates/637/labs/464358

#### install pubsub python
`
pip install --upgrade google-cloud-pubsub
`

#### git sample code
`
git clone https://github.com/googleapis/python-pubsub.git
`

#### create topic
`
python publisher.py $GOOGLE_CLOUD_PROJECT create MyTopic
`

#### create subscription
`
python subscriber.py $GOOGLE_CLOUD_PROJECT create MyTopic MySub
`

#### publish a message
`
gcloud pubsub topics publish MyTopic --message "Hello"
`

#### pull messages
`
python subscriber.py $GOOGLE_CLOUD_PROJECT receive MySub
`

## Set Up an App Dev Environment on Google Cloud: Challenge Lab
https://www.cloudskillsboost.google/course_templates/637/labs/464359

#### create a bucket
```
gcloud config set compute/zone us-west1-c
export ZONE=$(gcloud config get compute/zone)

gcloud config set compute/region us-west1
export REGION=$(gcloud config get compute/region)

gsutil mb gs://qwiklabs-gcp-03-b0d6b5de67f1-bucket
```

#### create a pub/sub topic
``` shell
gcloud pubsub topics create topic-memories-316 
```

#### upload image to bucket
`
gsutil cp map.jpg gs://qwiklabs-gcp-03-b0d6b5de67f1-bucket
`
