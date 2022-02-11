
# Architecture

![](.img/2022-01-24-12-19-13.png)

# Terraform

```sh
make tfenv-install
make terraform-install
```

# AWS

## CLI

It may fail due to `to many open files error` so follow this guide [Set Permanently ulimit -n / open files in ubuntu](https://muhammadtriwibowo.medium.com/set-permanently-ulimit-n-open-files-in-ubuntu-4d61064429a) and do

```sh
# try to set - if it fails continue rest of commands
ulimit -n 65535

# edit sysctl file
sudo vim /etc/sysctl.conf

# append this line
fs.file-max = 2097152
# then exec
sudo sysctl -p

# edit limits file
sudo vim /etc/security/limits.conf

# append this lines
root             -       nofile          unlimited
*                -       nofile          unlimited
*               hard     nofile          unlimited
*               soft     nofile          unlimited

# edit pam common file
sudo vim /etc/pam.d/common-session

# add this line to it
session required pam_limits.so

# reboot
sudo reboot

# try to set ulimit again
ulimit -n 65535
ulimit -n # should be 65535
ulimit -S # should be unlimited

```

Install with `make aws-install`

## Vault CLI

Install with `make aws-vault-install`

## Account

[https://console.aws.amazon.com/iam/home?#/security_credentials](https://console.aws.amazon.com/iam/home?#/security_credentials)

Grab new key and put to .secrets (git ignored) as aws-key.csv

![](.img/2022-01-25-17-00-37.png)


### AWS Local dev

First download AWS key to .secrets dir and split it on two files: AWS_ACCESS_KEY_ID, AWS_SECRET_ACCESS_KEY

```sh
make aws-config
```





## Setup terraform for aws

Ref:
- [Build Infrastructure - Terraform AWS Example | Terraform - HashiCorp Learn](https://learn.hashicorp.com/tutorials/terraform/aws-build?in=terraform/aws-get-started)
- [Build Infrastructure - Terraform AWS Example | Terraform - HashiCorp Learn](https://learn.hashicorp.com/tutorials/terraform/aws-build?in=terraform/aws-get-started#troubleshooting)
- [Find a Linux AMI - Amazon Elastic Compute Cloud](https://docs.aws.amazon.com/AWSEC2/latest/UserGuide/finding-an-ami.html#finding-an-ami-aws-cli)

### Find AMI for `EC2` in REGION

First execute

```sh
make aws-get-list-of-images
```

It will download info about all images (with AMI) to ` .data/aws/ec2-images.json`

Then go to

[https://console.aws.amazon.com/ec2/v2/home?region=us-east-1#Home:](https://console.aws.amazon.com/ec2/v2/home?region=us-east-1#Home:)

Change region in menu 

![](.img/2022-01-25-18-10-41.png)

![](.img/2022-01-25-18-10-52.png)

select `eu-central-1`

click `Launch Instance`

![](.img/2022-01-25-18-11-17.png)

now select image type, ie. Amazon Linux 2 

> TODO: what image to use [Amazon Linux vs. Ubuntu for Amazon EC2](https://serverfault.com/questions/275736/amazon-linux-vs-ubuntu-for-amazon-ec2)

it has AMI `ami-05cafdf7c9f772ad2` (we can look it up in .data/aws/ec2-images.json)

![](.img/2022-01-25-18-57-02.png)

Grab that AMI and add to `src/terraform/main.tf` 

In terraform we select `t2.micro` like this one

![](.img/2022-01-25-19-01-57.png)


# Terraform Dillemas

## Backend config changed

```s
Initializing the backend...
╷                     
│ Error: Backend configuration changed             
│                           
│ A change in the backend configuration has been detected, which may require migrating existing state.          
│ 
│ If you wish to attempt automatic migration of the state, use "terraform init -migrate-state".
│ If you wish to store the current configuration with no changes to the state, use "terraform init -reconfigure".
```

## Modules naming

As there already is `aws/aws_s3_bucket` we need to use different name ie. `mod_aws_s3_bucket`.


# AWS Example commands

##### Show ec2 regions

```sh
aws ec2 describe-regions --query "Regions[].{Name:RegionName}" --output text 
```