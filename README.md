# tfe-active-active-aws
This is a testing repository to install TFE Active/Active 


The code in this repository is based on this [Github repo](https://github.com/munnep/tfe_aws_active_mode_step) with few changes to have my own names and my own DNS. 


This repository is installing Terraform Enterprise Active/Active on AWS.

We will be using RDS PostgreSQL and a S3 Bucket on AWS. 

An autoscaling group is also used to scale the nodes


### Prerequisites

- [X] [Terraform](https://www.terraform.io/downloads)
- [X] [Replicated license](https://hashicorp.atlassian.net/wiki/spaces/tfsupport/pages/676792039/Terraform+Enterprise+Installation#Replicated-license)
- [X]  [Airgap bundle](https://www.hashicorp.com/blog/deploying-terraform-enterprise-in-airgapped-environments)
- [X]  [Download the installation bundle](https://install.terraform.io/airgap/latest.tar.gz)
- [X]  [AWS account](https://aws.amazon.com/free/?trk=65c60aef-03ac-4364-958d-38c6ccb6a7f7&sc_channel=ps&ef_id=Cj0KCQjwpPKiBhDvARIsACn-gzANBKWZIF6qWBv2Mo56iPFHfY1whaEvqVF18yhFhdqJfIQbJ7uRAXsaAvkXEALw_wcB:G:s&s_kwcid=AL!4422!3!458573551417!p!!g!!aws%20account!10908848282!107577274975&all-free-tier.sort-by=item.additionalFields.SortRank&all-free-tier.sort-order=asc&awsf.Free%20Tier%20Types=*all&awsf.Free%20Tier%20Categories=*all)
- [X]  [Install AWS CLI](https://aws.amazon.com/cli/)

## How to Use this Repo

- Clone this repository:
```shell
git clone git@github.com:dlavric/tfe-active-active-aws.git
```

- Go to the directory where the repo is stored:
```shell
cd tfe-active-active-aws
```

- Setup your AWS credentials as environment variables:
```shell
export AWS_ACCESS_KEY_ID=<your-aws-key-id>
export AWS_SECRET_ACCESS_KEY=<your-secret-access-key>
export AWS_SESSION_TOKEN=<your-session-token>
```

- Create a folder `files`:
```shell
mkdir files
```

- Go to the `files` directory:
```shell
cd files
```

- Store the license file here: `files/license.rli`

- Store the `.airgap` bundle to  `files/692.airgap`

- Store the installation bundle under `files/replicated.tar.gz`.The installation bundle contains the `install.sh` script along with the Replicated container images.

- Create a terraform file with your own variables `variables.auto.tfvars`:
```terraform
tag_prefix               = "daniela-tfe"                              # TAG prefix for names to easily find your AWS resources
region                   = "eu-west-2"                                # Region to create the environment
vpc_cidr                 = "10.234.0.0/16"                            # subnet mask that can be used 
ami                      = "ami-05147510eb2885c80"                    # AMI of the Ubuntu image  
rds_password             = "Password#1"                               # password used for the RDS environment
filename_airgap          = "692.airgap"                               # filename of your airgap software stored under ./airgap
filename_license         = "license.rli"                              # filename of your TFE license stored under ./airgap
filename_bootstrap       = "replicated.tar.gz"                        # filename of the bootstrap installer stored under ./airgap
dns_hostname             = "daniela-tfe"                              # DNS hostname for the TFE
dns_zonename             = "tf-support.hashicorpdemo.com"             # DNS zone name to be used
tfe_password             = "Password#1"                               # TFE password for the dashboard and encryption of the data
certificate_email        = "<your-email>@hashicorp.com"              # Your email address used by TLS certificate registration
terraform_client_version = "1.1.7"                                    # Terraform version you want to have installed on the client machine
public_key               = "ssh-rsa AAAAB3Nza"                        # The public key for you to connect to the server over SSH
asg_min_size             = 1                                          # autoscaling group minimal size. Currently 1 is the only option
asg_max_size             = 1                                          # autoscaling group maximum size. Currently 1 is the only option
asg_desired_capacity     = 1                                          # autoscaling group desired capacity. Currently 1 is the only option
tfe_active_active        = false                                      # TFE instance setup of active/active in the launch of the instance.Default false to start with
```

- Initialize Terraform:
```shell
terraform init
```

- Apply the changes:
```shell
terraform apply
```

- If everything is okay, Terraform should create 48 resources and show you the public DNS you can use to connect to the TFE instance
```shell
Apply complete! Resources: 48 added, 0 changed, 0 destroyed.
```




