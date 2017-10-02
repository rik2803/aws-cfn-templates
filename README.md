# aws-cfn-templates

## static-web-s3-encrypted.yml

This template created:

* S3 bucket
* Website is enabled on the bucket
* Enforced data-at-rest encryption throug a bucket policy
* Cloudfront configuration with the bucket as origin


### Set the environment

* Install the AWS CLI (http://docs.aws.amazon.com/cli/latest/userguide/installing.html)
* Set the correct AWS profile: `export AWS_DEFAULT_PROFILE=myprofile` or
* `export AWS_ACCESS_KEY_ID=myaccesskey; export AWS_SECRET_ACCESS_KEY=mysecretaccesskey`

###To create the stack


```
aws cloudformation create-stack \
    --stack-name myStackName \
    --template-body file://static-web-s3-encrypted.yml \
    --parameters ParameterKey=BucketName,ParameterValue=yourBucket \
    --parameters ParameterKey=CloudFrontCNAME,ParameterValue=yourDNSPointingToTheCloudFrontDistribution \
    --parameters ParameterKey=Application,ParameterValue=applicationName \
    --parameters ParameterKey=Environment,ParameterValue=applicateionEnvironment \
    --region eu-central-1
```

The _CloudFormation_ parameters used in the template are:

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

###To update the stack

The only difference is the subcommand `update-stack`.

```
aws cloudformation update-stack \
    --stack-name myStackName \
    --template-body file://static-web-s3-encrypted.yml \
    --parameters ParameterKey=BucketName,ParameterValue=yourBucket \
    --parameters ParameterKey=CloudFrontCNAME,ParameterValue=yourDNSPointingToTheCloudFrontDistribution \
    --parameters ParameterKey=Application,ParameterValue=applicationName \
    --parameters ParameterKey=Environment,ParameterValue=applicateionEnvironment \
    --region eu-central-1
```

### To copy the files to the S3 bucket

To copy the files with public read permissions to the bucket:

```
cd to_where_your_files_are
aws s3 cp . s3://yourBucket --recursive --acl public-read --sse
cd -
```

Note that the `--sse` switch is required because the buckets policy is set to only accept encrypted puts. This is 
put in place to enforce encryption of data at rest. 
