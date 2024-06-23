## Link
https://www.cloudskillsboost.google/course_templates/636/labs/464835

### Command

#### retrieve project id
`
gcloud config list --format 'value(core.project)'
`

#### main.tf
```
provider "google" {
  project     = "# REPLACE WITH YOUR PROJECT ID"
  region      = "us-east1"
}

resource "google_storage_bucket" "test-bucket-for-state" {
  name        = "# REPLACE WITH YOUR PROJECT ID"
  location    = "US"
  uniform_bucket_level_access = true
}
```

#### add local backend to main.tf
```
terraform {
  backend "local" {
    path = "terraform/state/terraform.tfstate"
  }
}
```

#### add a cloud storage backend
```shell
terraform {
  backend "gcs" {
    bucket  = "qwiklabs-gcp-01-319e8ca8f084"
    prefix  = "terraform/state"
  }
}
```

#### migrate state
```shell
terraform init -migrate-state
```

#### create docker container
```
docker run --name hashicorp-learn --detach --publish 8080:80 nginx:latest
docker ps
terraform import docker_container.web $(docker inspect -f {{.ID}} hashicorp-learn)
```

### create configuration

#### copy terraform state into docker.tf
`
terraform show -no-color > docker.tf
`

#### docker.tf
```
resource "docker_container" "web" {
    image = "sha256:87a94228f133e2da99cb16d653cd1373c5b4e8689956386c1c12b60a20421a02"
    name  = "hashicorp-learn"
    ports {
        external = 8080
        internal = 80
        ip       = "0.0.0.0"
        protocol = "tcp"
    }
}
```

