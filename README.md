# aws-cfn-templates

## Set the environment

You have to set credentials for all these CFN templates. This is the way to do that, but remember
to comply with the _least privilege_ best practice.
 
* Install the AWS CLI (http://docs.aws.amazon.com/cli/latest/userguide/installing.html)
* Set the correct AWS profile: `export AWS_DEFAULT_PROFILE=myprofile` or
* `export AWS_ACCESS_KEY_ID=myaccesskey; export AWS_SECRET_ACCESS_KEY=mysecretaccesskey`

## static-web-s3-encrypted.yml

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

## VPC.yml

The first 2 octets (`xxx.yyy`) of the VPN CIDR are an input parameter for the template. The created VPN is
always a `/16` network.

This template creates:

* A public subnet `0.0/24` (i.e. for bastion)
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
* 2 routing tables (private and public)

###To create the stack

```
aws cloudformation create-stack \
    --stack-name myStackName \
    --template-body file://VPC.yml \
    --parameters ParameterKey=Application,ParameterValue=myApplication \
    --parameters ParameterKey=CIDR,ParameterValue=172.12 \
    --parameters ParameterKey=AZs,ParameterValue=3 \
    --parameters ParameterKey=VpcName,ParameterValue=myVPC \
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
