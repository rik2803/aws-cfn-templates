# aws-cfn-templates

## Set the environment

You have to set credentials for all these CFN templates. This is the way to do that, but remember
to comply with the _least privilege_ best practice.
 
* Install the AWS CLI (http://docs.aws.amazon.com/cli/latest/userguide/installing.html)
* Set the correct AWS profile: `export AWS_DEFAULT_PROFILE=myprofile` or
* `export AWS_ACCESS_KEY_ID=myaccesskey; export AWS_SECRET_ACCESS_KEY=mysecretaccesskey`

## `static-web-s3-encrypted.yml`

<a href="https://console.aws.amazon.com/cloudformation/home?region=eu-central-1#/stacks/new?stackName=S3Web&templateURL=https://s3.amazonaws.com/tryx-prd-aws-cloudformation/aws-cfn-templates/static-web-s3-encrypted.yml">
  <img height="24px" src="https://camo.githubusercontent.com/210bb3bfeebe0dd2b4db57ef83837273e1a51891/68747470733a2f2f73332e616d617a6f6e6177732e636f6d2f636c6f7564666f726d6174696f6e2d6578616d706c65732f636c6f7564666f726d6174696f6e2d6c61756e63682d737461636b2e706e67" data-canonical-src="https://s3.amazonaws.com/cloudformation-examples/cloudformation-launch-stack.png" style="max-width:100%;">
</a>

This template creates:

* S3 bucket
* Website is enabled on the bucket
* Enforced data-at-rest encryption throug a bucket policy
* Cloudfront configuration with the bucket as origin
 
### To create the stack

```
aws cloudformation create-stack \
    --stack-name myStackName \
    --template-body file://static-web-s3-encrypted.yml \
    --parameters ParameterKey=BucketName,ParameterValue=yourBucket \
    --parameters ParameterKey=CloudFrontCNAME,ParameterValue=yourDNSPointingToTheCloudFrontDistribution \
    --parameters ParameterKey=Application,ParameterValue=applicationName \
    --parameters ParameterKey=Environment,ParameterValue=applicationEnvironment \
    --parameters ParameterKey=CertificateArn,ParameterValue=youCertificateArn \
    --region eu-central-1
```


### To update the stack

The only difference is the subcommand `update-stack`.

```
aws cloudformation update-stack \
    --stack-name myStackName \
    --template-body file://static-web-s3-encrypted.yml \
    --parameters ParameterKey=BucketName,ParameterValue=yourBucket \
    --parameters ParameterKey=CloudFrontCNAME,ParameterValue=yourDNSPointingToTheCloudFrontDistribution \
    --parameters ParameterKey=Application,ParameterValue=applicationName \
    --parameters ParameterKey=Environment,ParameterValue=applicateionEnvironment \
    --parameters ParameterKey=CertificateArn,ParameterValue=youCertificateArn \
    --region eu-central-1
```

### The _CloudFormation_ parameters used in the template are:

#### `BucketName`

This is the name of the bucket that will be created by the template. The bucket will be created in
the same AWS Region where you create the AWS CloudFormation stack.

#### `CloudFrontCNAME`

The DNS entry you have configured (or will configure if the _CloudFront_ resource has not been created yet) to point
to the _CloudFront_ distribution.

#### `Application` and `Environment`

A descriptive name for the application that will be using the resources created by this stack. Is only used to
tag the resources (where possible; the _CloudFront_ distribution cannnot be tagged from a _CloudFormation_ template).

The environment can be something like `tst`, `prod`, `acc`. `stg` or whatever environment your organization might
be running.

#### `CertificateArn`

Because security is not optional, so is this parameter. Create a AWS Certificate for the CNAME and pass
the ARN of the certificate to the _CloudFormation_ template.

### To copy the files to the S3 bucket

To copy the files with public read permissions to the bucket:

```
cd to_where_your_files_are
aws s3 cp . s3://yourBucket --recursive --acl public-read --sse
cd -
```

Note that the `--sse` switch is required because the buckets policy is set to only accept encrypted puts. This is 
put in place to enforce encryption of data at rest.

## `VPC.yml`

<a href="https://console.aws.amazon.com/cloudformation/home?region=eu-central-1#/stacks/new?stackName=VPC&templateURL=https://s3.amazonaws.com/tryx-prd-aws-cloudformation/aws-cfn-templates/VPC.yml">
  <img height="24px" src="https://camo.githubusercontent.com/210bb3bfeebe0dd2b4db57ef83837273e1a51891/68747470733a2f2f73332e616d617a6f6e6177732e636f6d2f636c6f7564666f726d6174696f6e2d6578616d706c65732f636c6f7564666f726d6174696f6e2d6c61756e63682d737461636b2e706e67" data-canonical-src="https://s3.amazonaws.com/cloudformation-examples/cloudformation-launch-stack.png" style="max-width:100%;">
</a>

The first 2 octets (`xxx.yyy`) of the VPN CIDR are an input parameter for the template. The created VPN is
always a `/16` network.

This template creates:

* A public subnet `xxx.yyy.0.0/24` (i.e. for bastion)
* A IGW
* A NAT GW
* 3 (or 2) Private subnets for applications
  * `xxx.yyy.10.0/24`
  * `xxx.yyy.11.0/24`
  * `xxx.yyy.12.0/24`
* 3 (or 2) Public subnets for ELB
  * `xxx.yyy.20.0/24`
  * `xxx.yyy.21.0/24`
  * `xxx.yyy.22.0/24`
* 3 (or 2) Public subnets for RDS (optional)
  * `xxx.yyy.30.0/24`
  * `xxx.yyy.31.0/24`
  * `xxx.yyy.32.0/24`
* 2 routing tables (private and public)

### To create the stack

```
aws cloudformation create-stack \
    --stack-name myStackName \
    --template-body file://VPC.yml \
    --parameters ParameterKey=Application,ParameterValue=myApplication \
    --parameters ParameterKey=CIDR,ParameterValue=172.12 \
    --parameters ParameterKey=AZs,ParameterValue=3 \
    --parameters ParameterKey=VpcName,ParameterValue=myVPC \
    --parameters ParameterKey=RDSSubNets,ParameterValue=true \
    --region eu-central-1
```

### To update the stack

### The _CloudFormation_ parameters used in the template are:

#### `CIDR`

The first 2 octets of the VPC CIDR. It is completed with the string `.0.0/16` to create a `/16`
VPC.

#### `AZs`

The number of AZs to create the resources in.

#### `VpcName`

The name of the VPC

#### `Application`

A string to define the application running in this VPC. Alos used to tag the resources
created by the template.

## `RDS.yml`

<a href="https://console.aws.amazon.com/cloudformation/home?region=eu-central-1#/stacks/new?stackName=RDS&templateURL=https://s3.amazonaws.com/tryx-prd-aws-cloudformation/aws-cfn-templates/RDS.yml">
  <img height="24px" src="https://camo.githubusercontent.com/210bb3bfeebe0dd2b4db57ef83837273e1a51891/68747470733a2f2f73332e616d617a6f6e6177732e636f6d2f636c6f7564666f726d6174696f6e2d6578616d706c65732f636c6f7564666f726d6174696f6e2d6c61756e63682d737461636b2e706e67" data-canonical-src="https://s3.amazonaws.com/cloudformation-examples/cloudformation-launch-stack.png" style="max-width:100%;">
</a>

### To create the stack

```
aws cloudformation create-stack \
    --stack-name myStackName \
    --template-body file://RDS.yml \
    --parameters ParameterKey=VPCStackName,ParameterValue=nameOgMyVPCStack \
    --parameters ParameterKey=DBName,ParameterValue=myDatabase \
    --parameters ParameterKey=DBUser,ParameterValue=myDatabaseUser \
    --parameters ParameterKey=DBPassword,ParameterValue=myDatabasePassword \
    --region eu-central-1
```

### The _CloudFormation_ parameters used in the template are:

#### `VPCStackName`

This template uses exported resources from the stack created with the `VPC.yml`
CFN template. To be able to extract the values, the name of the stack has to
be known.

#### `DBName`

The name of the DB that will be created.

#### `DBUser`

The user name to connect to the DB.

#### `DBPassword`

The password to connect to the DB.

## `BastionHost.yml`

<a href="https://console.aws.amazon.com/cloudformation/home?region=eu-central-1#/stacks/new?stackName=BastionHost&templateURL=https://s3.amazonaws.com/tryx-prd-aws-cloudformation/aws-cfn-templates/BastionHost.yml">
  <img height="24px" src="https://camo.githubusercontent.com/210bb3bfeebe0dd2b4db57ef83837273e1a51891/68747470733a2f2f73332e616d617a6f6e6177732e636f6d2f636c6f7564666f726d6174696f6e2d6578616d706c65732f636c6f7564666f726d6174696f6e2d6c61756e63682d737461636b2e706e67" data-canonical-src="https://s3.amazonaws.com/cloudformation-examples/cloudformation-launch-stack.png" style="max-width:100%;">
</a>

### To create the stack

```
aws cloudformation create-stack \
    --stack-name myStackName \
    --template-body file://BastionHost.yml \
    --parameters ParameterKey=VPCStackName,ParameterValue=nameOgMyVPCStack \
    --parameters ParameterKey=KeyPaitName,ParameterValue=myKeyPair \
    --region eu-central-1
```

### The _CloudFormation_ parameters used in the template are:

#### `VPCStackName`

This template uses exported resources from the stack created with the `VPC.yml`
CFN template. To be able to extract the values, the name of the stack has to
be known.

#### `KeyPairName`

The name of the keypair that will have to be used to ssh into the
bastion host.
