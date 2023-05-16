# tfe-active-active-aws
This is a testing repository to install TFE Active/Active 


The code in this repository is based on this [Github repo](https://github.com/munnep/tfe_aws_active_mode_step) with few changes to have my own names and my own DNS. 


This repository is installing Terraform Enterprise Active/Active on AWS.

We will be using RDS PostgreSQL and a S3 Bucket on AWS. 

An autoscaling group is used to scale the nodes.


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
region                   = "eu-west-1"                                # Region to create the environment
vpc_cidr                 = "10.234.0.0/16"                            # subnet mask that can be used 
ami                      = "ami-05147510eb2885c80"                    # AMI of the Ubuntu image  
rds_password             = "Password#1"                               # password used for the RDS environment
filename_airgap          = "692.airgap"                               # filename of your airgap software stored under ./airgap
filename_license         = "license.rli"                              # filename of your TFE license stored under ./airgap
filename_bootstrap       = "replicated.tar.gz"                        # filename of the bootstrap installer stored under ./airgap
dns_hostname             = "daniela-tfe"                              # DNS hostname for the TFE
dns_zonename             = "tf-support.hashicorpdemo.com"                 # DNS zone name to be used
tfe_password             = "Password#1"                               # TFE password for the dashboard and encryption of the data
certificate_email        = "<your-email>@hashicorp.com"               # Your email address used by TLS certificate registration
terraform_client_version = "1.1.7"                                    # Terraform version you want to have installed on the client machine
public_key               = "ssh-rsa AAAAB3Nza"                        # The public key for you to connect to the server over SSH. Get it with `$ cat ~/.ssh/id_rsa.pub`
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

Apply complete! Resources: 48 added, 0 changed, 0 destroyed.

Outputs:

ssh_tf_client = "ssh ubuntu@daniela-tfe-client.tf-support.hashicorpdemo.com"
ssh_tfe_server = "ssh -J ubuntu@daniela-tfe-client.tf-support.hashicorpdemo.com ubuntu@<internal ip address of the TFE server>"
tfe_appplication = "https://daniela-tfe.tf-support.hashicorpdemo.com"
tfe_dashboard = "https://daniela-tfe.tf-support.hashicorpdemo.com:8800"
tfe_netdata_performance_dashboard = "http://daniela-tfe.tf-support.hashicorpdemo.com:19999"
```

**Create an admin user, a workspace and an organization on the TFE application**

We are going to run the script created through the `tf-script/user=data-single.sh`file `/tmp/tfe_setup.sh`.

The script is going to create
- an admin user named: admin (with default password)
- an organization named: test
- a workspace named: test-workspace

```shell
ssh -J ubuntu@daniela-tfe-client.tf-support.hashicorpdemo.com ubuntu@<your-private-ip-address-of-the-asg-instance>

ssh -J ubuntu@daniela-tfe-client.tf-support.hashicorpdemo.com ubuntu@10.234.11.26 bash /tmp/tfe_setup.sh
```

- If you cannot login to the server through ssh and you get the following message remove 
the content of the folder `~/.ssh/known_hosts` with the `$ rm ~/.ssh/known_hosts` :
```shell
The authenticity of host '10.234.1.23 (<no hostip for proxy command>)' can't be established.
ED25519 key fingerprint is SHA256:j/qen2osWzv94fsNgV5+2zSJevq+BCTEsGEDTLuVqos.
This host key is known by the following other names/addresses:
    ~/.ssh/known_hosts:7: daniela-tfe-client.tf-support.hashicorpdemo.com
```

- Once the connection is made successfully and the script has executed, you will see the following
followed by a json response that indicates the organization and the workspace has been created:
```shell
HTTP/2 301 
TFE is up and running
Will continue in 1 minutes with the final steps
curl: (3) URL using bad/illegal format or missing URL
  % Total    % Received % Xferd  Average Speed   Time    Time     Time  Current
                                 Dload  Upload   Total   Spent    Left  Speed
100   202    0   121  100    81    162    109 --:--:-- --:--:-- --:--:--   271
  % Total    % Received % Xferd  Average Speed   Time    Time     Time  Current
                                 Dload  Upload   Total   Spent    Left  Speed
  0     0    0     0    0     0      0      0 --:--:-- --:--:-- --:--:--     
```

### NOTE
The following resource "aws_lb_target_group" "lb_target_group2" has an `unhealthy_threshold = 2`. 
[This value](https://registry.terraform.io/providers/hashicorp/aws/latest/docs/resources/lb_target_group#unhealthy_threshold) might cause the ASG to kill the old instance and spin up a new one.
If that happens, it is because the value is too low and you can increase it.

**Setting up the Active/Active**

- Go to the file `variables.auto.tfvars` and modify just the `tfe_active_active` to `true` 
to spin up just a single instance for now.

- Now, run and we should see only 1 change:
```shell
terraform apply

Terraform will perform the following actions:

  # aws_autoscaling_group.as_group will be updated in-place
  ~ resource "aws_autoscaling_group" "as_group" {
        id                        = "daniela-tfe1-asg"
      ~ launch_configuration      = "daniela-tfe1-lc20230512093022866300000005" -> "daniela-tfe1-lc220230512093022865600000004"
        name                      = "daniela-tfe1-asg"
        # (25 unchanged attributes hidden)

        # (2 unchanged blocks hidden)
    }

Apply complete! Resources: 0 added, 1 changed, 0 destroyed.
```

- Terminate the current instance from the AWS console:
![Terminate](https://github.com/dlavric/tfe-active-active-aws/blob/main/pictures/terminate-instance.png)

- Check that the ASG group is initializing a new instance now:
![Initializing](https://github.com/dlavric/tfe-active-active-aws/blob/main/pictures/initializing.png)

- With the new instance, everything is getting installed again

- Once the new instance is ready, the [connection to the dashboard](https://daniela-tfe.tf-support.hashicorpdemo.com:8800) should error 
with `502 Bad Gateway` as we disabled it.

- Connect to the new instance:
```shell
ssh -J ubuntu@daniela-tfe-client.tf-support.hashicorpdemo.com ubuntu@<new-private-ip-of-the-asg-instance>
```

- We can check the instance is up and running:
```shell
replicatedctl app status

ubuntu@ip-10-234-11-201:~$ replicatedctl app status
[
    {
        "AppID": "ee6ca8fb4fbe4d2d774578d0bc6301d5",
        "Sequence": 692,
        "PatchSequence": 0,
        "State": "started",
        "DesiredState": "started",
        "Error": "",
        "IsCancellable": false,
        "IsTransitioning": false,
        "LastModifiedAt": "2023-05-16T15:42:47.419858528Z"
    }
]
```

- Logout of the instance with `exit`

- Now we scale to 2 nodes, by modifying the file `variables.auto.tfvars` with the following to have 2 nodes:
```hcl
asg_max_size             = 1 ---> 3                        
asg_desired_capacity     = 1 ---> 2
```

- We apply the changes:
```shell
terraform apply

Terraform will perform the following actions:

  # aws_autoscaling_group.as_group will be updated in-place
  ~ resource "aws_autoscaling_group" "as_group" {
      ~ desired_capacity          = 1 -> 2
        id                        = "daniela-tfe1-asg"
      ~ max_size                  = 1 -> 3
        name                      = "daniela-tfe1-asg"
        # (24 unchanged attributes hidden)

        # (2 unchanged blocks hidden)
    }
Plan: 0 to add, 1 to change, 0 to destroy.

Apply complete! Resources: 0 added, 1 changed, 0 destroyed.
```

- If we check AWS, we will notice we will have 2 instance now running:
![Initializing](https://github.com/dlavric/tfe-active-active-aws/blob/main/pictures/2-instances.png)

- We can connect to the new instance:
```shell
ssh -J ubuntu@daniela-tfe-client.tf-support.hashicorpdemo.com ubuntu@<your-private-ip-address-of-the-asg-instance>
```

- Check if the application has started on the 2nd node:
```shell
$ replicatedctl app status

[
    {
        "AppID": "4c74159d3907446d40a2375f9f2eb855",
        "Sequence": 692,
        "PatchSequence": 0,
        "State": "stopped",
        "DesiredState": "started",
        "Error": "",
        "IsCancellable": true,
        "IsTransitioning": true,
        "LastModifiedAt": "2023-05-16T16:08:32.664264166Z"
    }
]

$ replicatedctl app status

[
    {
        "AppID": "4c74159d3907446d40a2375f9f2eb855",
        "Sequence": 692,
        "PatchSequence": 0,
        "State": "started",
        "DesiredState": "started",
        "Error": "",
        "IsCancellable": false,
        "IsTransitioning": false,
        "LastModifiedAt": "2023-05-16T16:09:17.397607268Z"
    }
]
```
**NOTE: Currently we only have a single TFE instance until the 2nd node comes up.**

- To check the Active/Active is running correctly on both instances, a run should be successful once both instances are up and running.

This can be done from the UI, by connecting to the application:
https://daniela-tfe.tf-support.hashicorpdemo.com




