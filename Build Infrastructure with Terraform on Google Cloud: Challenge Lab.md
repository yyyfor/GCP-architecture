# Link
https://www.cloudskillsboost.google/course_templates/636/labs/464836

## task1 create configuration files

#### create directories
```
main.tf
variables.tf
modules/
└── instances
    ├── instances.tf
    ├── outputs.tf
    └── variables.tf
└── storage
    ├── storage.tf
    ├── outputs.tf
    └── variables.tf
```

#### fill out variable
```
variable "region" {
  type = string
  default = "us-east1"
}

variable "zone" {
  type = string
  default = "us-east1-d"
}

variable "project_id" {
  type = string
  default = "qwiklabs-gcp-01-0b73d2ba76fc"
}

variable "bucket_name" {
  type = string
  default = "tf-bucket-787896"
}

variable "instance_name" {
  type = string
  default = "tf-instance-368908"
}

variable "vpc_name" {
  type = string
  default = "tf-vpc-789333"
}
```
#### add main.tf
```
provider "google" {
  project     = var.project_id
  region      = var.region
  zone        = var.zone
}


module "instances" {
  source     = "./modules/instances"
}
```

### import infrastructure

#### add module to instance.tf
```
resource "google_compute_instance" "vm_instance_1" {
  name         = "tf-instance-1"
  machine_type = "e2-micro"
  boot_disk {
    initialize_params {
      image = "debian-11-bullseye-v20240611"
    }
  }
  network_interface {
    network = "default"
  } 
  metadata_startup_script = <<-EOT
    #!/bin/bash
  EOT

  allow_stopping_for_update = true
}

resource "google_compute_instance" "vm_instance_2" {
  name         = "tf-instance-2"
  machine_type = "e2-micro"
  boot_disk {
    initialize_params {
      image = "debian-11-bullseye-v20240611"
    }
  }
  network_interface {
    network = "default"
  }
  metadata_startup_script = <<-EOT
    #!/bin/bash
  EOT
  
  allow_stopping_for_update = true
}
```

#### import instance
`
terraform import module.instances.google_compute_instance.vm_instance_1 1270429772816954647
terraform import module.instances.google_compute_instance.vm_instance_2 53944096554071319
`

### Configure a remote backend

#### configure cloud bucket
```
resource "google_storage_bucket" "test-bucket-for-state" {
  name        = "tf-bucket-690740"
  location    = "US"
  force_destroy = true
  uniform_bucket_level_access = true
}
```

#### configure backend to main.tf
```
terraform {
  backend "gcs" {
    bucket  = "tf-bucket-787896"
    prefix  = "terraform/state"
  }
}
```

### modify and update infra

#### update instances
```
resource "google_compute_instance" "vm_instance_1" {
  name         = "tf-instance-1"
  machine_type = "e2-standard-2"
  boot_disk {
    initialize_params {
      image = "debian-11-bullseye-v20240611"
    }
  }
  network_interface {
    network = "default"
  } 
  metadata_startup_script = <<-EOT
    #!/bin/bash
  EOT

  allow_stopping_for_update = true
}

resource "google_compute_instance" "vm_instance_2" {
  name         = "tf-instance-2"
  machine_type = "e2-standard-2"
  boot_disk {
    initialize_params {
      image = "debian-11-bullseye-v20240611"
    }
  }
  network_interface {
    network = "default"
  }
  metadata_startup_script = <<-EOT
    #!/bin/bash
  EOT
  
  allow_stopping_for_update = true
}

resource "google_compute_instance" "vm_instance_3" {
  name         = "tf-instance-368908"
  machine_type = "e2-standard-2"
  boot_disk {
    initialize_params {
      image = "debian-11-bullseye-v20240611"
    }
  }
  network_interface {
    network = "default"
  }
  metadata_startup_script = <<-EOT
    #!/bin/bash
  EOT
  
  allow_stopping_for_update = true
}
```

### use a module from registry

#### create network in main.tf
```
module "network" {
  source  = "terraform-google-modules/network/google"
  version = "6.0.0"
  # insert the 3 required variables here
  project_id   = "qwiklabs-gcp-01-0b73d2ba76fc"
  network_name = "tf-vpc-789333"

  subnets = [
      {
          subnet_name           = "subnet-01"
          subnet_ip             = "10.10.10.0/24"
          subnet_region         = "us-west1"
      },
      {
          subnet_name           = "subnet-02"
          subnet_ip             = "10.10.20.0/24"
          subnet_region         = "us-west1"
      }
  ]
}
```

### modify instance
```
network_interface {
    network = "tf-vpc-789333"
    subnetwork = "subnet-01"
  } 
```

### configure a firewall
```
resource "google_compute_firewall" "firewall" {
  name    = "tf-firewall"
  network = "projects/qwiklabs-gcp-01-0b73d2ba76fc/global/networks/"

  source_ranges = ["0.0.0.0/0"]
  allow {
    protocol = "tcp"
    ports    = ["80"]
  }
}
```





