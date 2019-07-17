# Top-level Terraform module for creating a Lambda based API Gateway (Example)

    Reference this website for more detailed information if desired
    https://learn.hashicorp.com/terraform/aws/lambda-api-gateway

This example uses an AWS S3 bucket to store the node.js files.  Multiple versions can be stored and the version number is referenced when the Terraform run is initited.


The Terraform scripts will build the following AWS resources:
* API Gateway referencing the node.js files in S3
* A Lambda function that interacts with the API Gateway
* The necessary configurations for the API Gateway
* the aws_iam_role for Lambda

This module takes these parameters, identified in `lambda.tf`
* `app_version` - The variable referencing the version of the node.js file desired

## Prerequisites

In order to follow this guide you will need an AWS account and to have Terraform installed.

## AWS Account

You'll need an AWS Commercial account (i.e. not GovCloud) in order to repeatably use the Terraform scripts. 

## Create the S3 Bucket for this exercise
For the sake of this tutorial we will create a temporary S3 bucket using the AWS CLI. S3 bucket names are globally unique, so you may need to change the --bucket= argument in the following example and substitute your new bucket name throughout the rest of this tutorial.

    aws s3api create-bucket --bucket=terraform-serverless-example --region=us-east-1

## Creating the example.zip file
The main.js file is zipped and copied to the S3 bucket for use by the Lambda service

    zip example.zip main.js

## Copy the zip file to the S3 bucket

    aws s3 cp example.zip s3://terraform-serverless-example/v1.0.0/example.zip

## Terraform commands
Before you can work with a new configuration directory, it must be initialized using terraform init, which in this case will install the AWS provider:

    terraform init  

    terraform apply -var="app_version=1.0.0"

NOTE; the var parameter value is the same as the version given in the aws s3 cp command above, which basically places the zipped file in a specific s3 folder structure.  Multiple versions of the example.zip file can be created and then later referenced when running the terraform apply command, therefore allowing multiple versions of the main.js file to be saved on s3.  (this is done in the example url and the greeting is changed in the version 1.0.1).


When the terraform apply is finished, you should see output similar to the following;
    
    base_url = https://bkqhuuz8r8.execute-api.us-east-1.amazonaws.com/test

Load the URL given in the output from your run in your favorite web browser. If everything has worked, you will see the text "Hello world!". This message is being returned from the Lambda function code uploaded earlier, via the API Gateway endpoint.

The API gateway can be reviewed by using the AWS console and reviewing the API Gateway screen.
    https://console.aws.amazon.com/apigateway/home?region=us-east-1

And, look at the Lambda function  from the AWS lambda console
    https://console.aws.amazon.com/lambda/home?region=us-east-1#/functions/ServerlessExample?tab=graph
    

A modified version can be created by following the above steps after changing the original main.js file with another greeting and running the terraform apply with the new version number;
    
    terraform apply -var="app_version=1.0.1"


To delete the environment when finished;

    terraform destroy -var="app_version=1.0.0"

NOTE: This will not delete the s3 bucket as it is created outside of the terraform scripts and must be deleted manually.
